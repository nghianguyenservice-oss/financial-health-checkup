# AGENTS.md: guardrails for this repository

These rules apply to every AI agent and every human contributor. If a request conflicts with them, stop and ask the owner instead of working around them. Rationale for each design choice lives in `docs/architecture.md`.

## 1. What this project is, and is not

A single-user, read-only application that pulls the owner's own financial data and shows a clear picture of net worth and its history.

Out of scope. Do not build, propose, or "just add" any of these without an approved OpenSpec change that amends this section:

- Moving money, payments, transfers, or trading. No Plaid Transfer, Payment Initiation, or similar products.
- Multi-user support, sign-up, authentication for other people, or public hosting.
- Storing or sending data for anyone other than the owner.
- Investment, tax, or legal recommendations. Output is informational and must not read as advice.

## 2. Spec-driven workflow (OpenSpec)

1. No feature code without an approved change in `openspec/changes/`. Start with `/opsx:propose "<idea>"` (Claude Code) or `/openspec-propose` (other agents).
2. A change must contain `proposal.md`, `design.md`, `tasks.md` and spec deltas under `specs/`, and must state what is out of scope and the security and privacy impact.
3. The owner reviews and approves the proposal before `/opsx:apply`. Implement only the tasks listed, then `/opsx:archive`.
4. Allowed without a change: typo or docs fixes, dependency patch bumps, and fixing a bug so code matches an existing spec.
5. Specs in `openspec/specs/` are the source of truth. If code and spec disagree, the spec wins until a change says otherwise.

## 3. Locked tech stack (confirmed by the owner on 2026-10-28)

| Concern | Choice |
| --- | --- |
| Language | Python 3.12 |
| Aggregator | Plaid, sandbox first. Behind the provider adapter so it can be swapped |
| Local store | DuckDB (single file, git-ignored) |
| Transforms | Plain SQL files plus Python. No dbt, Dagster, Airflow, or Spark in phase 1 |
| Scheduling | cron or GitHub Actions |
| Dashboard | Streamlit (phase 1) |
| Tests and lint | pytest, ruff |
| Later phases | Databricks (Delta, Unity Catalog, Lakeflow) in phase 2. Claude API plus an MCP server in phase 3 |

Do not add a framework, database, cloud service, or SDK that is not in this table. Propose it through OpenSpec with a design note justifying the deviation.

## 4. Phases: stay inside the current one

- Phase 1: local, free. Plaid sandbox, DuckDB, SQL transforms, Streamlit.
- Phase 2: move storage and pipelines to Databricks.
- Phase 3: Claude agent and MCP server over the gold tables.

Do not implement later-phase work early. Current phase: 1.

## 5. Security and privacy: hard rules

- Never commit secrets, tokens, `.env`, `.secrets/`, database files, or real financial data. Check `git diff --staged` before every commit.
- Development and tests use the Plaid sandbox and synthetic fixtures only. No real account names, numbers, balances, or transactions in the repo, logs, issues, or PR text.
- Access tokens live only in the secrets store (`.secrets/`, mode 600, git-ignored) and are never printed or logged.
- Mask account identifiers in logs and UI.
- The application is read-only against financial institutions.
- The only outbound network destinations are the configured aggregator and, in phase 3, the Anthropic API. No analytics or telemetry.
- Data sent to any LLM is limited to transaction date, amount, description, and category. Never send account numbers, institution login data, tokens, or account names.
- Pin dependencies. Do not add a dependency for something the standard library or an existing dependency already does.

## 5a. Correctness rules

- Money is `decimal.Decimal` in Python and `DECIMAL(18,2)` in DuckDB. Never `float`.
- Sign convention: assets positive, liabilities negative, transaction inflows positive and outflows negative. Convert provider sign conventions inside the adapter only.
- Only code in `src/**/providers/` may import a provider SDK or read provider-specific JSON. Everything else consumes the standard dataclasses.
- Layers are one-way: bronze (raw) to silver (typed) to gold (metrics). Gold is computed in SQL and never edited by hand.
- Every sync is idempotent: running it twice on the same day must not duplicate rows.
- Net worth history begins at the first sync, because the aggregator provides current balances only. Any backfill is a separate, approved change.

## 6. Definition of done

- Tests pass (`pytest`) and lint is clean (`ruff check .`).
- New behavior has a spec scenario and a test that cites it.
- `openspec validate` passes for the active change.
- No secrets or real data in the diff.
- Docs updated if architecture or stack changed.

## 7. When unsure

Ask. Prefer a short question to a large guess, and never widen scope, stack, or data access on your own.
