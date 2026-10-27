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
├── 2. PRODUCT PLANNING — "What exactly do we build, for whom?"                    ✅
│     ├── Users & roles (resident, concierge, property manager)                      ✅
│     ├── Features & endpoints (what the app can do)                                 ✅
│     ├── Roadmap (Prototype → Alpha → Beta → GA → Mobile)                           ✅
│     ├── Success metrics (5 metrics in the PRD; targets set after the alpha)        ✅
│     └── PRD (product requirements document) in about-the-product/                  ✅
│
├── 3. DESIGN — "How will it work and look?"                                       🔵
│     ├── Architecture (Next.js web + API → Supabase + Gemini; Expo later)           ✅
│     ├── Data model (the 6 database tables)                                         ✅
│     ├── Security design / threat model (issue #1 review)                           ✅
│     ├── RAG pipeline design (embed → retrieve → rerank → answer → check)          ✅
│     └── UX / UI design (screens, flows, design tokens)                             ⬜
│
├── 4. PROJECT SETUP — "Get the workshop ready"                                    🔵
│     ├── Git repo + .gitignore                                                      ✅
│     ├── Branch flow: feature-* → pre-main → main                                   ✅
│     ├── Env templates (secret vs public)                                           ✅
│     ├── Rules for Claude (CLAUDE.md) + memory                                      ✅
│     ├── AI agent team (.claude/agents/)                                            ✅
│     ├── Zero-budget tech stack (global skill) + switch                             🔵
│     ├── CI: automatic secret scanning                                              ✅
│     ├── Branch protection on GitHub                                                ⬜  (you)
│     └── Accounts: Supabase, Google AI Studio (Gemini), Vercel                      ⬜  (you)
│
├── 5. BUILD — "Write the code"                                                 ⬜ 🔁
│     ├── Database (migrations, RLS policies)                                        ⬜
│     ├── Backend API (auth middleware, routes, services)                            ⬜
│     ├── Spike: can PDF parsing fit Vercel's free time limit?                       ⬜
│     ├── Ingestion (upload → parse → chunk → embed)                                 ⬜
│     ├── RAG pipeline                                                               ⬜
│     └── Frontend (resident chat, concierge dashboard)                              ⬜
│           (after each app ships: update the PRD's Rollout + Prototype sections)
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
│     ├── Merge feature → pre-main → staging site                                    ⬜
│     ├── Privacy gate: off the AI free tier before any real person or document      ⬜
│     ├── Release Manager go/no-go + version number (v0.1.0) + release notes         ⬜
│     └── Merge pre-main → main                                                      ⬜
│
├── 9. DEPLOYMENT — "Put it on the internet"                                    ⬜ 🔁
│     ├── Web app + API on Vercel (staging from pre-main, live from main)            ⬜
│     └── Database on Supabase cloud (migrations applied)                            ⬜
│
├── 10. OPERATE & MONITOR — "Keep it healthy"                                   ⬜ 🔁
│     ├── Logs, errors, uptime alerts                                                ⬜
│     ├── AI quota + cost tracking, spend cap                                        ⬜
│     ├── Incident response + rollback                                               ⬜
│     └── Key rotation, log retention, access reviews                                ⬜
│
└── 11. FEEDBACK & SCALE — "Learn and grow"                                     ⬜ 🔁
      ├── Thumbs up/down, escalations, top questions → improvements                  ⬜
      ├── Internal alpha → Closed beta (15–20 residents) → GA                        ⬜
      └── Mobile app (Expo)                                                          ⬜
```

🔁 Steps 5–11 repeat for **every feature**: build a little → test → review → release → deploy →
watch → learn → repeat. That loop is how a project grows from one building to hundreds.

---

## The pipeline (how one piece of work travels)

```
 idea/issue ─▶ plan ─▶ feature-xyz ─▶ tests ─▶ reviews ─▶ PR ─▶ pre-main ─▶ staging site
                 │                                                    │
            CEO + Eng Mgr                                     human tries it
                                                                       │
                                     live site ◀─ main ◀─ Release Manager GO
```

---

## Agent scorecard

What each agent has done so far, and how useful it has been. Updated after every section.

| Agent | Runs | Verdicts (pass / changes) | Must-fix found | Fixed | False alarms | Escaped |
|---|---|---|---|---|---|---|
| qa-tester | 1 | 1 / 0 | 0 | – | 0 | – |
| frontend-designer | 0 | – | – | – | – | – |
| code-reviewer | 2 | 0 / 2 | 6 | 6 | 0 | – |
| rag-reviewer | 1 | 0 / 1 | 1 | 1 | 0 | – |
| security-reviewer | 1 | 0 / 1 | 2 | 2 | 0 | – |
| engineering-manager | 2 | 0 / 2 | 11 | 10 (vision sign-off is Varun's) | 0 | – |
| ceo-reviewer | 1 | 0 / 1 | 5 | 4 (cost math kept by owner's decision) | 0 | – |
| release-manager | 0 | – | – | – | – | – |

**How to read it**
- **Must-fix found / Fixed**: real problems the agent caught, and how many we fixed.
- **False alarms**: findings that turned out not to be problems. Too many means the agent's
  checklist needs tightening.
- **Escaped**: bugs found *later* (by another agent, a test, a human, or a user) that this agent
  should have caught. **This is the real success measure.** An agent that approves everything
  has a 100% "pass rate" and is useless; an agent with zero escapes is doing its job.
- **Catch rate** = must-fix found ÷ (must-fix found + escaped), shown once there's enough data.

---

## Section log

| Date | Section | Stage(s) covered |
|---|---|---|
| 2026-10-08 | Added README, CLAUDE.md, env template, .gitignore | 2 Planning, 4 Setup |
| 2026-10-08 | Fixed security review (issue #1) | 3 Design (security), 4 Setup |
| 2026-10-09 | Branch flow, agent team, this map | 4 Setup |
| 2026-10-26 | Zero-budget stack skill; PRD rewrite + README; stack switch; branches → pre-main/main | 2 Planning, 3 Design, 4 Setup |
