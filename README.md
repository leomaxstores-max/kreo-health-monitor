# Kreo Health Monitor

Independent, read-only production health monitor for the Kreo trading engine.

This public repository intentionally contains **no trading-engine source code, no database schema, no wallet data, no positions/P&L, no strategy logic, and no credentials**.

## What it does

A GitHub Actions workflow checks the secured production health boundary every five minutes and on trusted manual dispatch.

The monitor:

- sends `GET /api/health` with a dedicated `X-Health-Monitor-Key` credential;
- fails closed on network failure, HTTP errors, malformed health responses, or `HALTED` state;
- opens one generic incident issue for the first unhealthy state;
- deduplicates unchanged incidents and emits bounded reminders;
- closes the incident issue on recovery;
- performs no trading, database writes, wallet actions, order placement, continuity repair, or funds movement.

## Secret

The repository requires exactly one GitHub Actions encrypted secret:

- `HEALTH_MONITOR_KEY`

Do not place the value in source files, repository variables, issue text, logs, screenshots, workflow inputs, or artifacts.

## Security boundary

The dedicated production credential is valid only for the narrow health-observation path. This repository must never receive dashboard/admin keys, Supabase credentials, wallet/signing secrets, execution credentials, or private engine source.

Secrets are available only to trusted `schedule` and `workflow_dispatch` runs. The workflow is not triggered by `pull_request`, `pull_request_target`, `push`, or fork events.

## Cutover rule

The existing private watchdog remains authoritative fallback until this public monitor has completed:

1. at least three healthy scheduled five-minute ticks;
2. a controlled negative test that opens an incident and fails the run;
3. a recovery test that closes the incident on the next healthy check.

Only after those proofs should the private scheduled watchdog be made manual-only.
