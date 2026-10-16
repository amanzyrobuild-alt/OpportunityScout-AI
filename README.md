# OpportunityScout AI

> An AI-powered autonomous opportunity-discovery agent that helps students
> discover, evaluate, and track **hackathons, internships, scholarships,
> fellowships, and technology competitions**.

Built with a **React + TypeScript + Tailwind** frontend, a **Python +
FastAPI** backend, **SQLite**, **APScheduler** for periodic checks, and an
optional **LLM** (OpenAI-compatible) for richer drafting. It works fully
*without* any API key — matching and drafting fall back to transparent,
rule-based logic.

---

## ✨ Features

| # | Feature | Status |
|---|---------|--------|
| 1 | **AI Opportunity Discovery** — pulls from permitted public sources & official APIs | ✅ Devpost (real) + configurable HTML + labeled demo |
| 2 | **Personalized Matching** — scores opportunities against skills/interests/location/experience | ✅ transparent weighted score |
| 3 | **Eligibility Checker** — eligible / ineligible / verify, with reasons | ✅ rule-based, never invents rules |
| 4 | **Deadline Tracker** — stores deadlines, sorts by urgency, reminders | ✅ dashboard + scheduler watch |
| 5 | **AI Application Assistant** — drafts project descriptions, problem statements, team intros, answers | ✅ LLM or template fallback |
| 6 | **Opportunity Dashboard** — searchable, filterable, match scores, deadline & demo badges | ✅ |
| 7 | **Duplicate Detection** — links the same opp across sources | ✅ similarity + date heuristic |
| 8 | **Human Approval** — explicit gate before submitting / sharing info | ✅ approval queue |

---

## 🧱 Project structure

```
opportunityscout/
├── .env.example                 # template for all configuration
├── requirements.txt             # backend Python dependencies
├── README.md
├── backend/
│   ├── .env                     # (created from .env.example)
│   ├── main.py                  # FastAPI app + SPA static serving
│   ├── config.py                # settings (pydantic-settings)
│   ├── database.py              # SQLite schema + helpers
│   ├── schemas.py               # Pydantic API models
│   ├── utils.py                 # date parsing, normalization, JSON helpers
│   ├── matcher.py               # matching score + eligibility engine
│   ├── deduplication.py         # cross-source duplicate linking
│   ├── discovery.py             # discovery pipeline (sources -> normalize -> dedup -> DB)
│   ├── ai_assistant.py          # draft generation (LLM or template)
│   ├── scheduler_runner.py      # APScheduler jobs
│   └── scrapers/
│       ├── base.py              # Source adapter interface
│       ├── devpost.py           # REAL source: Devpost public API
│       ├── demo_data.py         # Clearly-labeled DEMO sample data
│       └── html_source.py       # Configurable permitted-HTML scraper
└── frontend/
    ├── package.json
    ├── vite.config.ts           # dev proxy /api -> :8000
    ├── tailwind.config.js
    ├── index.html
    └── src/
        ├── main.tsx, App.tsx, index.css
        ├── types.ts
        ├── lib/ (api.ts, format.ts, refresh.ts)
        ├── components/ (Layout, OpportunityCard, MatchScore, DeadlineBadge,
        │               EligibilityBadge, Modal, Spinner, icons)
        └── pages/ (Dashboard, OpportunityDetail, Saved, Deadlines,
                    Drafts, Approvals, Profile)
```

---

## 🚀 Installation & setup

### Prerequisites
- Python 3.11+
- Node.js 18+ and npm

### 1. Backend
```bash
cd backend
python3 -m venv .venv && source .venv/bin/activate   # optional
pip install -r ../requirements.txt
cp ../.env.example .env          # edit if you want an LLM key / demo toggle
python3 -m uvicorn main:app --host 0.0.0.0 --port 8000
```
The backend auto-creates the SQLite DB, runs an initial discovery in the
background, and (if the frontend is built) serves the whole app from `http://localhost:8000`.

### 2. Frontend (two options)

**Option A — run the built app through the backend (recommended for the demo):**
```bash
cd frontend
npm install
npm run build                 # outputs to frontend/dist
# backend now serves it at http://localhost:8000
```

**Option B — Vite dev server with hot reload:**
```bash
cd frontend
npm install
npm run dev                   # http://localhost:5173 (proxies /api -> :8000)
```

### 3. Optional — enable the LLM
Copy `.env` and set:
```ini
LLM_PROVIDER=auto
OPENAI_API_KEY=sk-...
OPENAI_BASE_URL=https://api.openai.com/v1   # any OpenAI-compatible endpoint
OPENAI_MODEL=gpt-4o-mini
```
Without a key the app stays fully functional in **rule-based mode**.

