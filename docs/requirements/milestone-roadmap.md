# Roadmap, risks and FIN task list

## Milestones
1. Foundation & database  2. Backend & financial data  3. ML systems  4. MLOps
5. AI engineering & RAG  6. Frontend  7. Integration, testing, deployment

## Key risks
| Risk | Mitigation |
|---|---|
| Cross-tenant data leak | company_id from membership (never client), composite FKs (done), RLS (M2), tenant tests, tenant-filtered Qdrant queries |
| Target leakage in payment risk | Features only from data available at prediction time; label only matured invoices; synthetic data keeps unpaid invoices censored |
| Forecasts no better than a baseline | Seasonal-naive baseline + temporal splits + quality gate before any model is registered |
| LLM invents numbers / unsafe SQL | Tool calling for all figures; prefer predefined tools; read-only role, allow-listed schema, SQL parsing, forced tenant filter, timeouts |
| Synthetic data too clean | Seeded noise, missing values, duplicates, outliers; metrics reported as synthetic-data results only |
| Scope creep | Strict milestone gates; features outside the current milestone are deferred |
| Float rounding in money | NUMERIC(18,2) in DB, Decimal in code |
