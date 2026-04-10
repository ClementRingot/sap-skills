---
name: clean-core
description: This skill defines the mandatory Clean Core / ABAP Cloud guardrails for all SAP development. It covers object release status verification, wrapper patterns, ABAP Cloud types, and constraints by environment (Private Cloud, Public Cloud, BTP).
---

## Fundamental Rules

### 1. Systematic Object Verification

Before using **any** SAP standard object (class, interface, table, CDS, BAPI, function module):

1. Verify its release status (via ADT, SAP API Business Hub, or the GitHub repository SAP/abap-atc-cr-cv-s4hc)
2. If **Released (C1)** → direct usage allowed
3. If **Classic API (C2)** → acceptable in Private Cloud, forbidden in Public Cloud/BTP
4. If **Not Released** → create a wrapper if On-Premise, not possible in Cloud

### 2. Behavior Definition

- `strict ( 2 )` mandatory in all BDEFs
- `managed` or `managed with unmanaged save` — never `unmanaged` alone
- No direct SELECT on non-released standard tables

### 3. Wrapper Pattern

When a standard object is required but not released:

```abap
CLASS zcl_wrapper_{name} DEFINITION
  PUBLIC FINAL
  CREATE PUBLIC.

  PUBLIC SECTION.
    METHODS execute
      IMPORTING is_input  TYPE zs_wrapper_input
      RETURNING VALUE(rs_output) TYPE zs_wrapper_output
      RAISING zcx_wrapper_error.
ENDCLASS.
```

> ⚠️ Wrappers are NOT POSSIBLE in S/4HANA Public Cloud and BTP ABAP Environment.

### 4. Availability by Environment

| Feature | Private Cloud | Public Cloud | BTP |
|---------|:---:|:---:|:---:|
| RAP BO Standard (I_*TP) | ✅ | ✅ | ✅ |
| Cross-BO Association | ✅ | ✅ | ✅ |
| Released APIs (C1) | ✅ | ✅ | ✅ |
| Classic APIs (C2) | ✅ | ❌ | ❌ |
| Non-released BAPIs | ✅ (wrapper) | ❌ | ❌ |
| Standard tables (read) | ✅ | ❌ | ❌ |

### 5. Preferred ABAP Cloud Types

- UUID: `sysuuid_x16`
- Timestamps: `abp_creation_tstmpl`, `abp_lastchange_tstmpl`, `abp_locinst_lastchange_tstmpl`
- Users: `abp_creation_user`, `abp_locinst_lastchange_user`
- Boolean: `abap_boolean`
- Amounts: `curr` with currency reference
- Quantities: `quan` with unit reference

---

## Clean Core Checklist

```
□ All SAP standard objects verified for release status
□ No direct SELECT on non-released standard tables
□ No CALL FUNCTION on non-released function modules
□ strict ( 2 ) in all BDEFs
□ ABAP Cloud types used (no obsolete types)
□ Wrappers created for non-released objects (On-Premise only)
□ No standard code modifications
□ No classic Enhancement Spots / Exits
```
