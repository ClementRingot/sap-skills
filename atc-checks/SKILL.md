---
name: atc-checks
description: This skill provides guidelines for running ABAP Test Cockpit (ATC) checks on RAP objects, interpreting results, and applying corrections. It covers common ATC errors with their fixes and defines a structured correction loop (max 3 iterations).
---

## Workflow

```
1. Collect the list of objects to check
2. Run ATC checks (via ADT or programmatically)
3. Analyze results
4. If errors found:
   a. Apply correction
   b. Activate the object
   c. Re-run ATC checks
   d. Repeat (max 3 iterations)
5. Produce final report
```

---

## ATC Error Categories

| Priority | Type | Action |
|----------|------|--------|
| 1 (Error) | Syntax errors, type mismatches | Fix immediately |
| 2 (Warning) | Unused variables, missing checks | Fix recommended |
| 3 (Info) | Style, naming suggestions | Ignore unless Clean Core related |

---

## Common ATC Errors and Fixes

| ATC Error | Fix |
|-----------|-----|
| Unused variable | Remove declaration or add `##NEEDED` pragma |
| Unmapped field in BDEF | Add to the mapping section |
| Missing authorization check | Add `authorization master ( global )` |
| Implicit data declaration | Use `DATA(...)` or `FINAL(...)` |
| Missing exception handling | Add `TRY...CATCH` or `RAISING` |
| Method too long | Extract into private helper methods |
| SELECT without ORDER BY on non-unique results | Add `ORDER BY` or use `UP TO 1 ROWS` |
| Obsolete statement | Replace with ABAP Cloud equivalent |
| Non-released object used | Replace with released API or create wrapper |
| Missing WHERE clause on DELETE/UPDATE | Add WHERE condition |

---

## Correction Rules

- **Never suppress a Priority 1 error** with a pragma — fix it properly.
- **Priority 2 warnings** can be suppressed with `##NEEDED` only for genuinely unused variables that are structurally required (e.g., `FAILED` parameter that must be declared but is not read).
- **Always re-activate the object** after correction before re-running ATC.
- **Stop after 3 iterations** — if errors persist, escalate for manual review.

---

## Report Format

```
## ATC REPORT — {date}

### Objects Checked: {n}
### Iterations: {n}/3

| Object | Type | Errors | Warnings | Status |
|--------|------|--------|----------|--------|
| ZBP_R_MYTP | CLAS | 0 | 1 | ✅ |
| ZR_MYTP | DDLS | 0 | 0 | ✅ |

### Corrections Applied
1. {object}: {correction description}

### VERDICT: ✅ ALL PASSED | ⚠️ WARNINGS REMAINING | ❌ ERRORS REMAINING
```
