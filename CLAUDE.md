# ORVEXA — Claude Code Instructions

## What this project is

ORVEXA is a web application that lets residential building residents ask plain-English questions
about building bylaws and get back accurate, cited answers powered by RAG (Retrieval-Augmented
Generation). Concierge staff upload documents; residents query them via chat.

This is a real product used by real residents. Wrong answers can cause fines. Treat every
generation-layer decision with that in mind.

---

## How we work

The user is learning the whole product lifecycle through this project. Explain in plain,
kid-friendly language, and avoid jargon unless it's explained.

### Every section starts and ends with the journey map
- **Before** a section: show the relevant branch of the tree in `docs/journey.md` with
  "📍 You are here" and one sentence on what this stage is and why it matters.
- **After** a section: update `docs/journey.md` (statuses, section log, and agent scorecard) and
  show what changed (⬜ → 🔵 → ✅), which agents ran and what each found, and what comes next.
  Never count an agent as run unless it actually ran in that section.

### Plan → build → test → review
1. **Plan**: anything beyond a small fix needs the goal, the plan in plain English, the files
   you'll touch, 3 checks that will prove it works, and which agents will review. Run
   `engineering-manager` on the plan (and `ceo-reviewer` for a new feature or phase). Wait for "go".
2. **Build** on a `feature/*` or `fix/*` branch, one small piece per commit.
3. **Test**: run `qa-tester`. Run each check for real and report pass/fail with the output.
   A check you didn't run stays unticked.
4. **Review** (agents in `.claude/agents/`):

   | Agent | Runs when |
   |---|---|
   | `code-reviewer` | Every PR |
   | `security-reviewer` | Auth, uploads, RLS, storage, env/config, personal data, or the RAG pipeline |
   | `rag-reviewer` | RAG services, the system prompt, model settings, or the eval set |
   | `frontend-designer` | Builds every screen; reviews any UI change |
   | `engineering-manager` | Every plan, and every PR before merging into develop |
   | `ceo-reviewer` | New feature or phase, and before the first release to real residents |
   | `release-manager` | Every merge from develop into main |

5. Fix every **must fix** finding before merging. Record any lasting decision in `docs/decisions.md`.
6. Stop and ask before: a new package, a schema change, anything touching personal data,
   a production deploy, or a legal/privacy claim.
7. Before any commit or push, check the change against the security rules below, and say what
   you checked.

### Memory: where things are remembered
- **CLAUDE.md** (this file): rules for every session and every collaborator.
- **`docs/decisions.md`**: what we decided and why, so nothing is re-explained or re-argued.
- **`docs/journey.md`**: where the project is in its lifecycle.
- **Claude's private memory**: the user's personal preferences across sessions.

---

## Tech stack — use exactly these, never substitute

| Layer | Technology |
|---|---|
| Frontend | React + TypeScript (Vite) — located in apps/web/ |
| Backend API | Node.js + Express + TypeScript — located in apps/api/ |
| Database | Supabase (Postgres + pgvector extension) |
| Auth | Supabase Auth — email + password, invite-only (open sign-up disabled) |
| File storage | Supabase Storage — private bucket, short-lived signed URLs |
| AI embeddings | OpenAI, model from `OPENAI_EMBEDDING_MODEL` (default text-embedding-3-small) |
| AI rerank / generation / faithfulness | OpenAI, models from `OPENAI_*_MODEL` env vars (default GPT-4o Mini) |
| Document parsing | pdf-parse (PDF), mammoth (.docx) |
| Frontend hosting | Vercel |
| API hosting | Railway |

---

## Where things live (one home per fact)

