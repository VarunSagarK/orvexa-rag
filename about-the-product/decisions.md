# ORVEXA — Decision Log

Important decisions and *why* we made them, so nobody (human or Claude) has to re-explain or
re-argue them. Newest first. Add an entry whenever a choice would surprise a newcomer.

| Date | Decision | Why |
|---|---|---|
| 2026-10-26 | Repo follows the global repo-organizer layout: code + `CLAUDE.md` in `the-app/` (web in `the-app/website/`, mobile later in `the-app/mobile/`), journey in `project-log/`, hooks in `.github/hooks/`. README headings have no numbers. | Everything sits in a plain-named folder; only files tools require stay at the top. Varun chose `the-app/CLAUDE.md` knowing Claude Code auto-loads it only when working in `the-app/`. |
| 2026-10-26 | Faithfulness check uses `gemini-2.5-flash`, a different and larger model than the one that writes answers (`gemini-2.5-flash-lite`). | A model checking its own work tends to miss its own mistakes (rag-reviewer). Both are on the free tier. |
| 2026-10-26 | Stack: Next.js (web + API) on Vercel's free plan; Expo mobile after GA; Supabase unchanged. | Follows the global zero-budget `tech-stack` skill; no app code existed, so switching was cheap. Railway has no lasting free plan. |
| 2026-10-26 | AI: Google Gemini free tier — `gemini-embedding-2` at 1536 dims, `gemini-2.5-flash-lite` for rerank/answer/faithfulness (env vars). | Zero budget. 1536 dims keeps `chunks.embedding vector(1536)` unchanged, so no schema change. |
| 2026-10-26 | Privacy gate (blocking): free tier only with sample documents and test accounts; move off it before the internal alpha or any real person or document. | Google's free tier may use content to improve its products; resident questions are personal data. |
| 2026-10-26 | Any failed model call or exhausted quota → "contact the concierge" + escalation; never an unchecked answer. | A missing faithfulness check could give a wrong rule and cause a fine (CEO review). |
| 2026-10-26 | Login: invite-only now, OAuth later, no guest login. | A guest has no building, so no building's rules can be shown safely. |
| 2026-10-26 | Portfolio / learning project for now (Vercel Hobby is non-commercial). | Check hosting terms before any commercial use. |
| 2026-10-26 | Web first; mobile (Expo) after GA. | Smallest version that proves the value (CEO review); the API takes Bearer tokens so the app can reuse it. |
| 2026-10-26 | Branch flow: `feature-*` → `pre-main` → `main`; `Root` and `develop` deleted after merging. | Matches the global `repo-organizer` skill. |
| 2026-10-26 | Product docs live in `about-the-product/` (PRD, decisions, journey, architecture). | One home per fact, matching the global skills. |
| 2026-10-09 | *(Replaced 2026-10-26)* Branch flow was `feature/*` → `develop` → `main`. | Every change gets tested on staging before residents see it; matches what Vercel, Railway and GitHub expect. |
| 2026-10-09 | Specialised review agents in `.claude/agents/` (QA, design, code, RAG, security, CEO, Eng Manager, Release Manager). | Independent checks catch what the builder misses; each has a focused checklist. |
| 2026-10-08 | API uses a per-request Supabase client from the user's token; service role key only for ingestion. | The service key bypasses Row Level Security, so RLS would protect nothing otherwise (issue #1). |
| 2026-10-08 | `building_id` and `role` are looked up server-side, never taken from the request or `user_metadata`. | Users can edit `user_metadata`, so they could switch buildings or make themselves concierge (issue #1). |
| 2026-10-08 | Secret vs public env vars kept apart (now: server-only vs `NEXT_PUBLIC_*` in `the-app/website`). | Public-prefixed values ship in the JavaScript bundle (issue #1). |
| 2026-10-08 | No `/auth/login` endpoint; the web app signs in with Supabase directly. | Passwords shouldn't pass through our API for no benefit (issue #1). |
| 2026-10-08 | Model names live in env vars (default was GPT-4o Mini; now Gemini, see 2026-10-26). | Easy to change or test models without editing code (issue #1). |
