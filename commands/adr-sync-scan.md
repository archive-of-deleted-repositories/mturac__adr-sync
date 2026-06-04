# /adr-sync-scan

Run a detailed staleness scan across all ADRs, showing every stale claim with specific evidence.

## Usage

```
/adr-sync-scan
/adr-sync-scan --adr 0003
```

## Options

- `--adr [number]` — Scan only the specified ADR number (e.g., `--adr 0003`)
- `--status [Accepted|Proposed|all]` — Filter by ADR status (default: `Accepted`)
- `--min-score [n]` — Only show ADRs with staleness score >= n (default: 16)

## What it does

Runs the full `adr-scan` skill with verbose output:
1. Lists all claims extracted per ADR
2. Shows verification result for each claim (pass/fail with evidence)
3. Calculates and displays the staleness score breakdown
4. Suggests a concrete remediation action per stale claim

## Skill

Uses: `skills/adr-scan`

## Example output

```
Detailed ADR Scan — docs/decisions/0003-use-zod-runtime-validation.md
======================================================================
Status:  Accepted
Score:   35/100 — Review suggested ⚠

Claims:
  [PASS] technology "Zod"        — package found: package.json (devDependencies)
  [FAIL] package "zod"           — not in dependencies or devDependencies
           Evidence: grep for "zod" in package.json returned 0 results
           Likely cause: replaced by valibot (found in devDependencies)
           Action: Update ADR decision section to reflect migration to valibot,
                   or add "Deprecated" status with pointer to new ADR
  [PASS] path "src/schemas/"     — directory exists (14 files)
  [FAIL] path "src/validation/"  — not found
           Evidence: ls src/validation/ → No such file or directory
           Likely cause: renamed to src/schemas/ in refactor-2025-11
           Action: Update ADR consequences to reference new path

Score breakdown:
  package removed: +30
  (all other claims pass)
  Total: 30/100

Remediation options:
  A) Update ADR-0003 to reflect current state (recommend)
  B) Deprecate ADR-0003 and draft a new ADR-0008 for the current validation approach
  C) Mark as Superseded and reference ADR-0008 once drafted
```
