# Integration inventory

Ranked by access risk (political x2 + technical).

| System | Data | Protocol | Owner | Tech | Political | Identity / access note |
|---|---|---|---|---|---|---|
| Document store | photos, estimates, police/medical reports, statements | object store list/fetch + text extraction | Records / InfoSec | 4/5 | 4/5 | highest PII sensitivity; synthetic only; injection vector (M5) |
| Claims management | claim, narrative, status, assigned adjuster | REST/JSON (read + idempotent draft write) | Claims IT / Head of Claims | 3/5 | 4/5 | only WRITE path; entitlement = assigned adjuster or team (M4) |
| Policy administration | in-force dates, cover, limits, excess, clauses | REST/JSON + clause export to corpus | Underwriting / Policy IT | 3/5 | 3/5 | read-only; dates and money computed in code, never by model |
