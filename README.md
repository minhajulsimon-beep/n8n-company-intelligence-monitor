# Company News Monitoring & Business Event Classification

An **n8n + PostgreSQL/Supabase** workflow that retrieves companies from a database, searches Google News RSS, stores newly observed articles without duplicates, and flags potential business events using a JavaScript keyword classifier.

## What it does

1. **Load active companies** from the `companies` table.
2. **Search Google News RSS** for each company and limit to 10 feed items per company in each run.
3. **Normalize and store observations** in PostgreSQL. An `ON CONFLICT (company_id, source_url) DO NOTHING` insert avoids duplicate observations.
4. **Process only new observations** based on the insert result.
5. **Classify news** into acquisition, funding, partnership, leadership, expansion, layoffs, or other using JavaScript keyword matching.
6. **Save flagged events** to `events` using the True branch of an IF node. Unflagged items end on the False branch.

## Workflow

```text
Manual Trigger
    ↓
Load Active Companies (PostgreSQL)
    ↓
Loop Companies → Google News RSS → Limit 10 → Normalize Article ↩
    ↓ (after loop)
Save New Observations (PostgreSQL)
    ↓
Is New Article? ── False → End
    ↓ True
JavaScript Rule-Based Classifier
    ↓
Is Significant? ── False → End
    ↓ True
Save Classified Events (PostgreSQL)
```

## Tech stack

- **n8n** — workflow orchestration
- **PostgreSQL / Supabase** — company records, observations, and events
- **Google News RSS** — public news feed ingestion
- **JavaScript** — keyword-based event classification
- **SQL** — insertion and duplicate checks

## Setup

1. Import [`workflows/company-news-monitoring.json`](workflows/company-news-monitoring.json) into n8n.
2. Configure your **own** Postgres credentials for each Postgres node. Credentials are intentionally omitted from the shared JSON.
3. Create or adapt the tables described in [`docs/database-schema.md`](docs/database-schema.md).
4. Ensure the database has active companies and run using the **manual trigger**.
5. Update field mappings and SQL if your schema differs.

This workflow **does not require an OpenAI API key**. The classification stage is rule-based, not LLM-driven.

## Current scope and limitations

- This export uses a **manual trigger**. It does not run automatically on a schedule.
- Keyword matches are **candidates**, not independently verified events. Ambiguous headlines, negation, and unrelated uses of terms can cause false positives.
- The `events` query checks for existing `observation_id` before inserting. This is not atomic conflict prevention under concurrent executions unless an appropriate unique constraint is also present.
- The event-saving portion has been **configured**, but its successful end-to-end execution has not been independently verified for this public export.
- Google News RSS availability and content may change.

## Security

No PostgreSQL credential references, API secrets, or n8n instance identifiers are included in the published workflow JSON. Configure credentials inside n8n after import. Do not commit `.env` files or exported credentials.

## Possible improvements

- Add a schedule trigger and alerts.
- Improve event detection beyond basic keyword matching.
- Add stronger database constraints and monitoring.
- Optionally integrate an LLM for more precise classification when API access is available.
