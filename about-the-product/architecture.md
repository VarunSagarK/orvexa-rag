# ORVEXA — Architecture notes and free-plan limits

The stack and rules live in `the-app/CLAUDE.md`; the product picture is in `prd.html` (Section 3).
This file holds the **facts checked on providers' own pages**, with the date, because free-plan
limits change. Never quote a limit from memory; check it and add a row.

## Checked

| Date | Provider | Fact | Source |
|---|---|---|---|
| 2026-10-26 | Google Gemini | Gemini 2.5 Flash-Lite: free tier input and output "Free of charge"; paid $0.10 in / $0.40 out per 1M tokens | ai.google.dev/gemini-api/docs/pricing |
| 2026-10-26 | Google Gemini | Gemini 2.5 Flash: free tier input and output "Free of charge"; paid $0.30 in / $2.50 out per 1M tokens | ai.google.dev/gemini-api/docs/pricing |
| 2026-10-26 | Google Gemini | `gemini-embedding-2`: free tier "Free of charge"; paid $0.20 per 1M input tokens | ai.google.dev/gemini-api/docs/pricing |
| 2026-10-26 | Google Gemini | Free tier content is "Used to improve our products" (paid: not used) | ai.google.dev/gemini-api/docs/pricing |
| 2026-10-26 | Google Gemini | `gemini-embedding-2` defaults to 3072 dims, supports 128–3072, recommends 768 / 1536 / 3072 | ai.google.dev/gemini-api/docs/embeddings |
| 2026-10-26 | Vercel (Hobby) | Function request/response body max **4.5 MB** (else 413); max duration **300 s** (default and max); memory 2 GB | vercel.com/docs/functions/limitations (page updated 2026-08-24) |
| 2026-10-26 | OpenAI (prototype reference) | GPT-4o Mini $0.15 in / $0.60 out per 1M tokens; text-embedding-3-small $0.02 per 1M | developers.openai.com/api/docs/pricing |

## Not checked yet (check before building the part that depends on it)

| Question | Needed for |
|---|---|
| Gemini free-tier requests per minute and per day for each model | `RAG_DAILY_QUERY_LIMIT` must fit inside it (4 model calls per question) |
| Whether parsing a large PDF/.docx from Storage fits in 300 s | Spike before building ingestion; fallback is a Supabase Edge Function |
| Supabase free plan: database size, storage size, inactivity pause | Demo stays awake; documents fit |
| Whether 1536-dim `gemini-embedding-2` output is already normalised (pgvector's cosine `<=>` is unaffected; inner product, L2 and score thresholds are) | Retrieval scores vs `RAG_CONFIDENCE_THRESHOLD`; normalise anyway until checked |
