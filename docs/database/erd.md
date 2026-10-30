# FinSight AI — Database ERD (Milestone 1)

PostgreSQL, UUID primary keys, `NUMERIC(18,2)` money, `company_id` on every business table.
Tenant integrity is enforced **inside the database** with composite foreign keys, e.g.
`invoices(company_id, customer_id) → customers(company_id, id)`: an invoice cannot reference
another company's customer even if application code has a bug.

```mermaid
erDiagram
    companies ||--o{ company_members : has
    users ||--o{ company_members : joins
    companies ||--o{ customers : owns
    companies ||--o{ products : owns
    companies ||--o{ invoices : owns
    customers ||--o{ invoices : "billed (same company)"
    invoices ||--o{ payments : "paid by (same company)"
    companies ||--o{ transactions : owns
    invoices ||--o{ transactions : "source"
    payments ||--o{ transactions : "source"
    transactions ||--o| transactions : "reversal_of"
    companies ||--o{ expenses : owns
    expenses }o--o| transactions : "ledger entry"
    companies ||--o{ imports : tracks
    users ||--o{ imports : uploads
    companies ||--o{ audit_logs : logs
    model_versions ||--o{ predictions : produced
    model_versions ||--o{ forecast_results : produced
    model_versions ||--o{ anomalies : produced
    model_versions ||--o{ risk_assessments : produced
    transactions ||--o{ anomalies : flagged
    customers ||--o{ risk_assessments : scored
    invoices ||--o{ risk_assessments : scored
    companies ||--o{ documents : owns
    documents ||--o{ document_chunks : "split into"

    companies { uuid id PK
        string name
        string industry
        string currency }
    users { uuid id PK
        string email UK
        string password_hash
        bool is_active
        bool is_superuser }
    company_members { uuid company_id FK
        uuid user_id FK
        string role }
    customers { uuid id PK
        uuid company_id FK
        string name }
    products { uuid id PK
        uuid company_id FK
        string category
        numeric unit_price
        numeric unit_cost }
    invoices { uuid id PK
        uuid company_id FK
        uuid customer_id FK
        string invoice_number
        date issue_date
        date due_date
        numeric amount
        string status }
    payments { uuid id PK
        uuid company_id FK
        uuid invoice_id FK
        numeric amount
        date payment_date }
    transactions { uuid id PK
        uuid company_id FK
        string transaction_type
        numeric amount
        string category
        timestamptz transaction_date
        uuid reversal_of_id FK }
    expenses { uuid id PK
        uuid company_id FK
        string category
        numeric amount
        date expense_date }
    imports { uuid id PK
        uuid company_id FK
        string file_name
        string status
        int invalid_rows }
    audit_logs { uuid id PK
        uuid company_id FK
        uuid user_id FK
        string action }
    model_versions { uuid id PK
        string model_name
        string version
        string stage
        json metrics }
    experiments { uuid id PK
        string model_name
        json params
        json metrics }
    predictions { uuid id PK
        uuid company_id FK
        uuid model_version_id FK
        string prediction_type
        json result }
    forecast_results { uuid id PK
        uuid company_id FK
        uuid model_version_id FK
        date forecast_date
        numeric predicted_value }
    anomalies { uuid id PK
        uuid company_id FK
        uuid transaction_id FK
        float anomaly_score
        string review_status }
    risk_assessments { uuid id PK
        uuid company_id FK
        uuid customer_id FK
        float risk_score
        string risk_category }
    documents { uuid id PK
        uuid company_id FK
        string filename
        string status }
    document_chunks { uuid id PK
        uuid company_id FK
        uuid document_id FK
        text text
        string vector_id }
```

## Design decisions & known limits
- **Roles**: `super_admin` is platform-level (`users.is_superuser`); per-company roles live in `company_members.role` (`business_owner`, `finance_manager`, `financial_analyst`, `viewer`).
- **Transactions are a cash ledger, not double-entry accounting.** KPIs such as net profit must be labelled accordingly (Milestone 2). If full accounting is needed later: `accounts`, `journal_entries`, `journal_entry_lines`.
- **Soft delete / reversal**: `deleted_at` on financial records; `transactions.reversal_of_id` for reversals. No hard deletes of financial data.
- **Row-Level Security**: planned for Milestone 2 together with auth, because policies need the authenticated tenant (`SET app.current_company`). Composite FKs already protect cross-tenant references today.
- `model_versions` / `experiments` are platform-level; they hold metrics and metadata, never company rows.
- `anomalies.review_status` exists because an anomaly is a review candidate, not proof of fraud.
