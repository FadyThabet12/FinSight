# FinSight AI

AI-powered financial intelligence & decision-support platform for SMEs: financial analytics,
forecasting, payment-risk prediction, anomaly detection, an AI Financial Copilot (RAG + tools +
safe Text-to-SQL) and a full MLOps lifecycle.

> Decision support only. Predictions are advisory, never automatic financial decisions, and
> FinSight AI is not a substitute for an accountant or financial advisor.

## Status

| Milestone | Scope | Status |
|---|---|---|
| **1** | Foundation & database | **Implemented — awaiting review** |
| 2 | Backend & financial data (auth, RBAC, imports, KPIs) | Not started |
| 3 | ML systems | Not started |
| 4 | MLOps | Not started |
| 5 | AI engineering & RAG | Not started |
| 6 | Frontend | Not started |
| 7 | Integration, testing, deployment | Not started |

## What exists in Milestone 1
- FastAPI app with `/health`, `/health/db`, OpenAPI at `/api/v1/openapi.json`
- SQLAlchemy 2.0 models for 19 tables + Alembic migration `0001`
- Tenant isolation inside the DB (composite foreign keys), CHECK constraints, soft delete, `NUMERIC(18,2)` money
- Deterministic synthetic data generator + DB loader (seeded anomalies, late payers, unpaid/censored invoices)
- Docker Compose (PostgreSQL + backend), minimal GitHub Actions CI
- Docs: `docs/architecture/system-architecture.md`, `docs/database/erd.md`

## Quick start (local, without Docker)

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env            # edit POSTGRES_PASSWORD

# PostgreSQL 16 must be running and the database/user from .env must exist
alembic upgrade head
PYTHONPATH=backend:. python -m database.seeds.load --seed 42 --months 24 --customers 40
PYTHONPATH=backend uvicorn app.main:app --reload
# http://localhost:8000/docs   |   http://localhost:8000/health/db
```

## Quick start (Docker)

```bash
cp .env.example .env            # edit POSTGRES_PASSWORD
docker compose up --build       # applies migrations, then serves on :8000
docker compose exec backend python -m database.seeds.load --seed 42
```

## Tests

```bash
pytest                                   # fast suite (in-memory SQLite) — Postgres tests are skipped
export TEST_DATABASE_URL=postgresql+psycopg://USER:PASS@localhost:5432/finsight_test
pytest                                   # full suite incl. real-PostgreSQL migration tests
ruff check .
```
`TEST_DATABASE_URL` must point at a **throwaway** database: the Postgres tests run
`alembic downgrade base` followed by `upgrade head`.

## Migrations

```bash
alembic upgrade head
alembic downgrade base
alembic revision --autogenerate -m "describe change"
alembic check                    # fails if models and migrations drift
```

## Common problems
| Symptom | Fix |
|---|---|
| `ModuleNotFoundError: app` | Run from repo root; use `PYTHONPATH=backend:.` for scripts |
| `connection refused` | PostgreSQL not running / wrong `POSTGRES_HOST` in `.env` |
| `password authentication failed` | `.env` password differs from the DB role's password |
| Postgres tests skipped | `TEST_DATABASE_URL` not exported |

## Security notes
Secrets live in `.env` (git-ignored); `.env.example` has placeholders only. Only synthetic data is used.
