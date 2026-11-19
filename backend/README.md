# backend

FastAPI (Python) service (planned):

- REST API for the worker and employer apps and the simulated WhatsApp webhook
- PostgreSQL + pgvector for profiles, listings, matches, ratings, embeddings and agent checkpoints
- S3-compatible object storage for audio and video
- Redis queue + background workers for transcription and video analysis
- Consent ledger and audit log (append-only tables)
- Mock services for government portals, ITI databases and WhatsApp, clearly labelled **simulated**

Deployment: Docker / docker compose.
