---
name: rap-business-object
description: This skill guides the complete creation of a RAP Business Object (RESTful ABAP Programming Model) in ABAP Cloud. It covers CDS data model, behavior definition, behavior implementation, draft handling, numbering, actions, feature control, side effects, cross-BO interactions (with cross associations, foreign entity), unmanaged save, and integration with SAP standard objects.
---

## CDS Architecture — Layer Stack

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  Service Binding (OData V4)                                                 │
│  ZUI_{NAME}_O4 / ZAPI_{NAME}                                                │
│    └────────┬─────────┘                                                     │
│             │ expose                                                        │
│    ┌────────▼─────────────────┐                                             │
│    │  Service Definition      │  ZC_{NAME}TP (UI) / ZI_{NAME}TP (API)       │
│    │                          │                                             │
│    └────────┬─────────────────┘                                             │
│             │ expose                                                        │
│    ┌────────▼─────────────────┐                                             │
│    │  CDS Projection          │  Z{NAME}                                    │
│    │  + Metadata Extension    │  (UI annotations in MetaExt only)           │
│    └────────┬─────────────────┘                                             │
│             │ projection on                                                 │
│    ┌────────▼─────────┐                                                     │
│    │  CDS BO          │  ZR_{NAME}TP + Behavior Definition                  │
│    │  (Root Entity)   │  + Behavior Implementation (ZBP_R_{NAME}TP)         │
│    └────────┬─────────┘                                                     │
│             │ select from                                                   │
│    ┌────────▼─────────┐                                                     │
│    │  CDS Base        │  ZI_{NAME}                                          │
│    │  (Interface)     │                                                     │
│    └────────┬─────────┘                                                     │
│             │ select from                                                   │
│    ┌────────▼─────────┐    ┌─────────────────┐                              │
│    │  DB Table        │    │  Draft Table    │                              │
│    │  Z{NAME}         │    │  Z{NAME}_D      │                              │
│    └──────────────────┘    └─────────────────┘                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Naming Conventions (MANDATORY)

### CDS Views

| Type | Pattern | Provider Contract | Example |
|------|---------|-------------------|---------|
| Base / Interface View | `ZI_{NAME}` | — | `ZI_DOCKAPPOINTMENT` |
| Value Help | `ZI_{NAME}_VH` | — | `ZI_STATUS_VH` |
| BO View (Root) | `ZR_{NAME}TP` | — | `ZR_DOCKAPPOINTMENTTP` |
| BO View (Child) | `ZR_{NAME}TP` | — | `ZR_APPOINTMENTITEMTP` |
| Projection UI | `ZC_{NAME}TP` | `transactional_query` | `ZC_DOCKAPPOINTMENTTP` |
| Projection API | `ZI_{NAME}TP` | `transactional_interface` | `ZI_DOCKAPPOINTMENTTP` |
| Abstract Entity | `ZA_{NAME}` | — | `ZA_APPROVEPARAMS` |
| Custom Entity | `ZCE_{NAME}` | — | `ZCE_EXTERNALDATA` |

### Tables

| Type | Pattern | Example |
|------|---------|---------|
| Persistence table | `Z{NAME}` | `ZDOCKAPPOINTMENT` |
| Draft table | `Z{NAME}_D` | `ZDOCKAPPOINTMENT_D` |

### Other Objects

| Type | Pattern | Example |
|------|---------|---------|
| Behavior Pool | `ZBP_R_{NAME}TP` | `ZBP_R_DOCKAPPOINTMENTTP` |
| Service Definition | `Z{NAME}` | `ZDOCKAPPOINTMENT` |
| Service Binding UI | `ZUI_{NAME}_O4` | `ZUI_DOCKAPPOINTMENT_O4` |
| Service Binding API | `ZAPI_{NAME}` | `ZAPI_DOCKAPPOINTMENT` |
| Metadata Extension | `ZC_{NAME}TP` | `ZC_DOCKAPPOINTMENTTP` |

### Casing Rules

- **Table** names: UPPERCASE (snake_case for fields)
- **CDS** element names: CamelCase
- **Admin fields**: always CamelCase (`CreatedBy`, `CreatedAt`, `LastChangedBy`, `LastChangedAt`, `LocalLastChangedAt`)

---

## Object Creation Order

