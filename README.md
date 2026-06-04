# adr-sync

A Claude Code plugin that keeps Architecture Decision Records (ADRs) aligned with actual code changes.

ADRs rot silently — packages get swapped, directories get renamed, architectural patterns drift — while the recorded decisions remain frozen in time. `adr-sync` detects that drift and helps you stay current.

## Install

```bash
# From the adr-sync directory
claude plugin install .

# Or from a path
claude plugin install /path/to/adr-sync
```

## Commands

| Command | Description |
|---|---|
| `/adr-sync` | Quick staleness summary across all ADRs |
| `/adr-sync-scan` | Full detailed report with evidence per claim |
| `/adr-draft [topic]` | Draft a new ADR with codebase-aware context |
| `/adr-link [adr-number]` | Link an ADR to implementing code files |

## ADR formats supported

- Files named `NNNN-*.md` (e.g., `0001-use-postgresql.md`)
- Files named `ADR-NNNN-*.md` (e.g., `ADR-0001-use-postgresql.md`)
- Any `.md` file inside these directories: `docs/decisions/`, `docs/adr/`, `docs/architecture/decisions/`, `adr/`, `.adr/`

Standard sections parsed: `## Context`, `## Decision`, `## Consequences`
Status field: `**Status:** Accepted` (inline) or `## Status` (section heading)

## Example staleness report

```
ADR Staleness Report — 2026-06-05
===================================

[0001] Use PostgreSQL as primary database
  Status: Accepted | Score: 0/100 | Fresh ✓
  Claims verified: pg (package.json ✓), src/db/ (exists ✓)

[0003] Use Zod for runtime validation
  Status: Accepted | Score: 35/100 | Review suggested ⚠
  Stale claims:
    - package "zod": NOT in package.json (found "valibot" instead)
    - path "src/validation/": does not exist (renamed to src/schemas/)

[0007] Adopt microservices architecture
  Status: Accepted | Score: 55/100 | Stale ✗
  Stale claims:
    - path "services/": does not exist
    - pattern "microservices": only one package.json found (monolith detected)

Summary: 3 scanned | 1 Fresh | 1 Review | 1 Stale | 0 Critical
```

## How staleness is scored

| Condition | Points |
|---|---|
| Accepted ADR references a removed package | 30 |
| Accepted ADR references a non-existent path | 25 |
| Architectural pattern contradicted by structure | 20 |
| Version claims outdated | 15 |
| Missing required sections (Context/Decision/Consequences) | 10 |
| No code references found (orphaned decision) | 10 |
| Superseded ADR missing forward reference | 5 |

Scores: 0–15 Fresh · 16–40 Review · 41–70 Stale · 71–100 Critical drift

## Skills

- **adr-scan** — staleness detection engine
- **adr-draft** — new ADR generator with codebase-aware context
- **adr-link** — inserts `// See: docs/decisions/NNNN-slug.md` reference comments into implementing files
