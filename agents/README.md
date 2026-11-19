# agents

LangGraph orchestrator and six agents (planned):

| Agent | Responsibility |
|---|---|
| Intake | Voice / video / text input, language detection, transcription, voice follow-up questions |
| Skill & Video Check | Structured profile extraction, video evidence scoring |
| Listing & Fraud | Rule engine + LLM checks on every employer listing before publishing |
| Matching | Hard filters → multilingual embedding search → LLM re-rank with reasons |
| Consent | Per-employer consent via interrupt; writes to the consent ledger |
| Follow-up & Rating | Day 7 / 30 / 90 check-ins, employer ratings, outcome feedback to matching |

See [../docs/architecture.md](../docs/architecture.md) for how they fit together.
