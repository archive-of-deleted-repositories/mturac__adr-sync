# /adr-sync

Scan all Architecture Decision Records in this project for staleness and summarize the results.

## Usage

```
/adr-sync
```

## What it does

1. Finds all ADR files in `docs/decisions/`, `docs/adr/`, `adr/`, or similar directories
2. Parses each ADR for status, technical claims, and referenced paths/packages
3. Verifies each claim against the current codebase (package.json, file existence, architecture patterns)
4. Outputs a ranked staleness report — Fresh / Review suggested / Stale / Critical drift
5. Highlights the most urgent ADRs to update

## Skill

Uses: `skills/adr-scan`

## Example output

```
ADR Staleness Report — 2026-06-05
===================================

[0001] Use PostgreSQL as primary database
  Status: Accepted | Score: 0 | Fresh ✓

[0003] Use Zod for runtime validation
  Status: Accepted | Score: 35 | Review suggested ⚠
  → package "zod" not found in package.json

[0007] Adopt microservices architecture
  Status: Accepted | Score: 55 | Stale ✗
  → path "services/" does not exist
  → monolith structure detected (single package.json)

Summary: 3 scanned | 1 Fresh | 1 Review | 1 Stale | 0 Critical
Run /adr-sync-scan for full detail on stale ADRs.
```
