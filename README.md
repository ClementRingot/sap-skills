# SAP Skills

A collection of AI coding assistant skills for SAP development (RAP, Fiori Elements, SAPUI5, Clean Core, ABAP Cloud). Each skill is a standalone `SKILL.md` file that can be used independently.

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
| [sapui5-review](sapui5-review/SKILL.md) | SAPUI5 & Fiori Elements UI code reviewer (XML views, controllers, manifest, OData V4 bindings, FPM) |
| [sap-rap-orchestrator](sap-rap-orchestrator/SKILL.md) | Orchestrator that ties all skills together into a recommended workflow |

## Recommended Workflow

```
1. Generate RAP / Fiori code  (rap-business-object, fiori-elements)
2. Review backend code         (sap-reviewer)
3. Review UI code              (sapui5-review)
4. Generate unit tests         (abap-unit-tests)
5. Run ATC checks              (atc-checks)
```

## Usage

Copy the skills you need into your AI assistant's skills directory (e.g. `.copilot/skills/` or `.claude/skills/`).

## License

[MIT](LICENSE)
