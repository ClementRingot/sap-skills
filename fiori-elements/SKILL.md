---
name: fiori-elements
description: This skill guides the creation of Fiori Elements applications based on RAP. It covers template selection (List Report, Worklist, OVP, ALP, Form Entry), UI annotations, Metadata Extension, Value Helps, CDS Projection, and OData V4 Service Binding.
---

## Template Selection

| Template | Use Case | Provider Contract |
|----------|----------|-------------------|
| List Report + Object Page | Standard CRUD, master data management | `transactional_query` |
| Worklist | Work queue, sequential processing | `transactional_query` |
| Overview Page (OVP) | Dashboard, KPIs, overview | `transactional_query` |
| Analytical List Page (ALP) | Analysis, interactive reporting | `transactional_query` |
| Form Entry Object Page | Simple form entry (no list) | `transactional_query` |
| Freestyle (FPM) | Very specific cases not covered above | — |

### Decision Tree

```
Primary need?
├── Display a list + details → List Report + Object Page
├── Process items one by one → Worklist
├── See KPIs + drill-down → Overview Page
├── Analyze data + charts → Analytical List Page
├── Enter a form without a list → Form Entry Object Page
└── Nothing fits → Freestyle (FPM)
```

---

## Annotations — Absolute Rule

> 📱 **ALL UI annotations in the Metadata Extension — NEVER in the CDS views.**

### Metadata Extension — Standard Template

```cds
@Metadata.layer: #CUSTOMER

annotate view ZC_{NAME}TP with
{
  " === Header Info ===
  @UI.headerInfo: {
    typeName: '{EntityLabel}',
    typeNamePlural: '{EntityLabelPlural}',
    title: { type: #STANDARD, value: 'KeyField' },
    description: { type: #STANDARD, value: 'DescriptionField' }
  }

  " === Facets (Object Page) ===
  @UI.facet: [
    { id: 'GeneralInfo',
      type: #COLLECTION,
      label: 'General Information',
      position: 10 },
    { id: 'GeneralData',
      parentId: 'GeneralInfo',
      type: #FIELDGROUP_REFERENCE,
      targetQualifier: 'GeneralData',
      position: 10 },
    { id: 'AdminData',
      type: #FIELDGROUP_REFERENCE,
      label: 'Admin Data',
      targetQualifier: 'AdminData',
      position: 20 }
  ]

  " === Fields ===
  @UI.lineItem: [{ position: 10 }]
  @UI.selectionField: [{ position: 10 }]
  @UI.fieldGroup: [{ qualifier: 'GeneralData', position: 10 }]
  KeyField;

  @UI.lineItem: [{ position: 20 }]
  @UI.fieldGroup: [{ qualifier: 'GeneralData', position: 20 }]
  DescriptionField;

  " === Admin Fields (readonly) ===
  @UI.fieldGroup: [{ qualifier: 'AdminData', position: 10 }]
  CreatedBy;

  @UI.fieldGroup: [{ qualifier: 'AdminData', position: 20 }]
  CreatedAt;
}
```

### Common Annotations

| Annotation | Usage |
|-----------|-------|
| `@UI.lineItem` | Columns in the list |
| `@UI.selectionField` | Filters at the top of the list |
| `@UI.identification` | Fields on the Object Page (header section) |
| `@UI.fieldGroup` | Field groups inside facets |
| `@UI.facet` | Object Page sections |
| `@UI.headerInfo` | Entity title and description |
| `@UI.dataPoint` | KPIs and highlighted values |
| `@UI.chart` | Charts (ALP, OVP) |

### Criticality (Colors / Status)

```cds
@UI.lineItem: [{ position: 30, criticality: 'StatusCriticality' }]
Status;
```

Criticality values: 0 = neutral, 1 = red (error), 2 = yellow (warning), 3 = green (success).

### Actions in the UI

```cds
@UI.lineItem: [{ type: #FOR_ACTION, dataAction: 'approve', label: 'Approve' }]
@UI.identification: [{ type: #FOR_ACTION, dataAction: 'approve', label: 'Approve' }]
KeyField;
```

---

## Value Helps

### Value Help CDS View

```cds
@EndUserText.label: 'Status Value Help'
@AccessControl.authorizationCheck: #NOT_REQUIRED
@ObjectModel.resultSet.sizeCategory: #XS

define view entity ZI_STATUS_VH
  as select from zstatus_table
{
  key StatusId,
      StatusDescription
}
```

Naming convention: `ZI_{NAME}_VH`.

### Value Help Annotation (in Metadata Extension)

```cds
@Consumption.valueHelpDefinition: [{
  entity: { name: 'ZI_STATUS_VH', element: 'StatusId' },
  additionalBinding: [{ localElement: 'StatusDescription',
                         element: 'StatusDescription',
                         usage: #RESULT }]
}]
Status;
```

### Value Help with Filter Dependency

```cds
@Consumption.valueHelpDefinition: [{
  entity: { name: 'ZI_CITY_VH', element: 'CityId' },
  additionalBinding: [{ localElement: 'CountryCode',
                         element: 'CountryCode',
                         usage: #FILTER }]
}]
CityId;
```

`#FILTER` restricts the value help results based on the current value of another field.
`#RESULT` fills another field when a value is selected.

---

## CDS Projection — Standard Template

```cds
@EndUserText.label: '{EntityLabel}'
@AccessControl.authorizationCheck: #NOT_REQUIRED
@Metadata.allowExtensions: true       " ← Required for Metadata Extension

define root view entity ZC_{NAME}TP
  provider contract transactional_query
  as projection on ZR_{NAME}TP
{
  key KeyField,
      Field1,
      Field2,
      " Associations
      _ChildEntity : redirected to composition child ZC_ChildTP,
      _ValueHelp   : redirected to ZI_STATUS_VH
}
```

> `@Metadata.allowExtensions: true` is mandatory for the Metadata Extension to work.

---

## Service Definition

```cds
@EndUserText.label: '{ServiceLabel}'
define service ZUI_{NAME}_O4 {
  expose ZC_{NAME}TP as {EntityName};
  expose ZC_ChildTP as {ChildName};
  expose ZI_STATUS_VH as StatusValueHelp;
}
```

---

## Fiori Elements Checklist

```
□ Fiori template chosen based on use case
□ Metadata Extension created (@Metadata.layer: #CUSTOMER)
□ @Metadata.allowExtensions: true on the CDS Projection
□ NO @UI annotations in CDS Base or Projection views
□ @UI.headerInfo defined
□ @UI.facet structured (COLLECTION + FIELDGROUP_REFERENCE)
□ @UI.lineItem on list columns
□ @UI.selectionField on filters
□ Actions exposed in @UI.lineItem AND @UI.identification
□ Value Helps: ZI_*_VH created + @Consumption.valueHelpDefinition set
□ Service Binding OData V4 published
□ Preview tested
```
