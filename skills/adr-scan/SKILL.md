---
name: adr-scan
description: Scan all Architecture Decision Records for staleness against the current codebase
version: 0.1.0
triggers:
  - scan ADRs
  - check ADR staleness
  - validate architecture decisions
  - ADR drift detection
---

# ADR Scan

Scan the codebase for Architecture Decision Records and check whether documented decisions still match reality.

## Steps

### Step 1: Find ADR files

Search for ADR files in these locations (priority order):

- `docs/decisions/`
- `docs/adr/`
- `docs/architecture/decisions/`
- `adr/`
- `.adr/`

File patterns to match:
- `NNNN-*.md` (e.g., `0001-use-postgresql.md`)
- `ADR-NNNN-*.md`
- `ADR-*.md`
- Any `.md` inside an `adr/` or `decisions/` directory

If no ADRs are found, report it and suggest running `/adr-draft` to create the first one.

### Step 2: Parse each ADR

Extract structured fields from each file:

- **Title:** First `# ` heading
- **Status:** Look for `**Status:**`, `## Status`, or `- Status:` followed by: `Proposed`, `Accepted`, `Deprecated`, `Superseded`, `Rejected`, or `Draft`
- **Context:** Text between `## Context` and next `##`
- **Decision:** Text between `## Decision` and next `##`
- **Consequences:** Text between `## Consequences` and next `##`
- **Date / Deciders:** Look for `**Date:**` and `**Deciders:**` fields

Mark missing sections as `[MISSING]` — they are themselves a staleness signal.

### Step 3: Extract technical claims

From Decision and Context sections, extract:

- **Technology names:** Libraries, frameworks, databases, services (e.g., "PostgreSQL", "Zod", "Redis")
- **File paths / directories:** Any paths mentioned (e.g., `src/api/`, `config/database.ts`)
- **Architectural patterns:** "monolith", "microservices", "CQRS", "event-driven", "hexagonal"
- **Package names:** npm/pip/cargo packages (e.g., `zod`, `express`, `typeorm`)
- **Version constraints:** Node 18+, TypeScript 5.x, etc.

Store each as: `{ type: "tech"|"path"|"pattern"|"package"|"version", value, context }`.

### Step 4: Check claims against the codebase

For each claim:

**Package claims → check manifests:**
- `package.json` dependencies/devDependencies
- `requirements.txt`, `pyproject.toml`, `Cargo.toml`, `go.mod`
- Missing package → stale claim: `package removed`
- Version mismatch → stale claim: `version mismatch`

**Path claims → check existence:**
- Run `ls` or glob for each mentioned path
- Missing path → stale claim: `path no longer exists`

**Pattern claims → heuristic:**
- "monorepo" → look for `workspaces` in package.json, `turbo.json`, `nx.json`
- "TypeScript" → look for `tsconfig.json`
- "GraphQL" → look for `.graphql` files or graphql dependency
- Contradicted patterns → stale claim: `pattern contradicted`

### Step 5: Score and report

**Staleness weights:**

| Condition | Points |
|---|---|
| Accepted ADR references removed packages | 30 |
| Accepted ADR references non-existent paths | 25 |
| Pattern claims contradicted by structure | 20 |
| Version claims outdated | 15 |
| Missing required sections | 10 |
| No code references (grep for ADR filename) | 10 |
| Superseded ADR missing forward reference | 5 |

Score = sum of applicable weights, capped at 100. Classify:
- 0–15 → **Fresh** (green)
- 16–40 → **Review suggested** (yellow)
- 41–70 → **Stale** (orange)
- 71–100 → **Critical drift** (red)

**Output format:**

```
ADR Staleness Report
====================

[0001] Use PostgreSQL as primary database
  Status: Accepted | Score: 0/100 | Fresh
  Claims verified: postgresql (in package.json ✓), src/db/ (exists ✓)

[0003] Use Zod for runtime validation
  Status: Accepted | Score: 35/100 | Review suggested
  Stale claims:
    - package zod: NOT FOUND in package.json (was removed in favour of valibot)
    - path src/validation/: does not exist (renamed to src/schemas/)

[0007] Adopt microservices architecture
  Status: Accepted | Score: 55/100 | Stale
  Stale claims:
    - pattern "microservices": only one package.json found, monolith structure detected
    - path services/: does not exist

Summary: 3 ADRs scanned | 1 Fresh | 1 Review | 1 Stale | 0 Critical
```
