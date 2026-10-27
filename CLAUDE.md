# ORVEXA — Claude Code Instructions

## What this project is

ORVEXA is a web application (a cross-platform mobile app follows after GA) that lets residential
building residents ask plain-English questions about building bylaws and get back accurate, cited
answers powered by RAG (Retrieval-Augmented Generation). Concierge staff upload documents;
residents query them via chat.

This is a real product used by real residents. Wrong answers can cause fines. Treat every
generation-layer decision with that in mind.

The product requirements live in `about-the-product/prd.html` (PDF: `ORVEXA-PRD.pdf`).

---

## How we work

The user is learning the whole product lifecycle through this project. Explain in plain,
kid-friendly language, and avoid jargon unless it's explained.

### Every section starts and ends with the journey map
- **Before** a section: show the relevant branch of the tree in `about-the-product/journey.md`
  with "📍 You are here" and one sentence on what this stage is and why it matters.
- **After** a section: update `about-the-product/journey.md` (statuses, section log, and agent
  scorecard) and show what changed (⬜ → 🔵 → ✅), which agents ran and what each found, and what
  comes next. Never count an agent as run unless it actually ran in that section.

### Plan → build → test → review
1. **Plan**: anything beyond a small fix needs the goal, the plan in plain English, the files
   you'll touch, 3 checks that will prove it works, and which agents will review. Run
   `engineering-manager` on the plan (and `ceo-reviewer` for a new feature or phase). Wait for "go".
2. **Build** on a `feature-*` or `fix-*` branch from `pre-main`, one small piece per commit.
3. **Test**: run `qa-tester`. Run each check for real and report pass/fail with the output.
   A check you didn't run stays unticked.
4. **Review** (agents in `.claude/agents/`):

   | Agent | Runs when |
   |---|---|
   | `code-reviewer` | Every PR |
   | `security-reviewer` | Auth, uploads, RLS, storage, env/config, personal data, or the RAG pipeline |
   | `rag-reviewer` | RAG services, the system prompt, model settings, or the eval set |
   | `frontend-designer` | Builds every screen; reviews any UI change |
   | `engineering-manager` | Every plan, and every PR before merging into pre-main |
   | `ceo-reviewer` | New feature or phase, and before the first release to real residents |
   | `release-manager` | Every merge from pre-main into main |

5. Fix every **must fix** finding before merging. Record any lasting decision in
   `about-the-product/decisions.md`.
6. Stop and ask before: a new package, a schema change, anything touching personal data,
   a production deploy, anything that costs money, or a legal/privacy claim.
7. Before any commit or push, check the change against the security rules below, and say what
   you checked.

### Memory: where things are remembered
- **CLAUDE.md** (this file): rules for every session and every collaborator.
- **`about-the-product/decisions.md`**: what we decided and why, so nothing is re-explained or re-argued.
- **`about-the-product/journey.md`**: where the project is in its lifecycle.
- **`about-the-product/architecture.md`**: free-plan limits, checked on the provider's page, with dates.
- **Claude's private memory**: the user's personal preferences across sessions.

---

## Tech stack — use exactly these, never substitute

Follows the global `tech-stack` skill (zero budget: $0 spend; ask before anything paid).

| Layer | Technology |
|---|---|
| Web app + API | Next.js (App Router) + TypeScript — located in apps/web/ (API = route handlers) |
| Mobile app (after GA) | React Native with Expo + TypeScript — apps/mobile/, calls the same API |
| Database | Supabase (Postgres + pgvector extension) |
| Auth | Supabase Auth — email + password, invite-only (open sign-up disabled). OAuth: later phase |
| File storage | Supabase Storage — private bucket, short-lived signed URLs |
| AI embeddings | Google Gemini, model from `GEMINI_EMBEDDING_MODEL` (default gemini-embedding-2), output size `GEMINI_EMBEDDING_DIMENSIONS` (1536) |
| AI rerank / generation | Google Gemini free tier, models from `GEMINI_RERANK_MODEL` / `GEMINI_GENERATION_MODEL` (default gemini-2.5-flash-lite) |
| AI faithfulness check | A **different, larger** Gemini free-tier model, `GEMINI_FAITHFULNESS_MODEL` (default gemini-2.5-flash), so the answer is never graded by the model that wrote it |
| Document parsing | pdf-parse (PDF), mammoth (.docx); OCR approach still open (see PRD) |
| Hosting | Vercel free (Hobby) plan — portfolio / non-commercial phase only |
| Planned, needs approval at install | Tailwind CSS, shadcn/ui, Vitest, Playwright |