```
1.  Persistence table (Z{NAME})
2.  CDS Base (ZI_{NAME})
3.  Draft table (Z{NAME}_D) ← AFTER the CDS Base!
4.  CDS BO View (ZR_{NAME}TP)
5.  CDS Projection (ZC_{NAME}TP or ZI_{NAME}TP)
6.  Behavior Definition (same name as CDS BO)
7.  Behavior Implementation (ZBP_R_{NAME}TP)
8.  Metadata Extension (if UI)
9.  Service Definition
10. Service Binding + Publish
```

> ⚠️ **CRITICAL RULE**: The draft table must be created AFTER the CDS Base.
> It must use the **same field names as the CDS** (not the persistence table).

---

## Mandatory Admin Fields

Every persistence table MUST contain these 5 fields:

```abap
created_by            : abp_creation_user;
created_at            : abp_creation_tstmpl;
last_changed_by       : abp_locinst_lastchange_user;
last_changed_at       : abp_lastchange_tstmpl;
local_last_changed_at : abp_locinst_lastchange_tstmpl;
```

Corresponding annotations in the CDS Base:

```cds
@Semantics.user.createdBy: true
CreatedBy,
@Semantics.systemDateTime.createdAt: true
CreatedAt,
@Semantics.user.lastChangedBy: true
LastChangedBy,
@Semantics.systemDateTime.lastChangedAt: true
LastChangedAt,
@Semantics.systemDateTime.localInstanceLastChangedAt: true
LocalLastChangedAt
```

---

## Behavior Definition — Standard Template

```abap
managed implementation in class zbp_r_{name}tp unique;
strict ( 2 );
with draft;

define behavior for ZR_{NAME}TP alias {Alias}
persistent table z{name}
draft table z{name}_d
etag master LocalLastChangedAt
lock master total etag LastChangedAt
authorization master ( global )
{
  create;
  update;
  delete;

  " Fields
  field ( mandatory ) RequiredField1, RequiredField2;
  field ( readonly ) CreatedBy, CreatedAt, LastChangedBy, LastChangedAt, LocalLastChangedAt;
  field ( readonly : update ) KeyField;

  " Draft
  draft action Edit;
  draft action Activate optimized;
  draft action Discard;
  draft action Resume;
  draft determine action Prepare;

  " Validations / Determinations
  validation validateRequiredFields on save { create; update; }
  determination setDefaults on modify { create; }

  " Mapping
  mapping for z{name}
  {
    KeyField = key_field;
    Field1 = field_1;
    CreatedBy = created_by;
    CreatedAt = created_at;
    LastChangedBy = last_changed_by;
    LastChangedAt = last_changed_at;
    LocalLastChangedAt = local_last_changed_at;
  }
}
```

### BDEF Rules

- **ALWAYS** `strict ( 2 )` in the header
- **ALWAYS** `managed` or `managed with unmanaged save` — **NEVER** `unmanaged` alone
- **ALWAYS** enable `with draft` for transactional UI applications
- **ALWAYS** include complete field mapping

---

## Numbering

### Decision Tree

```
What type of key does the entity use?
├── UUID (auto-generated, no business meaning)
│   └── Managed numbering
│       field ( readonly ) MyUUID;
│       The framework generates the UUID automatically.
│
├── Sequential business number (e.g., order number, document ID)
│   └── Late numbering
│       The key is assigned in the save phase via adjust_numbers.
│       Use number ranges or custom logic.
│
└── Key provided by the user or external system
    └── Early numbering
        The key is provided at creation time by the consumer.
```

### Managed Numbering (UUID)

```abap
define behavior for ZR_OrderTP alias Order
persistent table zorder
{
  field ( readonly ) OrderUUID;  " Framework generates UUID
  ...
}
```

### Late Numbering

```abap
define behavior for ZR_OrderTP alias Order
late numbering
persistent table zorder
{
  field ( readonly ) OrderId;
  ...
}
```

Implementation in the behavior pool:

```abap
METHOD adjust_numbers.
  " Called during save phase — assign final keys here
  LOOP AT mapped-order ASSIGNING FIELD-SYMBOL(<order>).
    " Get next number from number range or custom logic
    <order>-OrderId = get_next_order_number( ).
  ENDLOOP.
ENDMETHOD.
```

### Early Numbering

```abap
define behavior for ZR_OrderTP alias Order
early numbering
persistent table zorder
{
  field ( readonly : update ) OrderId;  " Provided on create, immutable after
  ...
}
```