```
Orvexa_RAG/
├── apps/
│   ├── web/                    # React frontend
│   │   ├── .env.example        # Browser-safe env vars only (VITE_*)
│   │   └── src/{pages,components,hooks,lib}
│   └── api/                    # Express backend
│       ├── .env.example        # Server-only env vars
│       └── src/
│           ├── routes/         # Thin Express route handlers
│           ├── services/       # RAG pipeline, ingestion, OpenAI calls
│           ├── middleware/
│           ├── prompts/
│           │   └── system.md   # THE system prompt — loaded at runtime
│           └── db/             # Supabase clients (per-user + service)
├── supabase/migrations/        # Source of truth for the schema (once created)
├── .claude/                    # Claude Code settings + agents
├── CLAUDE.md
└── README.md
```

- **System prompt:** `apps/api/src/prompts/system.md`. Never edit its wording without explicit instruction.
- **Schema:** once `supabase/migrations/` exists, it is the truth and the summary below must match it.

---

## Database schema (Supabase Postgres + pgvector)

Core tables. Do not rename them or add columns without explicit instruction.

```
buildings         — id, name, address, created_at
users             — id (Supabase Auth UID), building_id, role, email, created_at
documents         — id, building_id, name, file_type, storage_path, published_at, status, url
chunks            — id, document_id, building_id, section_id, content, embedding (vector 1536), token_count, page_number
query_logs        — id, user_id, building_id, query_text, chunk_ids[], similarity_scores[], model_version, tokens_in, tokens_out, latency_ms, confidence, thumbs, created_at
escalations       — id, query_log_id, reason, conversation (jsonb), status, resolved_at
```

`documents.url` must never hold a permanent public link. Generate short-lived signed URLs from
`storage_path` when a link is needed.

---

## API endpoints

| Method | Path | Auth required | What it does |
|---|---|---|---|
| POST | /query | Yes (resident) | Run RAG pipeline, return answer + citation |
| POST | /feedback | Yes | Save thumbs up/down on a query_log |
| POST | /documents/upload | Yes (concierge) | Upload PDF or .docx, trigger ingestion |
| GET | /documents | Yes (concierge) | List documents for the concierge's building |
| GET | /escalations | Yes (concierge) | List open escalations |
| PATCH | /escalations/:id | Yes (concierge) | Mark escalation resolved |
| GET | /analytics/top-queries | Yes (concierge) | Top 20 most-queried topics (P2) |

There is no login endpoint. The web app signs in with Supabase Auth directly; passwords never
pass through the API. The API only verifies the access token with `supabase.auth.getUser(token)`.

---

## RAG pipeline (apps/api/src/services/rag.ts)

The query flow must follow this exact order:

1. Embed the query.
2. Retrieve top `RAG_TOP_K_RETRIEVE` chunks from pgvector, filtered **in SQL** by the caller's
   building_id and status = 'active'.
3. Re-rank to the best `RAG_TOP_K_RERANK` with an LLM call.
4. Generate the answer with the system prompt from `prompts/system.md`.
5. Run a faithfulness check — does every claim in the answer exist in the retrieved chunks?
6. If faithfulness passes, return answer + citation. If it fails, hold the answer and create an escalation.

Prompt-injection controls (a bylaw PDF can contain "ignore previous instructions"):
- Wrap each chunk in clear delimiters and label it as reference text, not instructions.
- The building_id filter lives in SQL, never in the prompt.
- Check citations in code against the IDs of the chunks actually retrieved, not the citation text the model writes.
- The eval set must include one "poisoned" test document.

Log tokens in/out and latency for every model call (the `query_logs` columns exist for this).

---

## Security rules — never violate these

1. **building_id and role come from the server, never from the request.**
   After verifying the token, look both up in the `users` table (or `app_metadata` / a custom
   access token hook). Never read them from `user_metadata` (users can edit it), the request body,
   or query params. Use `req.user.building_id` and `req.user.role` only.

2. **Resident and concierge requests use a per-request Supabase client built from the caller's
   own access token** (anon key + `Authorization: Bearer <token>`), so Row Level Security applies.
   The service role key bypasses RLS — use it **only** for document ingestion.

