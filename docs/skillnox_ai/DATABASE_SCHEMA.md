# Database Schema — Skillnox.AI Interview Platform

This document describes the complete PostgreSQL database schema powering the Skillnox.AI platform. All tables use UUID primary keys (`gen_random_uuid()`), indexed foreign keys with cascading deletes, and timestamp tracking.

---

## Entity Relationship Diagram

```
┌──────────┐       ┌──────────────┐       ┌─────────────────────┐
│  users   │──1:N──│  interviews  │──1:N──│ interview_questions  │
│          │       │              │       │                     │
│ id (PK)  │       │ id (PK)      │       │ id (PK)             │
│ email    │       │ userId (FK)  │       │ interviewId (FK)    │
│ role     │       │ type         │       │ question            │
│ rollNo   │       │ company      │       │ round               │
│ dept     │       │ status       │       │ userAnswer          │
│ college  │       │ simMode      │       │ score               │
└──────────┘       │ currentRound │       │ feedback            │
     │             │ roundResults │       └─────────────────────┘
     │             │ shareToken   │
     │             │ isShared     │
     │             └──────────────┘
     │
     ├──1:N──┌──────────────┐
     │       │   resumes    │
     │       │ parsedData   │
     │       │ skills[]     │
     │       │ overallScore │
     │       └──────────────┘
     │
     ├──1:N──┌───────────────────────┐
     │       │   job_descriptions    │
     │       │ requiredSkills[]      │
     │       │ matchScore            │
     │       │ skillGaps[]           │
     │       └───────────────────────┘
     │
     ├──1:N──┌───────────────────────────┐
     │       │ personality_assessments   │
     │       │ introvertExtrovert       │
     │       │ thinkerFeeler            │
     │       │ dominantTraits[]         │
     │       └───────────────────────────┘
     │
     ├──1:N──┌───────────────────────────┐
     │       │ placement_probabilities   │
     │       │ probability30/60/90Days  │
     │       │ confidence               │
     │       │ recommendations[]        │
     │       └───────────────────────────┘
     │
     └──1:N──┌───────────────┐
             │  gd_sessions  │
             │ topic         │
             │ transcript    │
             │ overallScore  │
             └───────────────┘

Standalone Tables:
┌──────────────────────┐  ┌─────────────────┐  ┌────────────────┐
│ scheduled_campaigns  │  │ daily_analytics │  │ global_settings│
│ title, company       │  │ date            │  │ key, value     │
│ difficulty, branch   │  │ totalInterviews │  └────────────────┘
│ scheduledAt, status  │  │ avgDuration     │
└──────────────────────┘  │ peakHour        │
                          └─────────────────┘
```

---

## Table Definitions

### 1. `users`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | `VARCHAR` | PK, UUID default | Unique user identifier |
| `email` | `VARCHAR` | UNIQUE, NOT NULL | Login email |
| `password_hash` | `VARCHAR` | | Bcrypt hashed password |
| `roll_number` | `VARCHAR` | INDEXED | Student roll number |
| `first_name` | `VARCHAR` | | |
| `last_name` | `VARCHAR` | | |
| `role` | `ENUM('student','admin')` | NOT NULL, default `student` | Access control role |
| `year` | `INTEGER` | | Academic year |
| `department` | `VARCHAR` | | Branch / department |
| `college` | `VARCHAR` | | Institution name |
| `interview_count` | `INTEGER` | NOT NULL, default 0 | Total interviews taken |
| `created_at` | `TIMESTAMP` | default `NOW()` | |