---

## Actions

### Instance Action (operates on a specific instance)

```abap
" BDEF
action approve result [1] $self;
```

```abap
" Implementation
METHOD approve.
  MODIFY ENTITIES OF zr_ordertp IN LOCAL MODE
    ENTITY Order
    UPDATE FIELDS ( Status )
    WITH VALUE #( FOR key IN keys
      ( %tky = key-%tky  Status = 'APPROVED' ) )
    FAILED failed
    REPORTED reported.

  READ ENTITIES OF zr_ordertp IN LOCAL MODE
    ENTITY Order ALL FIELDS
    WITH CORRESPONDING #( keys )
    RESULT DATA(orders).

  result = VALUE #( FOR order IN orders
    ( %tky = order-%tky  %param = order ) ).
ENDMETHOD.
```

### Action with Parameters (Abstract Entity)

```abap
" Abstract entity for parameters
define abstract entity ZA_RejectParams
{
  Reason : abap.string( 256 );
}
```

```abap
" BDEF
action reject parameter ZA_RejectParams result [1] $self;
```

```abap
" Implementation
METHOD reject.
  DATA(lv_reason) = keys[ 1 ]-%param-Reason.
  ...
ENDMETHOD.
```

### Static Action (not bound to an instance)

```abap
" BDEF
static action massApprove parameter ZA_MassApproveParams result [*] $self;
```

### Factory Action (creates a new instance from an existing one)

```abap
" BDEF
factory action copyOrder [1];
```

```abap
" Implementation
METHOD copyOrder.
  READ ENTITIES OF zr_ordertp IN LOCAL MODE
    ENTITY Order ALL FIELDS
    WITH CORRESPONDING #( keys )
    RESULT DATA(orders).

  MODIFY ENTITIES OF zr_ordertp IN LOCAL MODE
    ENTITY Order
    CREATE FIELDS ( Field1 Field2 )
    WITH VALUE #( FOR order IN orders
      ( %cid = |COPY_{ order-OrderId }|
        Field1 = order-Field1
        Field2 = order-Field2 ) )
    MAPPED mapped
    FAILED failed
    REPORTED reported.
ENDMETHOD.
```

---

## Feature Control (Static + Dynamic)

### Static Feature Control (in BDEF only)

```abap
field ( readonly : update ) OrderId;        " Immutable after create
field ( mandatory : create ) CustomerId;    " Required on create only
field ( readonly ) TotalAmount;             " Always readonly (calculated)
```

### Dynamic Feature Control (BDEF + implementation)

```abap
" BDEF — declare features : instance
action ( features : instance ) approve;
action ( features : instance ) reject;
field ( features : instance ) Status;
```

```abap
" Implementation
METHOD get_instance_features.
  READ ENTITIES OF zr_ordertp IN LOCAL MODE
    ENTITY Order
    FIELDS ( Status )
    WITH CORRESPONDING #( keys )
    RESULT DATA(orders).

  result = VALUE #( FOR order IN orders
    ( %tky = order-%tky
      " Disable approve/reject if already approved
      %action-approve = COND #(
        WHEN order-Status = 'APPROVED'
        THEN if_abap_behv=>fc-o-disabled
        ELSE if_abap_behv=>fc-o-enabled )
      %action-reject = COND #(
        WHEN order-Status = 'APPROVED'
        THEN if_abap_behv=>fc-o-disabled
        ELSE if_abap_behv=>fc-o-enabled )
      " Make Status field readonly
      %field-Status = if_abap_behv=>fc-f-read_only
    ) ).
ENDMETHOD.
```

Feature control constants:
- Actions: `if_abap_behv=>fc-o-enabled` / `if_abap_behv=>fc-o-disabled`
- Fields: `if_abap_behv=>fc-f-read_only` / `if_abap_behv=>fc-f-mandatory` / `if_abap_behv=>fc-f-unrestricted`

---

## Side Effects

Side effects tell the Fiori Elements UI which fields to refresh after a value changes, avoiding a full round-trip.

### In the BDEF

```abap
side effects
{
  " When Quantity or Price changes → recalculate TotalAmount
  field Quantity affects field TotalAmount;
  field Price affects field TotalAmount;

  " When CustomerId changes → refresh the _Customer association
  field CustomerId affects entity _Customer;

  " When an action is executed → refresh the whole entity
  determine action Prepare executed on field Status affects entity _Order;
}
```