3. **RLS is on for every table in `public`**, each with a policy, and a test proving a user from
   building A cannot read building B. Never disable it. Example:
   ```sql
   alter table chunks enable row level security;
   create policy "read own building" on chunks for select to authenticated
     using (building_id = (select building_id from public.users where id = auth.uid()));
   ```

4. **Database functions (including vector search) never take building_id from the caller.**
   Derive it inside the function from `auth.uid()` and keep them `SECURITY INVOKER`.

5. **The SUPABASE_SERVICE_ROLE_KEY and OPENAI_API_KEY never reach the frontend.** They live in
   `apps/api/.env` locally and in Railway env settings in production. `apps/web/.env` holds only
   `VITE_*` browser-safe values.

6. **File uploads:** check the real file type from its first bytes (magic bytes), not the
   extension — PDF and .docx only. Parse in a worker with a timeout (a .docx is a zip and can be a
   zip bomb). Max size from `UPLOAD_MAX_MB`.

7. **Rate limiting:** enforce `RAG_DAILY_QUERY_LIMIT` queries per user per day on /query, with the
   counter stored in Postgres (not memory — Railway restarts reset memory).

8. **Never log the content of uploaded documents** — only document_id and building_id.

9. **Resident queries are personal data.** Store only user_id (UUID), never full names, alongside
   query text. Never put query text, emails, or names in application logs.

10. **Error responses never include raw error text.**

---

## Code rules

- Always write TypeScript. Never use plain JavaScript (.js files).
- Always add error handling to every API route. Never return a 500 with a raw error message.
- Use async/await. Never use callbacks or raw Promises without .catch().
- Never hardcode API keys, URLs, model names, or secrets. Always use process.env.VARIABLE_NAME.
- Every environment variable must exist, with a placeholder value, in the matching
  `apps/api/.env.example` or `apps/web/.env.example`.
- Keep route handlers thin — business logic belongs in services/.
- Add a JSDoc comment to every exported function in services/.

---

## Placeholder numbers (to be measured, not facts)

`RAG_CONFIDENCE_THRESHOLD=0.72`, the 20 queries/day limit, the 50 MB upload limit, and the
"≥30% fewer concierge questions" goal are placeholders. Don't treat them as validated.

---

## What Claude must never do

- Never commit a .env file or any file containing a real API key or secret.
- Never install a new npm package without asking first — list what you want to add and why.
- Never rename an existing database table or column without explicit instruction.
- Never change the system prompt wording without explicit instruction.
- Never skip the faithfulness check in the RAG pipeline.
- Never change the default model names without explicit instruction.
- Never expose the Supabase service role key to the frontend.

---

## How to run locally

```bash
# API (from apps/api/)
cp .env.example .env          # then fill in real values
npm install
npm run dev                   # starts on port 3000

# Web (from apps/web/)
cp .env.example .env          # then fill in real values
npm install
npm run dev                   # starts on port 5173
```

Secret scanning: install gitleaks (`brew install gitleaks`) and enable the repo hook once with
`git config core.hooksPath .githooks`.

---

## Git conventions — branch flow

```
feature/* ─┐
fix/*     ─┼─▶ PR ─▶ develop (staging) ─▶ release PR ─▶ main (live) ─▶ tag vX.Y.Z
           ┘
hotfix/* (from main) ─▶ PR ─▶ main, then merge main back into develop
```

- **Never commit or push directly to `develop` or `main`.** All work goes on a child branch first.
- Branch naming: `feature/short-description`, `fix/short-description`, `hotfix/short-description`.
- Feature and fix PRs target `develop`. Only release PRs (develop → main) and hotfixes target `main`.
- A release to `main` needs a **GO** from `release-manager` and the user's explicit approval.
  Tag every release with a semantic version (`v0.1.0`) and plain-language release notes.
- Commit messages: present tense, imperative — "Add login route" not "Added login route".
- The PR description must reference the GitHub issue number it closes.
- Never force-push to `develop` or `main`.