### 2. `interviews`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | `VARCHAR` | PK, UUID | |
| `user_id` | `VARCHAR` | FK → users, CASCADE, INDEXED | |
| `type` | `ENUM` | NOT NULL | Primary interview type |
| `types` | `JSONB` | | Array of selected types |
| `difficulty` | `ENUM('easy','medium','hard')` | | |
| `status` | `ENUM` | NOT NULL, INDEXED | `pending` / `in_progress` / `completed` / `cancelled` |
| `company` | `VARCHAR` | | Target company (e.g. "Google") |
| `simulation_mode` | `VARCHAR` | | `full` (multi-round) or `combined` |
| `current_round` | `INTEGER` | default 0 | Active round index (0-based) |
| `round_results` | `JSONB` | | `[{round, score, passed, feedback}]` |
| `trending_enabled` | `BOOLEAN` | default false | Whether trending topics were injected |
| `technical_score` | `REAL` | | |
| `communication_score` | `REAL` | | |
| `emotion_score` | `REAL` | | |
| `voice_score` | `REAL` | | |
| `overall_score` | `REAL` | | |
| `feedback` | `TEXT` | | AI-generated summary feedback |
| `share_token` | `VARCHAR` | | UUID token for public portfolio link |
| `is_shared` | `BOOLEAN` | default false | Whether the report is publicly accessible |
| `created_at` | `TIMESTAMP` | INDEXED | |

### 3. `interview_questions`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | `VARCHAR` | PK, UUID | |
| `interview_id` | `VARCHAR` | FK → interviews, CASCADE, INDEXED | |
| `question` | `TEXT` | NOT NULL | The question text |
| `round` | `VARCHAR` | | Round name: `aptitude`, `technical`, `hr` |
| `expected_answer` | `TEXT` | | Model answer / rubric |
| `user_answer` | `TEXT` | | Student's submitted answer |
| `score` | `REAL` | | AI-assigned score (0-100) |
| `feedback` | `TEXT` | | AI feedback on the answer |
| `order_index` | `INTEGER` | NOT NULL | Display order within the interview |

### 4. `scheduled_campaigns`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | `VARCHAR` | PK, UUID | |
| `title` | `VARCHAR` | NOT NULL | Campaign name |
| `company` | `VARCHAR` | | Target company or null |
| `difficulty` | `VARCHAR` | NOT NULL | |
| `simulation_mode` | `VARCHAR` | NOT NULL | |
| `branch` | `VARCHAR` | | Target department filter |
| `scheduled_at` | `TIMESTAMP` | NOT NULL | When to auto-enroll students |
| `status` | `VARCHAR` | default `pending` | `pending` / `active` / `completed` |

### 5. Other Tables

| Table | Purpose | Key Columns |
|---|---|---|
| `resumes` | Stores parsed resume data and AI skill extraction | `parsed_data (JSONB)`, `skills[]`, `overall_score` |
| `job_descriptions` | JD matching analysis with skill gap detection | `required_skills[]`, `match_score`, `skill_gaps[]` |
| `personality_assessments` | 4-axis personality profiling | `introvert_extrovert`, `thinker_feeler`, `dominant_traits[]` |
| `placement_probabilities` | AI-predicted placement likelihood | `probability_30/60/90_days`, `confidence`, `recommendations[]` |
| `gd_sessions` | Group discussion simulations | `topic`, `transcript`, `leadership_score`, `overall_score` |
| `daily_analytics` | Aggregated admin dashboard metrics | `total_interviews`, `avg_duration_minutes`, `peak_hour` |
| `global_settings` | Key-value admin configuration | `key`, `value`, `description` |
| `interview_slots` | Bookable interview time slots | `start_time`, `end_time`, `is_booked`, `booked_by_user_id` |

---

## Indexes

| Table | Index | Column(s) |
|---|---|---|
| `users` | `idx_users_roll_number` | `roll_number` |
| `interviews` | `idx_interviews_user_id` | `user_id` |
| `interviews` | `idx_interviews_status` | `status` |
| `interviews` | `idx_interviews_created_at` | `created_at` |
| `interview_questions` | `idx_interview_questions_interview_id` | `interview_id` |
| `resumes` | `idx_resumes_user_id` | `user_id` |
| `job_descriptions` | `idx_job_descriptions_user_id` | `user_id` |

---

## Schema Management

- **ORM**: Drizzle ORM with compile-time type safety and automatic Zod validation schema generation (`createInsertSchema`).
- **Migrations**: `npm run db:push` applies schema changes directly. Production migrations via `drizzle-kit generate` + `drizzle-kit migrate`.
- **Relations**: Defined using Drizzle's `relations()` API for type-safe joins and nested queries.