### Rules

- Side effects are declared in the **BDEF entity behavior**, not in annotations.
- Fiori Elements consumes them automatically — no annotation needed on the UI side.
- The `affects` target can be: `field`, `entity` (association), or `$self` (whole instance).
- Side effects trigger a re-read of the affected fields/entities on the UI after a PATCH.

---

## RAP Transactional Model — Phases

```
┌─────────────────────────────────────────────────────────────────┐
│                    INTERACTION PHASE                             │
│  (Determinations on modify, Actions, Side Effects)              │
│                                                                 │
│  ✅ READ + MODIFY allowed on own BO and cross-BO               │
│  ❌ No COMMIT ENTITIES (not even IN SIMULATION MODE)           │
│  The transactional buffer is modified but not persisted.        │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    VALIDATION PHASE                              │
│  (Validations on save, draft determine action Prepare)          │
│                                                                 │
│  ✅ READ allowed on own BO and cross-BO                        │
│  ❌ MODIFY not allowed                                         │
│  If a validation fails → FAILED → entire SAVE rejected.        │
│                                                                 │
│  If a cross-BO validation fails, EVERYTHING is rejected         │
│  (including your BO). This is the "all or nothing" behavior.   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    SAVE PHASE                                    │
│  (Determinations on save, adjust_numbers, save_modified,        │
│   map_messages, cleanup_finalize)                               │
│                                                                 │
│  The framework handles the COMMIT for all BOs together.         │
│  ❌ No cross-BO EML calls in save phase.                       │
│  ❌ No COMMIT ENTITIES / COMMIT WORK                           │
└─────────────────────────────────────────────────────────────────┘
```

### Fundamental Transactional Rules

1. **NEVER** call `COMMIT ENTITIES` inside a RAP handler (determination, validation, action, save). The framework handles it.
2. **NEVER** call `COMMIT WORK` inside a RAP handler.
3. All BOs modified in the same LUW **live and die together** — if one BO fails validation, everything is rejected.
4. To decouple transactions → use `Additional Save` with `PERFORM ... ON COMMIT` (but this breaks the transactional model, use with caution).

---

## Unmanaged Save — `save_modified` Template

Use `managed with unmanaged save` when persistence goes through a standard API (BAPI, Released API) rather than direct INSERT.

### BDEF Header

```abap
managed with unmanaged save implementation in class zbp_r_{name}tp unique;
strict ( 2 );
with draft;
```

### `save_modified` Implementation

```abap
METHOD save_modified.
  " CREATE
  IF create-order IS NOT INITIAL.
    LOOP AT create-order INTO DATA(ls_create).
      " Call BAPI / Released API / Function Module
      CALL FUNCTION 'Z_WRAPPER_CREATE_ORDER'
        EXPORTING is_order = CORRESPONDING #( ls_create )
        IMPORTING ev_order_id = DATA(lv_id)
        EXCEPTIONS OTHERS = 1.
      IF sy-subrc <> 0.
        " Handle error → fill reported
      ENDIF.
    ENDLOOP.
  ENDIF.

  " UPDATE
  IF update-order IS NOT INITIAL.
    LOOP AT update-order INTO DATA(ls_update).
      " Call update API
    ENDLOOP.
  ENDIF.

  " DELETE
  IF delete-order IS NOT INITIAL.
    LOOP AT delete-order INTO DATA(ls_delete).
      " Call delete API
    ENDLOOP.
  ENDIF.
ENDMETHOD.
```

### When to Use Unmanaged Save

- The target persistence is a **SAP standard table** that must be written via BAPI/API (not direct INSERT).
- You need to call **external systems** during persistence (RFC, HTTP).
- You wrap an existing **non-RAP API** behind a RAP facade.

> In all other cases, use plain `managed` — let the framework handle persistence.

---

## Integration with SAP Standard — Cross-BO Interactions

### Decision Tree

```
Is the target object a Released RAP BO (I_*TP)?
    │
    ├── YES → Is there a FK relationship between the two entities?
    │         │
    │         ├── YES → with cross associations
    │         │         (Framework orchestrates everything:
    │         │          read/create by association, link/unlink)
    │         │
    │         └── NO  → foreign entity
    │                   (Your EML code in determination/action,
    │                    message mapping via MAP_MESSAGES)
    │
    ├── Released API exists (but not a RAP BO)?
    │         └── Call in save_modified (unmanaged save)
    │
    ├── BAPI only option (On-Premise)?
    │         └── Create wrapper class → call in save_modified
    │
    └── No API available (Cloud)?
              └── Not possible. Request from SAP.
```

