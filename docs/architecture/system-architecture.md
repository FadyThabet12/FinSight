# FinSight AI — System Architecture

Modular backend first; separate inference / AI services only when they earn their keep.

```mermaid
flowchart TB
    subgraph Client
        FE["React + TypeScript + Tailwind<br/>Dashboard · Reports · Copilot · Admin"]
    end
    subgraph Backend["FastAPI modular backend (/api/v1)"]
        AUTH["Auth · RBAC · Tenant isolation"]
        FIN["Financial services<br/>transactions · invoices · payments · KPIs · imports"]
        AUD["Audit logging"]
    end
    subgraph Intelligence
        ML["ML inference<br/>revenue · cash flow · payment risk · anomalies"]
        AI["AI services<br/>LLM provider · RAG · tools · safe Text-to-SQL"]
    end
    subgraph DataEng["Data engineering"]
        ETL["Ingestion · validation · cleaning · features"]
    end
    subgraph Data["Data layer"]
        PG[("PostgreSQL<br/>+ RLS")]
        RD[("Redis")]
        QD[("Qdrant<br/>vectors")]
        FS[("File storage")]
    end
    subgraph MLOps
        MLF["MLflow tracking + registry"]
        DVC["DVC datasets"]
        PRE["Prefect retraining flows"]
        MON["Evidently · Prometheus · Grafana"]
    end
    OLL["Ollama (local LLM)"]

    FE -->|REST + JWT| AUTH --> FIN
    FIN --> PG
    FIN --> AUD --> PG
    FIN --> ETL --> PG
    FE --> AI
    AI -->|"deterministic tools (tenant-checked)"| FIN
    AI -->|"tools"| ML
    AI -->|"tenant-filtered retrieval"| QD
    AI -->|"read-only role, allow-listed SQL"| PG
    AI --> OLL
    ML --> PG
    ML --> MLF
    PRE --> ML
    PRE --> MLF
    DVC --> ETL
    MON --> ML
    MON --> PG
    FIN --> FS
    AI --> RD
```

## Communication rules
- The frontend never decides authorization; the backend derives `company_id` from verified membership.
- The LLM never computes financial figures. It selects **tools**; tools query PostgreSQL / call ML models.
- Predictions are advisory and stored with the model version that produced them.
- Only models that passed registry quality gates serve production traffic.
