# System Design Document — Skillnox.AI Interview Platform

## 1. Executive Summary

Skillnox.AI is an AI-powered placement interview simulation platform that evaluates candidates using locally hosted large language models (LLMs). The system orchestrates multi-round interview simulations (Aptitude → Technical → HR), scores answers in real-time using a Qwen-based local inference pipeline, and provides recruiter-facing portfolio sharing — all while keeping candidate data fully on-premise.

## 2. System Architecture

### 2.1 High-Level Architecture

```
                      ┌──────────────────────────┐
                      │   Client Web Browser     │
                      │  (React 18 SPA + Vite)   │
                      └────────────┬─────────────┘
                                   │ HTTPS (REST API)
                                   ▼
                      ┌──────────────────────────┐
                      │   Node.js / Express.js   │
                      │   (API Gateway + Auth)   │
                      └───┬──────────┬───────────┘
                          │          │
          ┌───────────────▼──┐  ┌────▼───────────────────┐
          │  PostgreSQL 15   │  │  Python FastAPI Service │
          │  (Drizzle ORM)   │  │  (Port 8000)           │
          │  Primary DB      │  ├────────────────────────┤
          └──────────────────┘  │  Ollama Runtime        │
                                │  (Qwen 2.5 3B LLM)    │
                                │  Local Inference       │
                                └────────────────────────┘
```

### 2.2 Component Architecture

1. **Frontend (Client)**
   * **Framework**: React 18 + TypeScript + Vite, styled with Tailwind CSS and shadcn/ui components.
   * **State Management**: TanStack React Query for server-state caching and declarative data fetching.
   * **Interview Room**: Real-time question display with voice-to-text transcription (Web Speech API), timed answer submission, and emotion detection via webcam capture.
   * **Recruiter Portfolio**: Public `/shared/report/:token` route renders a privacy-masked, read-only candidate scorecard accessible without authentication.

2. **Backend (Node.js API Gateway)**
   * **Express.js Server**: RESTful API with JWT-based authentication (cookie transport), Zod schema validation, and gzip compression.
   * **Multi-Round Engine**: The `POST /api/interviews/:id/next-round` endpoint evaluates the current round's average score against a configurable passing threshold. If passed, it generates questions for the next round from a 1,000+ company-specific question bank. If failed, the interview terminates.
   * **Evaluation Queue**: A priority queue (`EvaluationQueue` class) with concurrency limiting (2 active evaluations), retry backoff (up to 3 attempts with 5s × attempt delay), load shedding (rejects at 100 pending), and heuristic fallback scoring.
   * **Background Scheduler Worker**: A `setInterval`-based poller (30s interval) that auto-enrolls students into scheduled placement campaigns when the launch time arrives. Cleans up on `SIGINT`/`SIGTERM`.

3. **AI Service (Python FastAPI + Ollama)**
   * **Qwen 2.5 3B**: Locally hosted via Ollama for zero-latency, zero-cost inference with complete data privacy.
   * **Dynamic Prompt Engineering**: Each interview question prompt is dynamically constructed using company-specific interviewer personas (e.g., Google's Googliness rubric, Amazon's Leadership Principles) and trending 2025-26 technical topics (RAG, Vector Databases, LLM fine-tuning).
   * **Services**: `POST /evaluate` (answer scoring), `POST /generate-question` (dynamic question generation), `POST /analyze-resume` (resume parsing with Jinja2 templates), `POST /analyze-emotion` (facial expression classification from base64 webcam frames).

4. **Data Layer**
   * **PostgreSQL 15**: Primary relational database with UUID primary keys, indexed foreign keys, and cascading deletes. Managed via Drizzle ORM with type-safe schema generation.
   * **13 Tables**: `users`, `interviews`, `interview_questions`, `resumes`, `job_descriptions`, `personality_assessments`, `placement_probabilities`, `gd_sessions`, `scheduled_campaigns`, `interview_slots`, `daily_analytics`, `system_logs`, `global_settings`.

## 3. Interview Simulation Flow