### `with cross associations` vs `foreign entity` vs Raw EML call

| Aspect | `with cross associations` | `foreign entity` | Raw EML (nothing declared) |
|--------|--------------------------|-------------------|-----------------------------|
| **Purpose** | Full cross-BO transactional enablement | Cross-BO message mapping | Modify another BO |
| **CDS association required** | ✅ Yes (FK between entities) | ❌ No | ❌ No |
| **OData navigation** | ✅ read/create by association | ❌ No | ❌ No |
| **Who triggers** | Client (Fiori UI) via association | Your EML code in determination/action | Your EML code in determination/action |
| **Message mapping** | Automatic (foreign entity auto-included) | Via `MAP_MESSAGES` in saver class | ❌ Messages lost |
| **Late numbering support** | ✅ Orchestrated by framework | ❌ Handle manually | ❌ Handle manually |
| **BDEF header** | `with cross associations;` | `foreign entity {BO};` | Nothing |
| **Approach** | Declarative — framework does everything | Imperative — your code + message relay | Imperative — your code, messages lost |

### When to Use What

**`with cross associations`** → When you want to expose a **navigable transactional relationship** between two BOs.
The Fiori client can directly create/read via the association. The framework orchestrates everything,
including complex scenarios (late numbering → `adjust_numbers` → `link action`).
Prerequisite: a CDS association with ON condition and a FK in one of the two entities.

```abap
" BDEF header
managed implementation in class zbp_r_ordertp unique;
strict ( 2 );
with cross associations;
with draft;

define behavior for ZR_OrderTP alias Order
...
{
  association _DeliveryDocument { with draft; create; }
}
```

**`foreign entity`** → When you call another BO via EML **without an association** and need to
capture and remap the error messages from the called BO.
Typical use case: an **action** like "Generate Invoice" that creates a separate BO via EML.

```abap
" BDEF header
managed implementation in class zbp_r_ordertp unique;
strict ( 2 );
with draft;

foreign entity ZR_InvoiceTP;  " ← Enables message mapping

define behavior for ZR_OrderTP alias Order
...
{
  action createInvoice result [1] $self;
}
```

Action implementation:

```abap
METHOD createInvoice.
  " EML call to an independent BO (no association)
  MODIFY ENTITIES OF zr_invoicetp
    ENTITY Invoice
    CREATE FROM VALUE #( ( %cid = 'INV1' ... ) )
    MAPPED DATA(mapped_inv)
    FAILED DATA(failed_inv)
    REPORTED DATA(reported_inv).

  " Messages are in reported_inv.
  " Thanks to foreign entity, they will be available in
  " reported-zr_invoicetp for MAP_MESSAGES.
ENDMETHOD.
```

Saver class (optional but recommended):

```abap
CLASS lsc DEFINITION INHERITING FROM cl_abap_behavior_saver.
  PROTECTED SECTION.
    METHODS map_messages REDEFINITION.
ENDCLASS.

CLASS lsc IMPLEMENTATION.
  METHOD map_messages.
    LOOP AT reported-zr_invoicetp INTO DATA(ls_inv_msg).
      APPEND VALUE #(
        %tky = <corresponding_key_of_my_bo>
        %msg = ls_inv_msg-%msg
      ) TO reported-order.
    ENDLOOP.
    CLEAR reported-zr_invoicetp.
  ENDMETHOD.
ENDCLASS.
```

**Raw EML call (nothing declared)** → **AVOID**. Messages from the called BO do not surface
to your BO or to the Fiori UI. Errors are silently lost — the user sees nothing.
Always use at least `foreign entity` when making cross-BO EML calls.

### Details on `with cross associations`

The `with cross associations` keyword in the BDEF header:

1. **Applies to the entire BO** — cannot be configured per association.
2. **Activates the managed BO provider** for all associations (except those marked `unmanaged`).
3. **Automatically provides**: read-by-association, create-by-association, link action, unlink action, inverse function.
4. **Enables additional syntax checks** for cross-BO modeling (errors caught at compile time, not runtime).
5. **Automatically includes foreign entities** for association targets (no separate `foreign entity` needed). This can be disabled with `without response` on the association.
6. **Required for late numbering**: without it, the `adjust_numbers` → `link action` orchestration does not work in managed mode.

