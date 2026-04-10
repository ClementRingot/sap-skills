# SAP RAP Skills

A collection of AI coding assistant skills for SAP RAP (RESTful ABAP Programming Model) development. Each skill is a standalone `SKILL.md` file that can be used independently.

## Skills

| Skill | Description |
|-------|-------------|
| [rap-business-object](rap-business-object/SKILL.md) | Complete RAP Business Object creation (CDS, BDEF, behavior implementation, draft, numbering, actions, side effects, cross-BO) |
| [fiori-elements](fiori-elements/SKILL.md) | Fiori Elements app templates, UI annotations, Metadata Extensions, Value Helps, and OData V4 Service Binding |
| [clean-core](clean-core/SKILL.md) | Clean Core / ABAP Cloud guardrails: release status verification, wrapper patterns, environment constraints |
| [abap-unit-tests](abap-unit-tests/SKILL.md) | ABAP Unit test generation for RAP BOs (CRUD, validations, determinations, actions, test doubles) |
| [atc-checks](atc-checks/SKILL.md) | ATC check execution, result interpretation, and structured correction loop |
| [sap-reviewer](sap-reviewer/SKILL.md) | Adversarial code review checklist for RAP / ABAP Cloud projects |
| [sap-released-objects](sap-released-objects/SKILL.md) | Query the SAP Cloudification Repository for object release status and successors |
| [sap-rap-orchestrator](sap-rap-orchestrator/SKILL.md) | Orchestrator that ties all skills together into a recommended workflow |

## Recommended Workflow

```
1. Generate RAP / Fiori code  (rap-business-object, fiori-elements)
2. Review with checklist       (sap-reviewer)
3. Generate unit tests         (abap-unit-tests)
4. Run ATC checks              (atc-checks)
```

## Usage

Copy the skills you need into your AI assistant's skills directory (e.g. `.copilot/skills/` or `.claude/skills/`).

## License

[MIT](LICENSE)
