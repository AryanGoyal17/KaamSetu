# KaamSetu architecture

Four zones, read left to right: **channels** feed an **orchestrator**, the orchestrator drives **six agents**, every agent reads and writes shared **memory and data**, and **external systems** (simulated in the prototype) sit alongside.

## 1. Channels

| Channel | Who it serves | Prototype status |
|---|---|---|
| Worker app (PWA) | Workers with an Android smartphone | Built |
| Employer app (PWA) | Small employers | Built |
| WhatsApp bot | Users who live on WhatsApp but won't install an app | Simulated |
| IVR phone call | Users with only a basic keypad phone | Proposed |
| JanAI Café | Anyone who needs a human helper, document checks or in-person verification | Assisted (operator uses the same app) |

All channels reach the same backend and the same case memory, so a person can start on one channel and continue on another without starting over.

## 2. Orchestrator

Built on **LangGraph**:

- Each case is a **graph**; agents are **nodes**; **conditional edges** decide what runs next (e.g. "profile incomplete → ask a follow-up question").
- The case **state** (transcript, profile, matches, consents, ratings) is **checkpointed** to PostgreSQL after every step. This is the system's memory: a case can pause for days and resume.
- **Interrupts** pause the graph for human input. This is how consent and other approval checkpoints are enforced.
- **Retries and fallback edges** handle failures; when retries don't help, the case is escalated to a human or a JanAI Café.

## 3. Agents

| Agent | Input | Output | Tools |
|---|---|---|---|
| **Intake** | Raw voice / video / text | Transcript, detected language, follow-up questions | Indic speech-to-text, language ID, Indic text-to-speech |
| **Skill & Video Check** | Transcript + video | Structured profile; confidence score per skill | LLM structured extraction, frame sampling, vision-language model, MediaPipe (proposed) |
| **Listing & Fraud** | Employer job profile | Pass / flagged / needs documents | Rule engine, LLM classifier, employer-history lookup |
| **Matching** | Worker profile | Top 5 employers, each with a reason | SQL hard filters, pgvector similarity search, LLM re-ranker |
| **Consent** | A proposed data share | Approved / declined, written to the ledger | Voice prompt, consent ledger |
| **Follow-up & Rating** | A placement | Check-in results, problems flagged, employer rating, day-90 outcome | Scheduled jobs, voice messages / calls |

**Human checkpoints** (enforced with LangGraph interrupts):
- the worker approves every share of their data, one employer at a time;
- flagged listings are reviewed by a person;
- nothing involving money happens without approval.

## 4. Memory and data

| Store | Holds |
|---|---|
| Case memory | Running history of each worker and employer, including ratings |
| PostgreSQL + pgvector | Profiles, job listings, matches, ratings, embeddings, LangGraph checkpoints |
| Object storage (S3-compatible) | Audio and video files (the database keeps only links) |
| Consent ledger + audit log | Append-only records of every consent and every agent action |

**Redis queues** decouple slow work (transcription, video analysis) from user requests: uploads are saved, a job is queued, and background workers process it.

## 5. External systems (simulated in prototype)

- Government employment portals (e.g. National Career Service)
- ITI databases (certificate checks)
- WhatsApp Business API

Each is replaced by a mock service that returns realistic responses, and is labelled **simulated** wherever it appears.

## 6. Example walkthrough

Ravi, an ITI-trained welder, sends a Kannada voice note on WhatsApp.

1. The message reaches the backend; the audio goes to object storage and a job is queued.
2. The orchestrator opens Ravi's case and calls the **Intake** agent, which transcribes the code-mixed Kannada-English speech.
3. The **Skill** agent extracts a profile. Travel distance is missing, so Intake sends a voice question and the case **pauses** (state checkpointed).
4. Ravi replies "15 km". His welding video is checked and receives a confidence score.
5. The **Matching** agent returns five employers with reasons; Intake reads them to him.
6. Ravi picks one. The **Consent** agent pauses for his explicit "yes", writes it to the ledger, then shares his contact.
7. The **Follow-up & Rating** agent schedules check-ins on day 7, 30 and 90 and collects the employer's rating, which is added to Ravi's Skill Passport.

## 7. Fairness safeguards

- Gender, caste and religion are never used as match features.
- Results are audited for skew; any group attributes used for audits are collected optionally, with consent, stored separately and never fed to the matcher.
- Ratings: one per verified hire, visible to the worker, open to dispute; new workers are shown as new, not low-rated.