> **Best practice**: whenever you do cross-BO on a managed BO, add `with cross associations`.
> Zero cost, full protection. The only case to skip it: you need fine-grained control per association
> (mix managed/unmanaged) and no late numbering on the target side.

Cross-BO also works **without** `with cross associations` by declaring `managed` implementation type
on each association individually, but it is more verbose and lacks the additional compile-time checks.

### Source-Resolved vs Target-Resolved Associations

- **Source-resolved (FK association)**: the FK is held by the source entity. After create-by-association with late numbering, the framework must UPDATE the source to fill the FK → automatically orchestrated with `with cross associations`.
- **Target-resolved (reverse FK association)**: the FK is held by the target entity. Simpler because the FK is filled directly on CREATE of the target. Supports `reverse association` and `inverse function`.

### SAP Standard BO Lookup Priority

ALWAYS check first if a Released RAP standard BO exists:

| Domain | Standard RAP BO |
|--------|----------------|
| Sales Order | `I_SalesOrderTP` |
| Purchase Order | `I_PurchaseOrderTP` |
| Delivery | `I_DeliveryDocumentTP` |
| Inbound Delivery | `I_InboundDeliveryTP` |
| Material Document | `I_MaterialDocumentTP` |
| Product | `I_ProductTP_2` |
| Business Partner | `I_BusinessPartnerTP` |
| Warehouse Task | `I_WarehouseTaskTP` |
| Handling Unit | `I_EWMHandlingUnitTP` |

> ⚠️ This list is not exhaustive. Verify via ADT, SAP API Business Hub, or the GitHub repository SAP/abap-atc-cr-cv-s4hc.

---

## UI Annotations

> 📱 **ABSOLUTE RULE**: ALL UI annotations go in the **Metadata Extension** — NEVER in the CDS views.

The CDS Base = pure data model.
The CDS Projection = provider contract + redirections.
The Metadata Extension = complete UI layout.

---

## Service Binding

- **OData V4 first** — always.
- OData V2 only if there is a technical constraint.
- Naming: `ZUI_{NAME}_O4` (UI) or `ZAPI_{NAME}` (API).

---

## Final Validation Checklist

```
□ CDS names follow ZI_*/ZR_*TP/ZI_*TP/ZC_*TP
□ Table names follow Z{NAME}/Z{NAME}_D
□ CDS fields in CamelCase
□ Table fields in snake_case
□ 5 admin fields present (CreatedBy, CreatedAt, LastChangedBy, LastChangedAt, LocalLastChangedAt)
□ @Semantics annotations on admin fields
□ BDEF is managed or managed with unmanaged save
□ strict ( 2 ) in BDEF header
□ with draft enabled (if UI)
□ Draft table created (after CDS Base, based on CDS field names)
□ Complete field mapping in BDEF
□ Service Binding is OData V4
□ Metadata Extension with UI annotations (not in CDS)
□ Numbering strategy chosen (managed/early/late)
□ Feature control on conditional actions/fields
□ Side effects declared for calculated fields
□ Cross-BO: with cross associations if FK exists, foreign entity if EML call without association
□ No COMMIT ENTITIES / COMMIT WORK in RAP handlers
```

---

## References

- Cross-BO Associations: https://help.sap.com/docs/abap-cloud/abap-rap/cross-bo-associations
- `with cross associations` syntax: https://help.sap.com/doc/abapdocu_cp_index_htm/CLOUD/en-US/abenbdl_with_cross_assoc.html
- `foreign entity` syntax: https://help.sap.com/doc/abapdocu_cp_index_htm/CLOUD/en-US/ABENBDL_FOREIGN.html
- Cross-BO Orchestration: https://help.sap.com/docs/ABAP_Cloud/f055b8bf582d4f34b91da667bc1fcce6/1cce9f381e87417a8450b52e18eaff00.html
- Message Mapping: https://help.sap.com/docs/ABAP_PLATFORM_NEW/fc4c71aa50014fd1b43721701471913d/741dfc88293d4ac99186815b9fc35e31.html
- Naming Conventions: https://help.sap.com/docs/abap-cloud/abap-rap/naming-conventions