```
┌──────────┐    ┌────────────────┐    ┌─────────────────┐    ┌─────────────┐
│ Student  │───▶│ Select Company │───▶│ Configure Sim   │───▶│  Start      │
│ Login    │    │ & Difficulty   │    │ Mode / Rounds   │    │  Interview  │
└──────────┘    └────────────────┘    └─────────────────┘    └──────┬──────┘
                                                                    │
                ┌───────────────────────────────────────────────────┘
                ▼
   ┌────────────────────┐     ┌─────────────────────┐     ┌──────────────┐
   │ Round 1: Aptitude  │────▶│ Score ≥ Threshold?  │─No─▶│ Interview    │
   │ (5 questions)      │     │ (default 40%)       │     │ Terminated   │
   └────────────────────┘     └─────────┬───────────┘     └──────────────┘
                                        │ Yes
                                        ▼
   ┌────────────────────┐     ┌─────────────────────┐     ┌──────────────┐
   │ Round 2: Technical │────▶│ Score ≥ Threshold?  │─No─▶│ Interview    │
   │ (5 questions)      │     │ (default 50%)       │     │ Terminated   │
   └────────────────────┘     └─────────┬───────────┘     └──────────────┘
                                        │ Yes
                                        ▼
   ┌────────────────────┐     ┌──────────────────────────────────────┐
   │ Round 3: HR        │────▶│ Final Scoring + Feedback Generation │
   │ (3 questions)      │     │ → Results Page + PDF Export         │
   └────────────────────┘     └──────────────────────────────────────┘
```

## 4. Evaluation Queue Architecture

The evaluation queue decouples answer submission from AI scoring to prevent request timeouts during LLM inference:

```
Student submits answer
        │
        ▼
┌──────────────────────────┐
│  EvaluationQueue.add()   │
│  Priority: 1 (default)   │
├──────────────────────────┤
│  Queue full (>100)?      │──Yes──▶ Load shedding: heuristic fallback
│                          │         (word-count based scoring)
│  No ▼                    │
│  Sort by priority        │
│  Process (max 2 active)  │
├──────────────────────────┤
│  Call Python AI Service  │
│  evaluateAnswer()        │
│  Timeout: 60 seconds     │
├──────────────────────────┤
│  Success?                │──No──▶ Retry (max 3x, backoff 5s × attempt)
│  Yes ▼                   │         └──▶ Max retries → heuristic fallback
│  Update DB with score    │
└──────────────────────────┘
```

## 5. Company Question Bank

The platform includes a structured question bank covering **23 companies** across 4 industry segments:

| Segment | Companies |
|---|---|
| **IT Services** | TCS, Infosys, Wipro, Accenture, Cognizant, Capgemini, HCL, Tech Mahindra, L&T Infotech, Mindtree |
| **Global Tech** | Google, Microsoft, Amazon, Meta |
| **Indian Startups** | Zoho, Flipkart, Paytm, Razorpay, Freshworks, CRED |
| **BFSI & Consulting** | IBM, Goldman Sachs, Deloitte |

Each company entry provides:
- **Interviewer persona** with company-specific evaluation rubrics
- **Round configurations** with customizable passing thresholds
- **1,000+ curated questions** across aptitude, technical, HR, and behavioral categories

## 6. Security & Privacy

- **JWT Authentication**: Stateless token-based auth with HTTP-only cookie transport.
- **Recruiter Data Masking**: Public portfolio links mask the candidate's last name (initials only), email address, roll number, and exclude voice/video recordings.
- **Input Sanitization**: All user inputs are validated through Zod schemas before database insertion.
- **Local AI Inference**: Zero data leaves the server — all LLM processing happens on-premise via Ollama.

## 7. Technology Stack

| Layer | Technology |
|---|---|
| Frontend | React 18, TypeScript, Vite, Tailwind CSS, shadcn/ui, TanStack Query, Framer Motion |
| Backend | Node.js, Express.js, JWT Auth, Zod Validation, Drizzle ORM |
| AI Service | Python, FastAPI, Ollama, Qwen 2.5 3B, Jinja2 Templates |
| Database | PostgreSQL 15, Drizzle ORM (type-safe schema) |
| DevOps | PM2, GitHub Actions (CI/CD), Docker (optional) |
