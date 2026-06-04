# /adr-link

Link an Architecture Decision Record to the code files that implement or depend on its decision.

## Usage

```
/adr-link [adr-number]
```

## Examples

```
/adr-link 3
/adr-link 0003
/adr-link 0003-use-zod-runtime-validation
/adr-link          ← lists all ADRs and their link coverage, lets you pick
```

## What it does

1. Resolves the target ADR file
2. Extracts key technical identifiers from the Decision section (package names, import patterns, class names)
3. Greps the codebase for files that use those identifiers
4. Shows a proposed insertion plan — which files will receive a reference comment
5. Inserts `// See: docs/decisions/NNNN-slug.md` above the first match in each file (after confirmation)
6. Reports orphaned ADRs — decisions with zero code references across the project

## Skill

Uses: `skills/adr-link`

## Comment styles by file type

| File type | Comment format |
|---|---|
| TypeScript / JavaScript | `// See: docs/decisions/0003-use-zod.md` |
| Python | `# See: docs/decisions/0003-use-zod.md` |
| Rust / Go | `// See: docs/decisions/0003-use-zod.md` |
| CSS / SCSS | `/* See: docs/decisions/0003-use-zod.md */` |
| YAML | `# See: docs/decisions/0003-use-zod.md` |

## Example output

```
ADR-0003: Use Zod for runtime validation

Files to link (8 found):
  [SKIP] src/schemas/user.schema.ts    — already linked
  [ADD]  src/middleware/validate.ts    — import { z } from 'zod' at line 4
  [ADD]  src/api/users.controller.ts   — UserSchema.parseAsync at line 12
  [ADD]  src/lib/validators.ts         — z.object at line 3

Write reference comments to 3 files? [yes/no]
```
