---
name: adr-draft
description: Draft a new Architecture Decision Record for a given decision topic
version: 0.1.0
triggers:
  - draft ADR
  - create ADR
  - new architecture decision
  - document decision
input:
  - name: topic
    description: The decision topic (e.g. "why we use Zod for validation")
    required: true
---

# ADR Draft

Generate a well-structured ADR for a given architectural decision, pre-filled with codebase context.

## Steps

### Step 1: Accept input

The user provides a decision topic. Examples:
- "why we use Zod for runtime validation"
- "adopt PostgreSQL over SQLite"
- "use Turborepo for monorepo tooling"
- "switch from REST to tRPC"

If no topic is provided, ask: "What decision should I document?"

### Step 2: Detect next ADR number

1. Find the ADR directory (same lookup as `adr-scan` Step 1).
2. If the directory does not exist yet, create `docs/decisions/` and start at `0001`.
3. List all existing ADR files, extract their leading 4-digit numbers, take the max, add 1.
4. Format as zero-padded 4 digits: `0004`, `0012`, etc.

### Step 3: Generate slug

Convert the topic to a URL-safe slug:
- Lowercase
- Replace spaces and special characters with hyphens
- Remove stop words (a, an, the, for, to, of, in, on, at, by, with) from the slug if the result is still clear
- Max 50 characters

Examples:
- "why we use Zod for runtime validation" → `use-zod-runtime-validation`
- "adopt PostgreSQL over SQLite" → `adopt-postgresql-over-sqlite`

### Step 4: Pre-fill Context from codebase

Inspect the current project to provide real context in the ADR:

1. **Detect tech stack:**
   - Read `package.json` for framework clues (next, remix, express, fastify, hono, react, vue, etc.)
   - Read `tsconfig.json` if present (indicates TypeScript project)
   - Check for `Cargo.toml`, `go.mod`, `pyproject.toml` (Rust/Go/Python)
   - Detect testing framework (vitest, jest, pytest, etc.)

2. **Detect existing decisions:**
   - Scan existing ADRs to reference related ones (e.g., "See ADR-0002 for the validation layer design")

3. **Detect problem context from topic:**
   - If topic mentions a library/tool, check if an alternative is already in package.json
   - Note the current state that motivates the decision

### Step 5: Generate ADR markdown

Produce the full ADR using this template, with real content (not placeholder lorem ipsum):

```markdown
# NNNN. [Title derived from topic]

**Date:** YYYY-MM-DD
**Status:** Proposed
**Deciders:** [leave blank for user to fill]

## Context

[2-4 sentences describing the current situation and why a decision is needed.
Reference the detected tech stack. Reference related ADRs if found.]

## Decision

[1-3 sentences stating the decision clearly.
Name the specific technology, pattern, or approach chosen.]

## Consequences

**Positive:**
- [Key benefit 1]
- [Key benefit 2]

**Negative / Trade-offs:**
- [Key trade-off 1]
- [Key trade-off 2]

**Neutral:**
- This decision will be reviewed if [relevant condition].
```

Use today's date for the Date field. Generate real sentences based on the topic and detected codebase — do not leave generic brackets unfilled.

### Step 6: Confirm before writing

Show the full draft to the user with the target path:

```
Draft ADR: docs/decisions/0004-use-zod-runtime-validation.md

---
[full markdown content]
---

Write this file? [yes/no]
```

If confirmed (user says "yes", "write it", "looks good", "go ahead"), write the file to the detected ADR directory. If declined, offer to revise specific sections.

### Step 7: Post-write suggestions

After writing, output:

```
Written: docs/decisions/0004-use-zod-runtime-validation.md

Next steps:
  /adr-link 4   — link this ADR to implementing code files
  /adr-sync     — re-scan all ADRs to confirm no drift
```
