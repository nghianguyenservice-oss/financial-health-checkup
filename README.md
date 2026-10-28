# Financial Health Checkup

A personal, single-user application that pulls my own financial data from my banks and brokerages, keeps a historical record, and shows a clear picture of my net worth over time.

It is also a learning project: the design deliberately mirrors a production data and AI platform (medallion lakehouse, a provider adapter boundary, governed access, and an AI agent layer) so it can grow from a free local setup into Databricks and Claude.

> Informational only. This is not financial, tax, or investment advice.

## What it does

- Connects to financial institutions through an aggregator (Plaid first).
- Takes a balance snapshot every day, because the aggregator only returns current balances.
- Computes net worth (assets minus liabilities) per day, plus breakdowns by account and asset class.
- Shows it on a dashboard, and later lets Claude answer questions over the data.

## Architecture at a glance

```
Banks -> Aggregator (Plaid) -> Provider adapter -> Bronze -> Silver -> Gold -> Dashboard / Claude + MCP
```

Full diagram and the reasoning behind every choice: [docs/architecture.md](docs/architecture.md).

## Stack

Python 3.12, Plaid (sandbox first), DuckDB, plain SQL plus Python scheduled with cron or GitHub Actions, Streamlit. Phase 2 adds Databricks, phase 3 adds Claude and an MCP server.

## How we build: OpenSpec

Development is spec-driven with [OpenSpec](https://github.com/Fission-AI/OpenSpec). Every feature starts as a change proposal in `openspec/changes/`, is reviewed, then implemented, then archived into `openspec/specs/`.

```
/opsx:propose "your idea"   # write the proposal, design, tasks and spec deltas
/opsx:apply                 # implement the approved tasks
/opsx:archive               # merge the change into the specs
```

## Guardrails

The rules every contributor and AI agent must follow, including the locked stack, scope limits, and security rules, are in [AGENTS.md](AGENTS.md). Claude Code reads them through [CLAUDE.md](CLAUDE.md).

## Setup

1. Create a Plaid developer account and copy the sandbox `client_id` and `secret`.
2. Copy `.env.example` to `.env` and fill in the values. Never commit `.env`.
3. Implementation starts with the first OpenSpec change; there is no runnable code yet.
