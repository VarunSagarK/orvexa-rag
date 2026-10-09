# ORVEXA — Project Journey Map

How a software product goes from an idea to something real people use, and where ORVEXA is
right now. Updated at the start and end of every work section.

**Legend:** ✅ done · 🔵 in progress · ⬜ not started · 🔁 repeats every feature

---

## The whole journey (Product + Software Development Life Cycle)

```
ORVEXA
│
├── 1. IDEATION — "What problem are we solving?"                                   ✅
│     ├── Problem: residents wait hours for the concierge to answer bylaw questions  ✅
│     └── Idea: an AI that answers instantly, with the exact bylaw as proof           ✅
│
├── 2. PRODUCT PLANNING — "What exactly do we build, for whom?"                    🔵
│     ├── Users & roles (resident, concierge, property manager)                      ✅
│     ├── Features & endpoints (what the app can do)                                 ✅
│     ├── Roadmap (Alpha → Beta → GA → Mobile)                                       ✅
│     ├── Success metrics (e.g. ≥30% fewer concierge questions — to be measured)     ✅
│     └── PRD (product requirements document) in docs/                               ⬜
│
├── 3. DESIGN — "How will it work and look?"                                       🔵
│     ├── Architecture (React → Express API → Supabase + OpenAI)                     ✅
│     ├── Data model (the 6 database tables)                                         ✅
│     ├── Security design / threat model (issue #1 review)                           ✅
│     ├── RAG pipeline design (embed → retrieve → rerank → answer → check)          ✅
│     └── UX / UI design (screens, flows, design tokens)                             ⬜
│
├── 4. PROJECT SETUP — "Get the workshop ready"                                    🔵
│     ├── Git repo + .gitignore                                                      ✅
│     ├── Branch flow: feature → develop → main                                      🔵
│     ├── Env templates (secret vs public)                                           ✅
│     ├── Rules for Claude (CLAUDE.md) + memory                                      ✅
│     ├── AI agent team (.claude/agents/)                                            🔵
│     ├── CI: automatic secret scanning                                              ✅
│     ├── Branch protection on GitHub                                                ⬜  (you)
│     └── Accounts: Supabase, OpenAI, Vercel, Railway                                ⬜  (you)
│
├── 5. BUILD — "Write the code"                                                 ⬜ 🔁
│     ├── Database (migrations, RLS policies)                                        ⬜
│     ├── Backend API (auth middleware, routes, services)                            ⬜
│     ├── Ingestion (upload → parse → chunk → embed)                                 ⬜
│     ├── RAG pipeline                                                               ⬜
│     └── Frontend (resident chat, concierge dashboard)                              ⬜
│
├── 6. TESTING — "Prove it works, then try to break it"                         ⬜ 🔁
│     ├── Unit tests — one small piece at a time                                     ⬜
│     ├── Integration tests — pieces working together (API + database)               ⬜
│     ├── Tenant-isolation tests — Building A can't see Building B                   ⬜
│     ├── RAG evals — are answers correct, cited, and refusing when they should?     ⬜
│     ├── Security tests — secrets, injection, permissions                           ⬜
│     ├── End-to-end tests — a robot clicks through the real app                     ⬜
│     ├── Accessibility tests — usable by everyone                                   ⬜
│     ├── Performance / load tests — fast with many users                            ⬜
│     └── User acceptance testing (UAT) — real concierge staff try it                ⬜
│
├── 7. REVIEW — "Someone else checks the work"                                  ⬜ 🔁
│     ├── Code review · Security review · RAG review · Design review                 ⬜
│     └── Leadership: CEO (right thing?) · Eng Manager (done right?)                 ⬜
│
├── 8. RELEASE — "Decide it's ready to go live"                                 ⬜ 🔁
│     ├── Merge feature → develop → staging site                                     ⬜
│     ├── Release Manager go/no-go + version number (v0.1.0) + release notes         ⬜
│     └── Merge develop → main                                                       ⬜
│
├── 9. DEPLOYMENT — "Put it on the internet"                                    ⬜ 🔁
│     ├── Frontend on Vercel (staging from develop, live from main)                  ⬜
│     ├── API on Railway (staging from develop, live from main)                      ⬜
│     └── Database on Supabase cloud (migrations applied)                            ⬜
│
├── 10. OPERATE & MONITOR — "Keep it healthy"                                   ⬜ 🔁
│     ├── Logs, errors, uptime alerts                                                ⬜
│     ├── AI cost tracking + spend cap                                               ⬜
│     ├── Incident response + rollback                                               ⬜
│     └── Key rotation, log retention, access reviews                                ⬜
│
└── 11. FEEDBACK & SCALE — "Learn and grow"                                     ⬜ 🔁
      ├── Thumbs up/down, escalations, top questions → improvements                  ⬜
      ├── Alpha (1 building) → Beta (3–5) → GA (all)                                 ⬜
      └── Mobile app (Expo)                                                          ⬜
```

🔁 Steps 5–11 repeat for **every feature**: build a little → test → review → release → deploy →
watch → learn → repeat. That loop is how a project grows from one building to hundreds.

---

## The pipeline (how one piece of work travels)

```
 idea/issue ─▶ plan ─▶ feature/xyz ─▶ tests ─▶ reviews ─▶ PR ─▶ develop ─▶ staging site
                 │                                                    │
            CEO + Eng Mgr                                     human tries it
                                                                       │
                                     live site ◀─ main ◀─ Release Manager GO
```

---

## Section log

| Date | Section | Stage(s) covered |
|---|---|---|
| 2026-10-08 | Added README, CLAUDE.md, env template, .gitignore | 2 Planning, 4 Setup |
| 2026-10-08 | Fixed security review (issue #1) | 3 Design (security), 4 Setup |
| 2026-10-09 | Branch flow, agent team, this map | 4 Setup |
