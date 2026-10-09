# ORVEXA

> A RAG-powered knowledge layer for residential buildings — gives residents instant, cited, grounded answers to building-specific bylaws and operational rules, without calling the concierge.

---

## What is ORVEXA?

ORVEXA is a web application (with a future mobile path) that lets residents of a residential building ask plain-English questions about building policies and get back accurate, cited answers grounded in the building's own bylaw documents.

Instead of emailing the concierge and waiting hours, a resident scans a QR code in the lobby, opens the ORVEXA web app, signs in, and asks: *"Can my guest park overnight in visitor stall 3?"* — and gets back the exact bylaw clause, with a citation and document date, in seconds.

Concierge staff get a separate dashboard to upload documents, manage residents, and review escalations.

---

## Who is it for?

| Role | What they do in ORVEXA |
|---|---|
| **Resident** | Asks bylaw and policy questions via chat; gets cited answers |
| **Concierge / Admin** | Uploads documents, reviews escalations, tests queries |
| **Property Manager** | (Phase 2) Cross-building analytics |

---

## Tech stack

| Layer | Technology |
|---|---|
| Frontend | React + TypeScript (Vite) |
| Backend API | Node.js + Express + TypeScript |
| Database | Supabase (Postgres + pgvector) |
| Auth | Supabase Auth (email + password, invite-only) |
| File storage | Supabase Storage |
| AI — embeddings | OpenAI text-embedding-3-small |
| AI — generation | OpenAI GPT-4o Mini |
| Frontend hosting | Vercel |
| API hosting | Railway |

---

## Project structure

```
Orvexa_RAG/
├── apps/
│   ├── web/                # React frontend (Vite + TypeScript)
│   │   └── .env.example    # Browser-safe env vars (VITE_*)
│   └── api/                # Node.js + Express backend
│       └── .env.example    # Server-only env vars (secrets)
├── .claude/                # Claude Code permissions + review agents
├── docs/
│   ├── journey.md          # Where the project is in its lifecycle
│   └── decisions.md        # What we decided and why
├── .github/workflows/      # CI (secret scanning)
├── .githooks/              # Local pre-commit secret scan
├── .gitignore
├── CLAUDE.md               # Instructions for Claude Code
├── LICENSE
└── README.md               # You are here
```

---

## Getting started (local development)

### Prerequisites
- Node.js 20+
- A Supabase project (free at supabase.com)
- An OpenAI API key (platform.openai.com)

### Steps

```bash
# 1. Clone the repo
git clone https://github.com/VarunSagarK/Orvexa_RAG.git
cd Orvexa_RAG

# 2. Enable the pre-commit secret scan (needs: brew install gitleaks)
git config core.hooksPath .githooks

# 3. Copy each env template and fill in your keys
cp apps/api/.env.example apps/api/.env
cp apps/web/.env.example apps/web/.env

# 4. Install dependencies
cd apps/api && npm install
cd ../web && npm install

# 5. Start the API (from apps/api/)
npm run dev

# 6. Start the web app (from apps/web/)
npm run dev
```

The web app runs at `http://localhost:5173`
The API runs at `http://localhost:3000`

---

## Environment variables

There are two templates, kept apart on purpose:

- `apps/api/.env.example` — **server only**: `SUPABASE_URL`, `SUPABASE_ANON_KEY`,
  `SUPABASE_SERVICE_ROLE_KEY` (ingestion only, bypasses RLS), `OPENAI_API_KEY`, model names, RAG settings.
- `apps/web/.env.example` — **browser-visible**: `VITE_SUPABASE_URL`, `VITE_SUPABASE_ANON_KEY`, `VITE_API_URL`.
  Anything prefixed `VITE_` ships in the public JavaScript bundle, so never put a secret there.

---

## Deployment

| What | Where | Command |
|---|---|---|
| Web app — staging | Vercel | Auto-deploys on merge to `develop` |
| Web app — live | Vercel | Auto-deploys on release to `main` |
| API — staging | Railway | Auto-deploys on merge to `develop` |
| API — live | Railway | Auto-deploys on release to `main` |
| Database | Supabase cloud | Migrations applied as part of a release |

Branch flow: `feature/*` → PR → `develop` (staging) → release PR → `main` (live). See `CLAUDE.md`.

---

## Roadmap

- **Alpha** — one pilot building, concierge team testing only
- **Beta** — 3–5 buildings, 15–20 residents per building
- **GA** — full rollout; concierge query volume measurably down (target ≥ 30% — to be measured)
- **Mobile** — Expo wrapper for iOS/Android App Store submission

---

## License

MIT — see `LICENSE`
