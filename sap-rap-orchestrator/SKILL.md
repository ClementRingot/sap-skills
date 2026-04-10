---
name: sap-rap-orchestrator
description: A collection of independent skills for SAP RAP, Fiori Elements, Clean Core, code review, ABAP Unit testing, and ATC checks. Each skill is self-contained and can be used on its own.
---

## Architecture

```
sap-skills-package/
├── SKILL.md                               ← This file
└── skills/
    ├── rap-business-object/SKILL.md        ← RAP Business Object creation
    ├── fiori-elements/SKILL.md             ← Fiori Elements UI annotations & templates
    ├── clean-core/SKILL.md                 ← Clean Core / ABAP Cloud guardrails
    ├── sap-reviewer/SKILL.md               ← Adversarial code review checklist
    ├── abap-unit-tests/SKILL.md            ← ABAP Unit test generation templates
    └── atc-checks/SKILL.md                 ← ATC check execution & correction loop
```

> Each skill is **fully independent** and can be used on its own without the others.

---

## Which Skill to Read

| Task | Skill |
|------|-------|
| Create a complete RAP BO | `rap-business-object` |
| Create a Fiori Elements app | `fiori-elements` |
| Verify Clean Core compliance | `clean-core` |
| Interact with a standard BO (cross-BO) | `rap-bo` (cross-BO section) |
| Review existing code | `sap-reviewer` |
| Generate ABAP Unit tests | `abap-unit-tests` |
| Run and fix ATC checks | `atc-checks` |

---

## Recommended Workflow (when all skills are available)

```
1. Read the skill matching the task
       │
2. Gather requirements / ask scoping questions
       │
3. Generate RAP / Fiori code
       │
4. Review with sap-reviewer checklist
       │  ├── APPROVED → step 5
       │  └── CHANGES REQUESTED → back to step 3
       │
5. Generate ABAP Unit tests with abap-unit-tests
       │
6. Run ATC checks with atc-checks
       │  ├── PASSED → done
       │  └── FAILED → back to step 3 (max 3 iterations)
```
