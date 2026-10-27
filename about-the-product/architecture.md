# ORVEXA — Architecture notes and free-plan limits

The stack and rules live in `CLAUDE.md`; the product picture is in `prd.html` (Section 3).
This file holds the **facts checked on providers' own pages**, with the date, because free-plan
limits change. Never quote a limit from memory; check it and add a row.

## Checked

| Date | Provider | Fact | Source |
|---|---|---|---|
| 2026-10-26 | Google Gemini | Gemini 2.5 Flash-Lite: free tier input and output "Free of charge"; paid $0.10 in / $0.40 out per 1M tokens | ai.google.dev/gemini-api/docs/pricing |
| 2026-10-26 | Google Gemini | `gemini-embedding-2`: free tier "Free of charge"; paid $0.20 per 1M input tokens | ai.google.dev/gemini-api/docs/pricing |
| 2026-10-26 | Google Gemini | Free tier content is "Used to improve our products" (paid: not used) | ai.google.dev/gemini-api/docs/pricing |
| 2026-10-26 | Google Gemini | `gemini-embedding-2` defaults to 3072 dims, supports 128–3072, recommends 768 / 1536 / 3072 | ai.google.dev/gemini-api/docs/embeddings |
| 2026-10-26 | OpenAI (prototype reference) | GPT-4o Mini $0.15 in / $0.60 out per 1M tokens; text-embedding-3-small $0.02 per 1M | developers.openai.com/api/docs/pricing |

## Not checked yet (check before building the part that depends on it)

| Question | Needed for |
|---|---|
| Gemini free-tier requests per minute and per day for each model | `RAG_DAILY_QUERY_LIMIT` must fit inside it (4 model calls per question) |
| Vercel Hobby function time limit and request body size | Whether PDF/.docx parsing (up to `UPLOAD_MAX_MB`) fits, or moves to a Supabase Edge Function |
| Supabase free plan: database size, storage size, inactivity pause | Demo stays awake; documents fit |
| Whether shortened (1536-dim) Gemini embeddings need normalising before cosine search | Retrieval quality (ask `rag-reviewer`) |
