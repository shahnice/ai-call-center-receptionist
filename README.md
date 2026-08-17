# AI Call Center Receptionist

A reusable blueprint for an AI receptionist that answers calls for any business: give it a set of
Q&A pairs and a business/workflow document, and it answers product and support questions — as text
chat today, and as a live, interruptible voice call via a browser demo (with a real Twilio phone
number as the natural next step).

## Stack
- Backend: FastAPI + SQLAlchemy + pgvector + OpenAI (chat, embeddings, Realtime API for voice)
- Frontend: React + Vite
- Vector store: Postgres with the `pgvector` extension
- Voice: OpenAI Realtime API, bridged to a browser mic today; Twilio Programmable Voice is the
  planned path to real phone numbers

## Concepts
- **Business**: a tenant. Owns its own Q&A pairs, documents, greeting/persona, and voice.
- **Q&A pairs + Documents**: both are embedded and merged into one retrieval context per question —
  curated Q&A pairs first, then document excerpts.
- **Voice demo**: browser-based "bring your own OpenAI API key" test call, so trying the voice
  feature doesn't cost the operator anything. Going live on a real phone number is a follow-up step
  (see "Contact us" in the Voice Demo tab).

## Setup

1. Postgres with `pgvector`:
   ```
   docker compose up -d
   ```
   (or a local Postgres 16 install with the `vector` extension enabled — see `backend/init.sql`).

2. Backend:
   ```
   cd backend
   python -m venv .venv && source .venv/bin/activate
   pip install -r requirements.txt
   cp .env.example .env   # add your OPENAI_API_KEY
   uvicorn app.main:app --reload
   ```

3. Frontend:
   ```
   cd frontend
   npm install
   cp .env.example .env
   npm run dev
   ```

4. Open http://localhost:5173:
   - Create a business, add Q&A pairs, upload a document.
   - Use "Test chat" to ask questions against the merged knowledge base.
   - Use the "Voice Demo" tab, enter your own OpenAI API key, and start a live test call.

## Notes
- PDF processing (extract → chunk → embed) happens synchronously in the upload request — fine for
  demo-sized PDFs. For larger scale, move it to a background task/queue.
- Retrieval uses pgvector cosine distance (`<=>`) over the top-K most relevant Q&A pairs and
  document chunks per question.
- The voice demo's OpenAI key is only ever held in the browser tab's `sessionStorage` and passed to
  the backend over the open call WebSocket — it is never written to the database.
- Real phone numbers (Twilio) aren't wired up yet; the "Contact us" link in the Voice Demo tab is the
  intended handoff point for setting that up on a real deployment.
