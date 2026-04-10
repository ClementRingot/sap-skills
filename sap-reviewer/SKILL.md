---
name: sap-reviewer
description: This skill provides a structured adversarial code review checklist for RAP and ABAP Cloud projects. It covers Clean Core compliance, naming conventions, transactional model, BDEF structure, CDS architecture, UI annotations, numbering, feature control, side effects, cross-BO patterns, error handling, and performance.
---

## Review Approach

Review the generated code **adversarially**: actively look for problems, not confirmations.
Go through each checkpoint below. Report findings in the output format at the end.

---

## Checklist (10 Points)

### 1. Clean Core Compliance
- [ ] All SAP standard objects used are Released (C1)?
- [ ] `strict ( 2 )` in all BDEFs?
- [ ] No direct SELECT on non-released standard tables?
- [ ] ABAP Cloud types used (no obsolete types)?

### 2. Naming Conventions
- [ ] CDS: ZI_*/ZR_*TP/ZC_*TP/ZI_*TP?
- [ ] Tables: Z{NAME}/Z{NAME}_D?
- [ ] Behavior Pool: ZBP_R_{NAME}TP?
- [ ] Service: ZUI_*_O4 / ZAPI_*?
- [ ] Value Helps: ZI_*_VH?
- [ ] Abstract Entities: ZA_*?

### 3. Transactional Model
- [ ] No COMMIT ENTITIES in handlers?
- [ ] No COMMIT WORK in handlers?
- [ ] Cross-BO with FK: `with cross associations` declared?
- [ ] Cross-BO without FK: `foreign entity` declared?
- [ ] No raw EML cross-BO calls without at least `foreign entity`?
- [ ] Cross-BO EML calls in interaction phase (not in save phase)?

### 4. BDEF Structure
- [ ] `managed` or `managed with unmanaged save`?
- [ ] `with draft` enabled (if UI)?
- [ ] Admin fields set to readonly?
- [ ] Complete field mapping?
- [ ] ETag configured (etag master + lock master total etag)?
- [ ] Numbering strategy correct (managed/early/late)?

### 5. CDS Architecture
- [ ] Draft table based on CDS field names (not DB table)?
- [ ] `@Metadata.allowExtensions: true` on projection?
- [ ] No `@UI` annotations in CDS Base or Projection?
- [ ] Correct provider contract on projection?
- [ ] 5 admin fields present with @Semantics annotations?

### 6. UI Annotations
- [ ] ALL in the Metadata Extension?
- [ ] `@UI.headerInfo` defined?
- [ ] `@UI.facet` structured correctly (COLLECTION + FIELDGROUP_REFERENCE)?
- [ ] Actions exposed in both `@UI.lineItem` AND `@UI.identification`?
- [ ] Value Helps: `@Consumption.valueHelpDefinition` set?

### 7. Feature Control & Side Effects
- [ ] Dynamic feature control on conditional actions (`features : instance`)?
- [ ] Calculated/derived fields declared readonly?
- [ ] Side effects declared for fields that trigger recalculations?
- [ ] `get_instance_features` implementation correct (enabled/disabled/read_only)?

### 8. Actions
- [ ] Correct action type used (instance/static/factory)?
- [ ] Abstract entity created for action parameters?
- [ ] Result type specified (`result [1] $self` or `result [*] $self`)?
- [ ] Factory actions return `mapped` correctly?

### 9. Error Handling
- [ ] Validations with explicit messages?
- [ ] FAILED/REPORTED filled correctly?
- [ ] `foreign entity` + `MAP_MESSAGES` if cross-BO EML without association?
- [ ] Unmanaged save: errors from BAPIs/APIs properly caught and reported?

### 10. Performance
- [ ] No SELECT inside a loop?
- [ ] `READ ENTITIES` with specific FIELDS (not ALL FIELDS in loops)?
- [ ] CDS associations with correct cardinality?
- [ ] No unnecessary READ before MODIFY in determinations?

---

## Output Format

```
## VERDICT: ✅ APPROVED | ⚠️ CHANGES REQUESTED | ❌ REJECTED

### Issues Found
1. [SEVERITY: HIGH/MEDIUM/LOW] Issue description
   - File: {name}
   - Line: {n}
   - Proposed fix: {code}

### Positive Points
- ...

### Improvement Suggestions (non-blocking)
- ...
```
