# AI-Driven Crime Analytics & Visualization Platform (Demo Build)

A hackathon-ready, full-stack crime intelligence platform: dark/glassmorphism
UI, animated dashboards, an AI chat assistant with voice I/O, criminal
network graphs, geo hotspot maps, offender profiling, financial crime
tracing, and simple ML-style forecasting — all running against a realistic
synthetic dataset with zero external API keys required.

## What's built vs. stubbed

| Module                          | Status |
|----------------------------------|--------|
| Auth (JWT) + 5-role RBAC         | ✅ Fully working |
| Dashboard stats + charts         | ✅ Fully working (live SQL aggregates) |
| Conversational AI chat + memory  | ✅ Working demo engine; **OpenAI plug-in point marked** in `backend/app/ai_service.py` |
| Voice assistant (speech-to-text/text-to-speech) | ✅ Uses browser Web Speech API (free, no key) |
| Criminal network graph (Cytoscape.js) | ✅ Fully working, generated from synthetic links |
| Gang/cluster detection           | ✅ Simplified shared-connection heuristic |
| Geo map + hotspots (Leaflet)      | ✅ Fully working |
| Global search                    | ✅ Fully working |
| Offender profiling                | ✅ Fully working, AI-generated summary text |
| Financial crime / money trail     | ✅ Fully working table + graph endpoint |
| Crime forecasting                 | ✅ Working moving-average model (swap for Prophet/ARIMA/sklearn later) |
| Explainable AI (evidence, reasoning, confidence) | ✅ Returned on every chat answer |
| Admin panel (users, CSV upload, audit logs) | ✅ Fully working |
| Notifications                     | ✅ Live-derived feed (polling, not WebSocket push) |
| Kannada / multi-language UI       | ⏳ Not wired in this pass — English only for now |
| FAISS / LangChain RAG             | ⏳ Stubbed with a clear "build_faiss_index()" extension point |

The whole thing runs on **SQLite** for zero-config demoing. Swapping to
Postgres is a one-line env var change (see below) — no model/query changes
needed since everything is SQLAlchemy ORM.

---

## Quick Start

### 1. Backend (FastAPI)

```bash
cd backend
python3 -m venv venv && source venv/bin/activate   # optional but recommended
pip install -r requirements.txt

# Seed the demo database (creates crime_platform.db)
python3 -m app.seed

# Run the API
uvicorn app.main:app --reload --port 8000
```

API docs: http://localhost:8000/docs

Demo logins (all use password `password123`):
`investigator`, `crime_analyst`, `supervisor`, `policy_maker`, `administrator`

### 2. Frontend (React + Vite)

```bash
cd frontend
npm install
npm run dev
```

App: http://localhost:5173

The frontend reads the API URL from `frontend/.env` (`VITE_API_BASE`,
defaults to `http://localhost:8000`).

---

## Switching to Postgres

```bash
export DATABASE_URL="postgresql+psycopg2://user:password@localhost:5432/crime_platform"
pip install psycopg2-binary
python3 -m app.seed        # re-seed against Postgres
uvicorn app.main:app --reload
```

## Wiring a real LLM (OpenAI GPT) into the chatbot

See the big comment block at the top of `backend/app/ai_service.py`.
The `generate_answer()` function is a drop-in swap point — replace its
body with an OpenAI chat-completions call (few lines), and optionally
build a FAISS index over FIR/case text for real RAG. No other file needs
to change; the `/api/chat/message` endpoint and the whole frontend chat
UI already work against the same response shape.

## Project layout

```
backend/
  app/
    main.py            FastAPI app + router registration
    database.py         SQLAlchemy engine/session (SQLite by default)
    models.py            All ORM tables (users, FIRs, cases, accused, etc.)
    schemas.py            Pydantic request/response models
    auth.py                JWT auth + RBAC dependency
    ai_service.py           Chat engine (OpenAI plug-in point marked)
    seed.py                  Synthetic dataset generator (Faker-based)
    routers/
      auth.py, dashboard.py, crimes.py, network.py, chat.py,
      geo.py, forecast.py, financial.py, admin.py, notifications.py
frontend/
  src/
    pages/    Login, Dashboard, Chat, Network, CrimeMap, Search,
              Offenders, Financial, Forecast, Admin
    components/ Sidebar, Layout, StatCard, ProtectedRoute
    context/AuthContext.tsx
    lib/api.ts   axios client w/ JWT interceptor
```

## Notes / known limitations of this demo pass

- The chatbot uses pattern-matching + SQL over the sample data rather than
  a real LLM, since no OpenAI key is configured in this environment — see
  the plug-in point above.
- Voice recognition (Web Speech API) works in Chrome/Edge; Safari/Firefox
  support varies.
- Gang detection and forecasting use simple, transparent heuristics
  rather than trained ML models — swap in scikit-learn/Prophet/graph
  community-detection algorithms for a production version.
- The production JS bundle is a single ~1.4MB chunk; for a real deployment,
  add route-based code-splitting (`React.lazy`) to shrink initial load.
