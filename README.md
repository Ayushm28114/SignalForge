# SignalForge

A small, opinionated API reliability platform. Instrument a Python API and answer four questions fast: **which endpoint regressed, when it began, how severe the user impact is, and whether a deploy coincided with the change.**

> 🚧 **Status:** MVP under active development — currently milestone 1 (skeleton + local setup).

## Why this exists

Small teams can read raw logs yet still struggle to answer "what regressed and when." Datadog / New Relic are too expensive and heavy for a 1–5 service setup; self-hosted stacks (SigNoz, HyperDX) are too operationally heavy. SignalForge is one opinionated path from a Python API to actionable endpoint health — with anomaly warnings that **show their evidence** instead of a black-box score.

## How it works

```
Your Python API
      │  middleware captures: method, route template, status, latency, trace_id
      ▼
SignalForge SDK ── batches (100 events / 10s), retries, never blocks your app
      │  POST /v1/ingest  (API-key auth, deduped by event_id)
      ▼
Ingest API (Django + DRF) ── 202 Accepted ──▶ Queue (Redis + Celery)
                                                     │
                                                     ▼
                                              Rollup workers
                                              1-min / 1-hour aggregates
                                                     │
                                                     ▼
                                    Dashboard: p50/p95/p99, error rates,
                                    SLO burn-rate, deploy markers, anomalies
```

## Tech stack

- **Backend:** Django + Django REST Framework, PostgreSQL, Redis + Celery
- **SDK:** pure-Python package (Django / WSGI middleware)
- **Frontend:** React *(planned)*
- **Local infra:** Docker Compose

## Project structure

```
signalforge/
├── backend/          # Django + DRF
│   ├── config/       # settings, urls, celery
│   └── apps/
│       ├── projects/   # Project, API keys, deploy events
│       ├── ingest/     # POST /v1/ingest: auth, validation, dedupe, enqueue
│       ├── rollups/    # raw staging + 1-min / 1-hour aggregate tables & workers
│       ├── slos/       # SLO targets, error-budget & burn-rate math
│       ├── anomalies/  # baseline detector + evidence-backed flags
│       └── dashboard/  # read-only query APIs for the frontend
├── sdk/              # pip-installable signalforge_sdk package
├── frontend/         # React dashboard (planned)
├── docs/             # mvp-scope.md, architecture decisions
└── docker-compose.yml
```

## Quickstart

Prerequisites: Python 3.11+, Docker Desktop running.

```powershell
git clone https://github.com/Ayushm28114/SignalForge.git
cd SignalForge
python -m venv .venv
.venv\Scripts\Activate.ps1
pip install -r backend/requirements.txt
docker compose up -d
cd backend
python manage.py migrate
python manage.py runserver
```

Open http://localhost:8000/health/ — you should see `{"status": "ok"}`.

## Documentation

- [`docs/mvp-scope.md`](docs/mvp-scope.md) — the approved MVP scope and telemetry event contract. Every milestone builds against this.

## Roadmap

- [ ] **M1** — skeleton, Docker Compose, Django boots, `/health`
- [ ] **M2** — `projects` app: Project + API key management
- [ ] **M3** — ingest API: validation, dedupe, async enqueue
- [ ] **M4** — Python SDK: middleware, batching, failure-safe client
- [ ] **M5** — rollup workers: 1-min / 1-hour aggregates
- [ ] **M6** — dashboard query APIs + React UI
- [ ] **M7** — SLOs, error budgets, burn-rate views
- [ ] **M8** — deploy events + timeline markers
- [ ] **M9** — explainable anomaly detection
- [ ] **M10** — load tests, live demo with seeded data

## License

MIT
