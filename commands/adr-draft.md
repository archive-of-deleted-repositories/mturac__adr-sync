# /adr-draft

Draft a new Architecture Decision Record for a given decision topic.

## Usage

```
/adr-draft [topic]
```

## Examples

```
/adr-draft why we use Zod for runtime validation
/adr-draft adopt PostgreSQL over SQLite
/adr-draft switch from REST to tRPC
/adr-draft use Turborepo for monorepo tooling
/adr-draft adopt feature flags with Unleash
```

## What it does

1. Detects the next available ADR number from existing files
2. Generates a URL-safe filename slug from the topic
3. Inspects `package.json` and project structure to pre-fill the Context section with real codebase state
4. Generates a complete ADR using the standard template (Title, Status, Context, Decision, Consequences)
5. Shows the draft and asks for confirmation before writing
6. Writes to `docs/decisions/NNNN-<slug>.md` (creates the directory if needed)

## Skill

Uses: `skills/adr-draft`

## ADR format produced

```markdown
# 0004. Use Zod for Runtime Validation

**Date:** 2026-06-05
**Status:** Proposed
**Deciders:** [fill in]

## Context

The project uses TypeScript throughout (tsconfig.json present). External API inputs
and form payloads currently lack runtime validation — type assertions provide
compile-time safety only. The team evaluated joi, yup, and Zod.

## Decision

Adopt Zod as the standard runtime validation library. All external input boundaries
(API request bodies, env vars, config files) must validate through Zod schemas
before use.

## Consequences

**Positive:**
- TypeScript-first API with inferred types from schemas
- Single source of truth for both runtime validation and TypeScript types
- Tree-shakeable, zero dependencies

**Negative / Trade-offs:**
- Adds a production dependency (~14 kB gzipped)
- Learning curve for team members unfamiliar with Zod's fluent API

**Neutral:**
- This decision will be reviewed if Zod drops TypeScript-first support or a
  significantly lighter alternative emerges.
```