---

## Where things live (one home per fact)

```
Orvexa_RAG/
├── about-the-product/          # PRD, decisions, journey map, architecture notes
├── designs-and-screens/        # User-flow diagrams (SVG), later app screenshots
├── apps/
│   ├── web/                    # Next.js web app + API
│   │   ├── .env.example        # Server-only vars + NEXT_PUBLIC_* browser-safe vars
│   │   └── src/
│   │       ├── app/            # Pages and API route handlers (thin)
│   │       └── server/         # Server-only code: never imported by browser components
│   │           ├── services/   # RAG pipeline, ingestion, Gemini calls
│   │           ├── prompts/
│   │           │   └── system.md   # THE system prompt — loaded at runtime
│   │           └── db/         # Supabase clients (per-user + service)
│   └── mobile/                 # Expo app (after GA); EXPO_PUBLIC_* vars only
├── supabase/migrations/        # Source of truth for the schema (once created)
├── .claude/                    # Claude Code settings + agents
├── CLAUDE.md
└── README.md
```

- **System prompt:** `apps/web/src/server/prompts/system.md`. Never edit its wording without
  explicit instruction. It must be bundled with the server build so it can be read at runtime.
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

## API endpoints (Next.js route handlers under apps/web/src/app/api/)

| Method | Path | Auth required | What it does |
|---|---|---|---|
| POST | /api/query | Yes (resident) | Run RAG pipeline, return answer + citation |
| POST | /api/feedback | Yes | Save thumbs up/down on a query_log |
| POST | /api/documents/upload | Yes (concierge) | Upload PDF or .docx, trigger ingestion |
| GET | /api/documents | Yes (concierge) | List documents for the concierge's building |
| GET | /api/escalations | Yes (concierge) | List open escalations |
| PATCH | /api/escalations/:id | Yes (concierge) | Mark escalation resolved |
| GET | /api/analytics/top-queries | Yes (concierge) | Top 20 most-queried topics (P2) |

There is no login endpoint. The apps sign in with Supabase Auth directly; passwords never pass
through the API. Every route reads the access token from the `Authorization: Bearer <token>`
header (so the future mobile app works the same way, without browser cookies) and verifies it
with `supabase.auth.getUser(token)`.

---

## RAG pipeline (apps/web/src/server/services/rag.ts)

The query flow must follow this exact order:

1. Embed the query.
2. Retrieve top `RAG_TOP_K_RETRIEVE` chunks from pgvector, filtered **in SQL** by the caller's
   building_id and status = 'active'.
3. Re-rank to the best `RAG_TOP_K_RERANK` with an LLM call.
4. Generate the answer with the system prompt from `prompts/system.md`.
5. Run a faithfulness check — does every claim in the answer exist in the retrieved chunks?
6. If faithfulness passes, return answer + citation. If it fails, hold the answer and create an escalation.

**If any model call fails or the AI quota runs out** at any step, return the single fallback
message (kept in one constant, not retyped) telling the resident to contact the concierge, and
create an escalation. During an outage, group these into one escalation per outage so the
concierge isn't flooded. Never return an answer that
skipped the faithfulness check. `RAG_DAILY_QUERY_LIMIT` × residents must fit inside the
free-tier daily quota recorded in `about-the-product/architecture.md`.

Prompt-injection controls (a bylaw PDF can contain "ignore previous instructions"):
- Wrap each chunk in clear delimiters and label it as reference text, not instructions.
- The building_id filter lives in SQL, never in the prompt.
- Check citations in code against the IDs of the chunks actually retrieved, not the citation text the model writes.
- The eval set must include one "poisoned" test document, known-unfaithful answers (to measure
  how often the faithfulness check misses them), and the failed-call / quota-exhausted paths.
  Re-run the eval set after any model change.
- Normalise embeddings to length 1 at ingestion and at query time, until Google's docs confirm
  whether 1536-dim `gemini-embedding-2` output is already normalised.

Log tokens in/out and latency for every model call (the `query_logs` columns exist for this).

---

## Privacy gate — blocking

Google's Gemini free tier may use content to improve Google's products. So, while ORVEXA runs on
the free tier:
- Use **only sample documents and test accounts**. No real resident, no real council or
  management member, no real building document, and no real emergency contact list.
- Before the internal alpha, or any real person or real document, Varun must first choose the
  paid tier or another option, and the decision is recorded in `about-the-product/decisions.md`.
