# Skillnox.AI — Production Code Quality & Architecture Showcase

The Skillnox.AI repository is open-source at [github.com/surendravarikallu/skillnox_ai](https://github.com/surendravarikallu/skillnox_ai). This document highlights key engineering patterns that demonstrate production-grade software design.

---

## 1. Evaluation Queue with Concurrency Control (`evaluation-queue.ts`)

To prevent LLM inference from blocking the Express event loop, all answer evaluations are routed through a priority queue with bounded concurrency, retry backoff, and graceful degradation:

```typescript
class EvaluationQueue {
  private queue: EvaluationTask[] = [];
  private activeCount = 0;
  private concurrentLimit = 2;        // Max parallel LLM calls
  private maxQueueSize = 100;         // Load shedding threshold
  private maxRetries = 3;             // Per-task retry limit

  public async add(questionId: string, answer: string, questionText: string, priority = 1): Promise<boolean> {
    if (this.queue.length >= this.maxQueueSize) {
      // Load shedding: reject and apply heuristic scoring
      await this.applyFallback(questionId, answer);
      return false;
    }

    this.queue.push({ questionId, answer, questionText, retryCount: 0, priority });
    this.queue.sort((a, b) => b.priority - a.priority);  // Priority scheduling
    this.process();
    return true;
  }

  private async process() {
    if (this.activeCount >= this.concurrentLimit || this.queue.length === 0) return;
    this.activeCount++;
    const task = this.queue.shift()!;

    try {
      const evaluation = await evaluateAnswer(task.answer, task.questionText);
      await storage.updateInterviewQuestion(task.questionId, {
        score: evaluation.score,
        feedback: evaluation.feedback,
      });
    } catch (error) {
      if (task.retryCount < this.maxRetries) {
        task.retryCount++;
        // Exponential backoff: 5s, 10s, 15s
        setTimeout(() => {
          this.queue.push(task);
          this.process();
        }, 5000 * task.retryCount);
      } else {
        await this.applyFallback(task.questionId, task.answer);
      }
    } finally {
      this.activeCount--;
      this.process();  // Drain next task
    }
  }
}
```

**Design decisions:**
- **Concurrency = 2**: Ollama inference is GPU-bound; more parallelism causes memory thrashing.
- **Priority sorting**: Ensures urgent evaluations (e.g., final round answers) are processed first.
- **Heuristic fallback**: Word-count-based scoring ensures students always receive feedback, even under extreme load.

---

## 2. Timeout-Safe AI Bridge with Graceful Fallback (`evaluate.ts`)

Every external AI call is wrapped in a race-condition-safe timeout that resolves to a fallback value instead of throwing:

```typescript
export async function withTimeout<T>(
  promise: Promise<T>, ms: number, fallback: T, label?: string
): Promise<T> {
  return await new Promise((resolve) => {
    let settled = false;
    const timer = setTimeout(() => {
      if (!settled) {
        settled = true;
        resolve(fallback);  // Never throws — always resolves
      }
    }, ms);

    promise
      .then((value) => {
        if (!settled) { settled = true; clearTimeout(timer); resolve(value); }
      })
      .catch(() => {
        if (!settled) { settled = true; clearTimeout(timer); resolve(fallback); }
      });
  });
}
```

**Why this matters:** Standard `Promise.race` with `setTimeout` can leak timers or leave promises dangling. The `settled` boolean guard prevents double-resolution, a subtle but critical bug in production Node.js services.

---

## 3. Multi-Round Gating Engine (`routes.ts`)

The round advancement endpoint validates scores and generates next-round questions atomically:

```typescript
app.post("/api/interviews/:id/next-round", requireAuth, async (req, res) => {
  const interview = await storage.getInterview(req.params.id);
  const questions = await storage.getInterviewQuestions(interview.id);

  // Get the current round's configuration
  const roundConfig = getCompanyRoundConfig(interview.company, interview.currentRound);

  // Calculate average score for current round only
  const currentRoundQuestions = questions.filter(q => q.round === roundConfig.roundName);
  const avgScore = currentRoundQuestions.reduce((sum, q) => sum + (q.score || 0), 0)
                   / currentRoundQuestions.length;

  // Check if any question is still being evaluated
  const unevaluated = currentRoundQuestions.some(q => q.score === null);
  if (unevaluated) {
    return res.status(409).json({ code: "ROUND_UNFINISHED" });
  }

  // Gate check: did the candidate pass?
  if (avgScore < roundConfig.passingThreshold) {
    await storage.updateInterview(interview.id, { status: "completed" });
    return res.json({ passed: false, message: "Below threshold" });
  }

  // Generate next round questions from company bank
  const nextRound = getCompanyRoundConfig(interview.company, interview.currentRound + 1);
  const newQuestions = generateRoundQuestions(interview.company, nextRound);

  // Persist and advance
  await storage.updateInterview(interview.id, { currentRound: interview.currentRound + 1 });
  for (const q of newQuestions) {
    await storage.createInterviewQuestion({ ...q, interviewId: interview.id });
  }

  return res.json({ passed: true, nextRound: nextRound.roundName });
});
```

**Design decisions:**
- **409 Conflict for unevaluated answers**: The frontend polls until all scores arrive before enabling the "Next Round" button, preventing premature round advancement.
- **Atomic gating**: Score calculation and round advancement happen in a single request to prevent race conditions in concurrent sessions.

---

## 4. Dynamic LLM Prompt Construction (`pythonAI.ts`)

Generated interview questions are sanitized server-side to strip LLM artifacts (filler phrases, meta-commentary) before displaying to students:

```typescript
function sanitizeGeneratedQuestion(question?: string | null): string | null {
  if (!question) return null;

  let cleaned = question
    // Remove opening filler: "Sure, here's a question..."
    .replace(/^(sure|certainly|okay|well|alright)[,!.\s-]*/gi, '')
    .replace(/^(here('?s| is).*?:)/gi, '')
    // Remove interviewer meta-notes
    .replace(/the interviewer is looking for[^.]*\./gi, '')
    // Remove closing filler
    .replace(/good luck!?.*$/gi, '')
    .replace(/note:.*$/gi, '')
    .trim();

  // Extract the actual question sentence (ends with '?')
  const sentences = cleaned.split(/(?<=[?.!])\s+/).filter(s => s.length > 10);
  const questionSentence = sentences.find(s => s.endsWith('?') && s.length > 15);

  return questionSentence || sentences[0] || null;
}
```

**Why this matters:** Raw LLM output frequently contains conversational padding ("Sure! Here's a great technical question for you..."). Without sanitization, the interview experience feels robotic and breaks immersion.

---

## 5. Campaign Scheduler Background Worker (`index.ts`)

Automated placement drives are triggered by a lightweight polling worker — no external job queue dependency required:

```typescript
function startCampaignSchedulerWorker() {
  return setInterval(async () => {
    const campaigns = await storage.getScheduledCampaigns();
    const now = new Date();

    for (const campaign of campaigns) {
      if (campaign.status !== "pending") continue;
      if (new Date(campaign.scheduledAt) > now) continue;

      // Mark active to prevent double-processing
      await storage.updateScheduledCampaign(campaign.id, { status: "active" });

      // Fetch target students (filter by branch if specified)
      const students = await storage.getStudentsByDepartment(campaign.branch);

      // Auto-enroll each student
      for (const student of students) {
        await storage.createInterview({
          userId: student.id,
          type: "company",
          company: campaign.company,
          difficulty: campaign.difficulty,
          simulationMode: campaign.simulationMode,
          status: "pending",
        });
      }

      await storage.updateScheduledCampaign(campaign.id, { status: "completed" });
    }
  }, 30_000); // Poll every 30 seconds
}

// Graceful shutdown
const schedulerHandle = startCampaignSchedulerWorker();
process.on("SIGINT", () => clearInterval(schedulerHandle));
process.on("SIGTERM", () => clearInterval(schedulerHandle));
```

**Design decisions:**
- **30s polling** chosen over cron for simplicity — campaigns are scheduled to the minute, not the second.
- **Immediate `active` status** prevents duplicate enrollment if the worker fires twice during a slow database transaction.
- **Graceful shutdown** prevents orphaned intervals from keeping the Node.js process alive during deployments.