---

## 🖥️ Demonstration workflow

1. **Open the app** → you land on *Discover*. A banner prompts you to
   complete your profile (matching is neutral until then).
2. **Complete profile** (`/profile`): add skills (`Python, Machine Learning`),
   interests, preferred categories, eligible countries, experience level.
3. **Discover** auto-ran on startup; click **Run discovery** to refresh from
   sources. The sidebar shows total opportunities and an AI-on/off indicator.
4. **Browse & filter** the dashboard by search text, category, source, remote-only,
   and sort (best match / deadline / recent / prize). Each card shows a
   **match score**, **deadline badge**, **eligibility badge**, and a **DEMO**
   flag for sample items.
5. **Open an opportunity** → see the transparent *“Why this match?”* reasons,
   the **eligibility check** (eligible / ineligible / verify with per-rule
   explanations), themes, prize, and the official application link.
6. **Save** it → it appears under *Saved* and *Deadlines* (countdown by urgency).
7. **Generate a draft** → pick a draft type (project description, problem
   statement, team intro, application answers). The assistant builds it from
   *your* profile + the opportunity’s real facts. Edit & save it under *Drafts*.
8. **Submit application** → opens the **human-approval** modal. Nothing is sent
   externally until you explicitly **Approve**. The request waits in
   *Approvals* (pending → approved/rejected).

> The demo ships with **real, live hackathons from Devpost** (54 at last run)
> plus a small, clearly-labeled **DEMO** dataset so non-hackathon categories
> (internships, scholarships, fellowships, grants, competitions) are visible.
> Demo items are always badged `DEMO` and carry **no invented application
> links or deadlines presented as real**. Disable them with
> `ENABLE_DEMO_DATA=false`.

---

## 🤖 AI agent workflow (as specified)

1. Read the user’s saved preferences and skills → `GET /api/profile`
2. Discover opportunities from configured sources → `POST /api/discover`
3. Extract & normalize details → `scrapers/*._normalize`
4. Check eligibility & missing info → `matcher.evaluate` (rule-based)
5. Calculate a transparent relevance score → `matcher.evaluate` (weighted sum + reasons)
6. Rank & display best matches → `GET /api/opportunities?sort=match`
7. Save selected opportunities & track deadlines → `POST /api/opportunities/:id/save`
8. Generate application drafts for review → `POST /api/opportunities/:id/draft`
9. Request explicit approval before external submission → `POST /api/approvals`

---

## 🔐 Trust & safety guarantees

- **No invented data.** Real items come from Devpost’s public API. Demo items
  are explicitly flagged and carry no real links/deadlines. Eligibility rules
  are derived only from observable facts; missing facts prompt *“verify”*
  rather than guessing.
- **Human approval gate.** Submitting applications or sharing personal
  information always creates a pending approval you must approve.
- **Rate-limit / terms respect.** The Devpost adapter sends a normal browser
  `User-Agent`, fetches a bounded number of pages per run, and never hammers
  the endpoint. The scheduler runs discovery at most every
  `DISCOVERY_INTERVAL_MINUTES` (default 360).
- **Configurable permitted scraping.** Add your own public pages via
  `CONFIGURED_SOURCES` (name|url|item|title|link|date|category). Only scrape
  sources you are permitted to.

---

## 🗄️ Database schema (SQLite)

Tables: `profile`, `opportunities`, `saved_opportunities`, `deadlines`,
`application_drafts`, `approval_requests`, `discovery_runs`, `schema_meta`.
Key columns on `opportunities`: `source`, `external_id` (unique per source),
`category`, `themes` (JSON), `deadline`, `is_demo`, `duplicate_of`,
`application_url`. Created automatically on first run.

---

## 🚢 Deployment

- **Frontend → Vercel:** import `frontend/`, build command `npm run build`,
  output `dist`. For production, point the API base at your backend and add a
  rewrite `/api/* → <backend-url>/api/*` (or keep same-origin by serving the
  build from the backend as shown here).
- **Backend → any Python host** (Render, Fly.io, Railway, Railway, a VM):
  `pip install -r requirements.txt` then
  `uvicorn main:app --host 0.0.0.0 --port $PORT`. Use a persistent disk for
  `opportunityscout.db`. Set `CORS_ORIGINS` to your Vercel URL if the frontend
  is hosted separately.

---

## ❗ Error handling

- Network/source failures are caught per-source; discovery logs the error and
  continues with whatever it gathered (`discovery_runs.error` is recorded).
- Missing/invalid opportunity fields are left `null` (never faked).
- All API errors return structured `{"detail": ...}` with appropriate HTTP codes.
- The LLM draft path falls back to the template engine on any API error.
