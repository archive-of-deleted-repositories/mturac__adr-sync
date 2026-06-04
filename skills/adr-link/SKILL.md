---
name: adr-link
description: Link an ADR to the code files that implement or depend on its decision
version: 0.1.0
triggers:
  - link ADR
  - add ADR references
  - connect ADR to code
  - find orphaned ADRs
input:
  - name: adr_number
    description: The ADR number to link (e.g. 3, 0003, or the full filename)
    required: false
---

# ADR Link

Find code files that implement or depend on a given ADR's decision, then insert reference comments so the decision is traceable from the code.

## Steps

### Step 1: Resolve the target ADR

If `adr_number` is provided:
1. Locate the ADR file using the number (zero-pad to 4 digits, match `0003-*.md` or `ADR-0003-*.md`)
2. If not found, list available ADRs and ask the user to pick one

If no `adr_number` is provided:
1. Run a quick staleness scan (adr-scan Step 1–2) to list all ADRs with their status
2. Highlight any ADRs with **no existing code references** (orphaned decisions)
3. Ask: "Which ADR should I link? (Enter number)"

### Step 2: Extract the key technical decision

Parse the ADR file (same as adr-scan Step 2–3) and identify:

- The **primary technology/package** named in the Decision section
- The **primary directory or module** the decision applies to
- Any **specific patterns** the decision enforces (e.g., "all validation must go through the validation layer")

Distill this into a **search fingerprint**: a list of 3–5 terms or identifiers to grep for in the codebase.

Example — ADR "Use Zod for runtime validation":
- Search terms: `zod`, `z.object`, `z.string`, `parseAsync`, `safeParse`

Example — ADR "Use Repository Pattern for data access":
- Search terms: `Repository`, `IRepository`, `UserRepository`, `findById`, `save(`

### Step 3: Search codebase for implementing files

For each search term, run a case-sensitive grep across source files (exclude `node_modules/`, `dist/`, `.git/`, `*.test.*`, `*.spec.*`):

```bash
grep -rl "TERM" src/ lib/ app/ packages/ --include="*.ts" --include="*.tsx" --include="*.js" --include="*.py" --include="*.rs" --include="*.go" 2>/dev/null | head -30
```

Collect unique file paths. Deduplicate. Rank by hit frequency (files with the most matches first).

Also search for the ADR filename itself to find files already referencing it:
```bash
grep -rl "0003-use-zod" . --exclude-dir={node_modules,dist,.git} 2>/dev/null
```

Files already referencing the ADR are marked as `already linked`.

### Step 4: Determine comment style per file type

| Extension | Comment style |
|---|---|
| `.ts`, `.tsx`, `.js`, `.jsx` | `// See: docs/decisions/NNNN-slug.md` |
| `.py` | `# See: docs/decisions/NNNN-slug.md` |
| `.rs` | `// See: docs/decisions/NNNN-slug.md` |
| `.go` | `// See: docs/decisions/NNNN-slug.md` |
| `.css`, `.scss` | `/* See: docs/decisions/NNNN-slug.md */` |
| `.html` | `<!-- See: docs/decisions/NNNN-slug.md -->` |
| `.yaml`, `.yml` | `# See: docs/decisions/NNNN-slug.md` |

### Step 5: Propose insertions

For each candidate file (max 15 files):
- Show the filename and the first matching line
- Show where the comment will be inserted (top of file, or directly above the first match)

Present the plan:

```
ADR-0003: Use Zod for runtime validation
ADR file: docs/decisions/0003-use-zod-runtime-validation.md

Files to link (8 found, 2 already linked):

  [SKIP] src/schemas/user.schema.ts — already references ADR-0003
  [SKIP] src/schemas/product.schema.ts — already references ADR-0003
  [ADD]  src/middleware/validate.ts — line 4: import { z } from 'zod'
  [ADD]  src/api/users/users.controller.ts — line 12: UserSchema.parseAsync(body)
  [ADD]  src/api/orders/orders.controller.ts — line 8: OrderSchema.safeParse(req.body)
  [ADD]  src/lib/validators.ts — line 1: import { z } from 'zod'
  [ADD]  src/types/schemas.ts — line 3: export const BaseSchema = z.object
  [ADD]  packages/shared/src/validation.ts — line 1: import { z } from 'zod'

Write reference comments to the 6 unmarked files? [yes/no/select]
```

If user responds "select", list files numbered so they can pick specific ones.

### Step 6: Insert comments

For each approved file:
1. Read the file
2. Determine insert position (top of file, after any existing license/copyright header, before the first import OR directly above the first match line)
3. Insert the comment line
4. Write the file back

Do not insert duplicate comments. Check that the reference doesn't already exist before writing.

### Step 7: Track orphaned ADRs

After linking, check all ADRs in the project:

For each ADR file, grep for its filename (without extension) across the codebase:
```bash
grep -rl "0003-use-zod" . --exclude-dir={node_modules,dist,.git,.claude-plugin,docs} 2>/dev/null
```

Output a summary:

```
ADR Link Coverage
=================
  0001-use-postgresql.md       — 12 code references (well covered)
  0002-adopt-typescript.md     — 34 code references (well covered)
  0003-use-zod.md              — 6 code references (just linked)
  0005-event-sourcing.md       — 0 code references (ORPHANED — no implementation found)
  0006-feature-flags.md        — 1 code reference (sparse)

1 orphaned ADR detected. Run /adr-sync-scan to check if 0005 is still relevant.
```
