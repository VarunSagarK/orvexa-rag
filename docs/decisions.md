# ORVEXA — Decision Log

Important decisions and *why* we made them, so nobody (human or Claude) has to re-explain or
re-argue them. Newest first. Add an entry whenever a choice would surprise a newcomer.

| Date | Decision | Why |
|---|---|---|
| 2026-10-09 | Branch flow is `feature/*` → `develop` (staging) → `main` (live). `Root` retired. | Every change gets tested on staging before residents see it; matches what Vercel, Railway and GitHub expect. |
| 2026-10-09 | Specialised review agents in `.claude/agents/` (QA, design, code, RAG, security, CEO, Eng Manager, Release Manager). | Independent checks catch what the builder misses; each has a focused checklist. |
| 2026-10-08 | API uses a per-request Supabase client from the user's token; service role key only for ingestion. | The service key bypasses Row Level Security, so RLS would protect nothing otherwise (issue #1). |
| 2026-10-08 | `building_id` and `role` are looked up server-side, never taken from the request or `user_metadata`. | Users can edit `user_metadata`, so they could switch buildings or make themselves concierge (issue #1). |
| 2026-10-08 | Separate env templates for `apps/api` (secret) and `apps/web` (public `VITE_*`). | `VITE_*` values ship in the public JavaScript bundle (issue #1). |
| 2026-10-08 | No `/auth/login` endpoint; the web app signs in with Supabase directly. | Passwords shouldn't pass through our API for no benefit (issue #1). |
| 2026-10-08 | Model names live in env vars (default GPT-4o Mini). | Easy to change or test models without editing code (issue #1). |