- `ceo-reviewer` and `release-manager` must block any release that breaks this gate.

---

## Security rules — never violate these

1. **building_id and role come from the server, never from the request.** `DEFAULT_BUILDING_NAME`
   only names the sample building when seeding; it is never a fallback building_id.
   After verifying the token, look both up in the `users` table (or `app_metadata` / a custom
   access token hook). Never read them from `user_metadata` (users can edit it), the request body,
   or query params. Use the server-side user record only.

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

5. **Secrets never reach the browser or the phone.** `SUPABASE_SERVICE_ROLE_KEY` and
   `GEMINI_API_KEY` are read only in `apps/web/src/server/` and route handlers (mark those modules
   `server-only`), never in a client component. Nothing secret ever gets a `NEXT_PUBLIC_*` or
   `EXPO_PUBLIC_*` prefix — those values ship in public bundles. Locally they live in
   `apps/web/.env.local`; in production, in Vercel's environment settings. Check auth inside
   **every** route handler, never only in `middleware.ts`. Don't use Server Actions for data
   changes; if one is ever needed, it runs the same token, role and building checks.
   (The `server-only` npm package needs install approval like any other package.)

6. **File uploads:** Vercel functions accept at most 4.5 MB per request and run at most 300 s on
   the free plan (see `about-the-product/architecture.md`), so files never pass through a route
   handler. The concierge's browser uploads straight to the private bucket with a short-lived
   signed upload URL. The server then checks the real file type from its first bytes (magic
   bytes), not the extension — PDF and .docx only — and the size (`UPLOAD_MAX_MB`), and parses it
   from Storage with a hard timeout and a cap on unzipped size (a .docx is a zip and can be a zip
   bomb). Rejected files are deleted. If parsing can't fit the time limit, it moves to a Supabase
   Edge Function with the same checks.

7. **Rate limiting:** enforce `RAG_DAILY_QUERY_LIMIT` queries per user per day on /api/query, with
   the counter stored in Postgres (not memory — serverless functions don't keep memory). The
   check-and-increment is one atomic SQL statement, and RLS stops users from changing or deleting
   their own counter row.

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
  `apps/web/.env.example` (or `apps/mobile/.env.example` once it exists).
- Keep route handlers thin — business logic belongs in `src/server/services/`.
- Add a JSDoc comment to every exported function in services/.

---

## Placeholder numbers (to be measured, not facts)

`RAG_CONFIDENCE_THRESHOLD=0.72`, the 20 queries/day limit and the 50 MB upload limit are
placeholders. Don't treat them as validated. Metric targets are not set yet.

---

## What Claude must never do

- Never commit a .env file or any file containing a real API key or secret.
- Never install a new npm package without asking first — list what you want to add and why.
- Never sign up for, or switch to, anything paid without asking first.
- Never rename an existing database table or column without explicit instruction.
- Never change the system prompt wording without explicit instruction.
- Never skip the faithfulness check in the RAG pipeline.
- Never change the default model names without explicit instruction.
- Never expose the Supabase service role key or the Gemini API key to the browser or the app.

---

## How to run locally (once the app exists)

```bash
# Web app + API (from apps/web/)
cp .env.example .env.local    # then fill in real values
npm install
npm run dev                   # starts on port 3000
```

Secret scanning: install gitleaks (`brew install gitleaks`) and enable the repo hook once with
`git config core.hooksPath .githooks`.

---

## Git conventions — branch flow

```
feature-* ─┐
fix-*     ─┼─▶ PR ─▶ pre-main (staging) ─▶ release PR ─▶ main (live) ─▶ tag vX.Y.Z
           ┘
hotfix-* (from main) ─▶ PR ─▶ main, then merge main back into pre-main
```

- **Never commit or push directly to `pre-main` or `main`.** All work goes on a child branch first.
- **Never delete `main` or `pre-main`.**
- Branch naming: `feature-short-description`, `fix-short-description`, `hotfix-short-description`.
- Feature and fix PRs target `pre-main`. Only release PRs (pre-main → main) and hotfixes target `main`.
- A release to `main` needs a **GO** from `release-manager` and the user's explicit approval.
  Tag every release with a semantic version (`v0.1.0`) and plain-language release notes.
- Commit messages: present tense, imperative — "Add login route" not "Added login route".
- The PR description must reference the GitHub issue number it closes.
- Never force-push to `pre-main` or `main`.
