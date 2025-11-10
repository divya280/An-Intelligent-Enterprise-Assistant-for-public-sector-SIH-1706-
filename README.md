# An-Intelligent-Enterprise-Assistant-for-public-sector-SIH-1706-
## Problem Statement
Develop an AI-powered chatbot for a large public-sector enterprise that understands employee questions around HR policies, IT support, company events, and other organizational topics. The assistant must:

- Reliably interpret natural-language queries and respond accurately within 5 seconds.
- Provide secure access with email-based two-factor authentication (2FA).
- Process employee-uploaded documents (8–10 pages) to extract key information and summaries.
- Filter inappropriate language using a system-maintained dictionary.
- Scale to handle at least five concurrent users.
- Persist conversations, documents, and audit trails in Supabase.

## Core Idea & Capabilities
- **Unified employee helpdesk:** Centralize HR, IT, and operations knowledge in a conversational interface.
- **Document understanding:** Summarize and extract keywords from uploaded PDFs to support quick insights.
- **Secure access control:** Combine Supabase Auth with OTP-based 2FA and role-aware endpoints.
- **Moderated conversations:** Automatically block or redact bad language via a managed dictionary.
- **Scalable async backend:** FastAPI with async database calls, background tasks, and optional caching ensures low latency.

## Tech Stack
- **Backend:** Python 3.11+, FastAPI, Uvicorn
- **Database & Auth:** Supabase (PostgreSQL + Supabase Auth)
- **Async ORM/Client:** Supabase Python client, asyncpg
- **Authentication:** `python-jose`, `passlib` for hashing, custom OTP flow via SMTP (`aiosmtplib`)
- **AI & NLP:** OpenAI-compatible API (configurable model), `nltk`, `scikit-learn` for document analysis
- **Document Handling:** `pypdf` for PDF parsing, `nltk` tokenizers and stopwords
- **Caching / Messaging (optional):** Redis for rate limiting or OTP cache
- **Configuration & Logging:** `pydantic-settings`, `loguru`
- **Deployment:** Container-friendly (Docker), ready for cloud hosting

## Architecture Flow
```
[Client UI]
   |
   v
[FastAPI Backend]
   |-- /api/auth -> Supabase Auth & OTP email (SMTP)
   |-- /api/chat -> Moderation -> LLM (OpenAI) -> Supabase chat history
   |-- /api/documents -> Storage upload -> PDF parsing -> NLP summary -> Supabase metadata
   |-- /api/moderation -> Admin CRUD on banned phrases

Core Services:
- Auth Service: JWT issuance, refresh tokens, OTP verification
- Chat Service: Retrieves past context, calls LLM, stores responses
- Document Service: Extracts text, summarizes, saves metadata & storage path
- Moderation Service: Caches banned terms from Supabase dictionary

Data Stores:
- Supabase tables (users, sessions, chat_messages, documents, moderation_dictionary, refresh_tokens)
- Supabase storage bucket for document binaries
- Optional Redis for caching rate limits and moderation dictionary
```

## Implementation Guide
1. **Environment Setup**
   - Install requirements: `pip install -r requirements.txt`
   - Create `.env` with FastAPI secrets, Supabase keys, SMTP credentials, and optional OpenAI API key.
   - Ensure Supabase project has required tables and storage bucket (`documents`).

2. **Database & Supabase Configuration**
   - Provision tables:
     - `users`, `chat_messages`, `documents`, `two_factor_otp`, `refresh_tokens`, `moderation_dictionary`, `audit_logs`.
   - Create Supabase Storage bucket for document uploads.
   - Add RPC or Row Level Security rules as appropriate for role-based access.

3. **Run the Backend**
   - Start API: `uvicorn app.main:app --reload`
   - Health check: `GET /api/health`

4. **Authentication Workflow**
   - `POST /api/auth/register` to create user (Supabase sign-up + metadata).
   - `POST /api/auth/login` (email + password) triggers OTP email.
   - `POST /api/auth/verify-otp` validates OTP and returns access/refresh tokens.
   - `GET /api/auth/me` requires Bearer token with `two_factor_verified=True`.

5. **Chat Interaction**
   - `POST /api/chat/query` with employee question.
   - Request is moderated, conversation context is retrieved, and LLM generates a response.
   - Conversation logs stored in `chat_messages` for analytics and follow-ups.

6. **Document Processing**
   - `POST /api/documents/upload` accepts PDF upload, stores binary in Supabase, extracts text, summary, and keywords.
   - `GET /api/documents/{id}` returns metadata and summary for authorized user.

7. **Moderation Management**
   - `GET /api/moderation/dictionary` allows admin/moderator roles to review banned phrases.
   - Extend with POST/DELETE operations as needed for dictionary maintenance.

8. **Scaling & Deployment**
   - Containerize FastAPI app and configure Supabase credentials via environment variables.
   - Use Gunicorn/Uvicorn workers or serverless deployment for horizontal scaling.
   - Optionally configure Redis or in-memory caching to reduce latency for moderation dictionary and chat context.

9. **Testing & Monitoring**
   - Add pytest suites for auth flows, OTP verification, chat moderation, and document processing.
   - Instrument logging with Loguru, export metrics via `/api/health` or custom Prometheus endpoint.
   - Monitor Supabase dashboards for authentication and database usage statistics.

## Next Steps
- Implement front-end client (web or Teams/Slack bot) consuming the FastAPI endpoints.
- Enhance knowledge retrieval with vector embeddings (e.g., `pgvector`, Supabase Vector) for semantic search.
- Automate moderation dictionary updates via admin panel or CSV import.
