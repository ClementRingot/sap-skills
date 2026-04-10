# SAPUI5 & Fiori Elements — Patterns & Anti-Patterns Reference

Read the relevant sections of this file before reviewing code. It contains canonical examples of correct vs incorrect patterns.

## Table of Contents
1. [XML Annotations](#1-xml-annotations)
2. [Building Blocks (Macros)](#2-building-blocks-macros)
3. [Controller Extensions (FPM)](#3-controller-extensions-fpm)
4. [XML Views Common Patterns](#4-xml-views-common-patterns)
5. [OData V4 Binding Patterns](#5-odata-v4-binding-patterns)
6. [manifest.json Patterns](#6-manifestjson-patterns)
7. [i18n Conventions](#7-i18n-conventions)
8. [Legacy API Migration Table](#8-legacy-api-migration-table)

---

## 1. XML Annotations

### Correct annotation file structure

```xml
<Annotations xmlns="http://docs.oasis-open.org/odata/ns/edm"
             xmlns:UI="com.sap.vocabularies.UI.v1"
             xmlns:Common="com.sap.vocabularies.Common.v1"
             xmlns:Communication="com.sap.vocabularies.Communication.v1">

  <Annotations Target="SAP__self.ProductType">
    <!-- HeaderInfo -->
    <Annotation Term="UI.HeaderInfo">
      <Record Type="UI.HeaderInfoType">
        <PropertyValue Property="TypeName" String="Product"/>
        <PropertyValue Property="TypeNamePlural" String="Products"/>
        <PropertyValue Property="Title">
          <Record Type="UI.DataField">
            <PropertyValue Property="Value" Path="ProductName"/>
          </Record>
        </PropertyValue>
        <PropertyValue Property="Description">
          <Record Type="UI.DataField">
            <PropertyValue Property="Value" Path="ProductID"/>
          </Record>
        </PropertyValue>
      </Record>
    </Annotation>
  </Annotations>
</Annotations>
```

### Common annotation mistakes

**Wrong**: Using EntitySet name as target instead of EntityType
```xml
<!-- WRONG -->
<Annotations Target="SAP__self.Products">
<!-- CORRECT -->
<Annotations Target="SAP__self.ProductType">
```

**Wrong**: Missing Record Type in DataField
```xml
<!-- WRONG -->
<Annotation Term="UI.LineItem">
  <Collection>
    <Record>
      <PropertyValue Property="Value" Path="Name"/>
    </Record>
  </Collection>
</Annotation>

<!-- CORRECT -->
<Annotation Term="UI.LineItem">
  <Collection>
    <Record Type="UI.DataField">
      <PropertyValue Property="Value" Path="Name"/>
      <PropertyValue Property="Label" String="{i18n>name}"/>
    </Record>
  </Collection>
</Annotation>
```

**Wrong**: DataFieldForAction with wrong Action path
```xml
<!-- WRONG — missing namespace or bound action prefix -->
<Record Type="UI.DataFieldForAction">
  <PropertyValue Property="Action" String="approve"/>
</Record>

<!-- CORRECT — fully qualified bound action -->
<Record Type="UI.DataFieldForAction">
  <PropertyValue Property="Action"
                 String="com.sap.gateway.srvd_a2x.zui_product_o4.v0001.approve"/>
  <PropertyValue Property="Label" String="{i18n>approve}"/>
</Record>
```

### UI.Facets structure

```xml
<Annotation Term="UI.Facets">
  <Collection>
    <!-- Reference Facet pointing to a FieldGroup -->
    <Record Type="UI.ReferenceFacet">
      <PropertyValue Property="ID" String="GeneralInfo"/>
      <PropertyValue Property="Label" String="{i18n>generalInfo}"/>
      <PropertyValue Property="Target"
                     AnnotationPath="@UI.FieldGroup#General"/>
    </Record>
    <!-- Collection Facet grouping multiple Reference Facets -->
    <Record Type="UI.CollectionFacet">
      <PropertyValue Property="ID" String="Details"/>
      <PropertyValue Property="Label" String="{i18n>details}"/>
      <PropertyValue Property="Facets">
        <Collection>
          <Record Type="UI.ReferenceFacet">
            <PropertyValue Property="ID" String="Pricing"/>
            <PropertyValue Property="Target"
                           AnnotationPath="@UI.FieldGroup#Pricing"/>
          </Record>
        </Collection>
      </PropertyValue>
    </Record>
  </Collection>
</Annotation>
```

### Common.ValueList

```xml
<Annotation Term="Common.ValueList">
  <Record Type="Common.ValueListType">
    <PropertyValue Property="CollectionPath" String="I_Currency"/>
    <PropertyValue Property="Label" String="Currency"/>
    <PropertyValue Property="Parameters">
      <Collection>
        <Record Type="Common.ValueListParameterInOut">
          <PropertyValue Property="LocalDataProperty" PropertyPath="Currency"/>
          <PropertyValue Property="ValueListProperty" String="Currency"/>
        </Record>
        <Record Type="Common.ValueListParameterDisplayOnly">
          <PropertyValue Property="ValueListProperty" String="Currency_Text"/>
        </Record>
      </Collection>
    </PropertyValue>
  </Record>
</Annotation>
```

### Common.SideEffects

```xml
<Annotation Term="Common.SideEffects" Qualifier="QuantityChanged">
  <Record Type="Common.SideEffectsType">
    <PropertyValue Property="SourceProperties">
      <Collection>
        <PropertyPath>Quantity</PropertyPath>
      </Collection>
    </PropertyValue>
    <PropertyValue Property="TargetProperties">
      <Collection>
        <PropertyPath>TotalPrice</PropertyPath>
        <PropertyPath>NetAmount</PropertyPath>
      </Collection>
    </PropertyValue>
  </Record>
</Annotation>
```

---

## 2. Building Blocks (Macros)

### Correct usage in a custom section

```xml
<core:FragmentDefinition
    xmlns:core="sap.ui.core"
    xmlns="sap.m"
    xmlns:macros="sap.fe.macros">

  <VBox>
    <macros:Chart
        id="revenueChart"
        contextPath="/Products"
        metaPath="@com.sap.vocabularies.UI.v1.Chart#Revenue"
        personalization="Sort,Type"
        selectionMode="Single"/>

    <macros:Table
        id="itemsTable"
        contextPath="/Products"
        metaPath="@com.sap.vocabularies.UI.v1.LineItem"
        readOnly="true"
        enableExport="true"/>
  </VBox>
</core:FragmentDefinition>
```

### Common building block mistakes

**Wrong**: Missing `id` (required for all building blocks)
```xml
<!-- WRONG -->
<macros:Field metaPath="Name"/>

<!-- CORRECT -->
<macros:Field id="nameField" metaPath="Name"/>
```

**Wrong**: Confusing `metaPath` and `contextPath`
```xml
<!-- WRONG — metaPath should not be the entity set -->
<macros:Table metaPath="/Products"
              contextPath="@com.sap.vocabularies.UI.v1.LineItem"/>

<!-- CORRECT -->
<macros:Table id="productTable"
              contextPath="/Products"
              metaPath="@com.sap.vocabularies.UI.v1.LineItem"/>
```

- `contextPath` = the OData entity set or navigation path (data context)
- `metaPath` = the annotation term path (metadata context)
- When used at Object Page level where context is already bound, `contextPath` can be omitted and `metaPath` can be a relative annotation path.

**Wrong**: Using building blocks outside of a custom section/subsection
Building blocks in Fiori Elements are designed to be used inside custom sections, custom subsections, custom header facets, or custom columns. They need the Fiori Elements templating context.

---

## 3. Controller Extensions (FPM)

### Correct controller extension structure

```javascript
sap.ui.define([
    "sap/ui/core/mvc/ControllerExtension",
    "sap/ui/core/mvc/OverrideExecution",
    "sap/base/Log"
], function (ControllerExtension, OverrideExecution, Log) {
    "use strict";

    return ControllerExtension.extend("my.app.ext.controller.ObjectPageExt", {
        metadata: {
            methods: {
                onBeforeSaveCustom: {
                    "public": true,
                    "final": false,
                    overrideExecution: OverrideExecution.After
                }
            }
        },

        override: {
            /**
             * Override edit flow hooks
             */
            editFlow: {
                onBeforeSave: function (mParameters) {
                    Log.info("Before save triggered");
                    // Return a Promise to allow async validation
                    return new Promise(function (resolve, reject) {
                        // validation logic
                        resolve();
                    });
                },

                onAfterSave: function (mParameters) {
                    // Show success message, navigate, etc.
                }
            },

            /**
             * Override routing
             */
            routing: {
                onBeforeBinding: function (oContext, mParameters) {
                    // Called before the page binds to the context
                },
                onAfterBinding: function (oContext, mParameters) {
                    // Called after binding — good place to load additional data
                }
            }
        }
    });
});
```

### Common controller extension mistakes

**Wrong**: Using `Controller.extend` instead of `ControllerExtension.extend`
```javascript
// WRONG
return Controller.extend("my.app.ObjectPageExt", { ... });
// CORRECT
return ControllerExtension.extend("my.app.ext.controller.ObjectPageExt", { ... });
```

**Wrong**: Forgetting to return a Promise in `onBeforeSave` when doing async work
```javascript
// WRONG — async validation but no Promise returned, FPM won't wait
onBeforeSave: function () {
    this._validateData(); // async
}

// CORRECT
onBeforeSave: function () {
    return this._validateData(); // returns Promise
}
```

**Wrong**: Accessing OData model with V2 API
```javascript
// WRONG — V2 pattern
var oModel = this.getView().getModel();
oModel.read("/Products", { ... });

// CORRECT — V4 pattern
var oModel = this.getView().getModel();
var oListBinding = oModel.bindList("/Products");
oListBinding.requestContexts(0, 100).then(function (aContexts) { ... });
```

### Custom action handler (bound to a button via annotation)

```javascript
// In manifest.json this is declared as a custom action
// handler module + method name
onApprovePress: function (oBindingContext, aSelectedContexts) {
    if (aSelectedContexts.length === 0) {
        return;
    }
    aSelectedContexts.forEach(function (oContext) {
        var oOperation = oContext.getModel().bindContext(
            "com.sap.gateway.srvd_a2x.zui_product_o4.v0001.approve(...)",
            oContext
        );
        oOperation.invoke().then(function () {
            // refresh
        });
    });
}
```

---

## 4. XML Views Common Patterns

### Correct namespace declarations
```xml
<mvc:View
    xmlns:mvc="sap.ui.core.mvc"
    xmlns="sap.m"
    xmlns:f="sap.f"
    xmlns:core="sap.ui.core"
    xmlns:l="sap.ui.layout"
    xmlns:fb="sap.ui.comp.filterbar"
    controllerName="my.app.controller.Main">
```

Only declare namespaces you actually use in the view.

### Binding patterns
```xml
<!-- Property binding -->
<Text text="{ProductName}"/>

<!-- Expression binding -->
<Text text="{= ${Status} === 'A' ? ${i18n>active} : ${i18n>inactive}}"/>

<!-- Aggregation binding (list) -->
<List items="{path: '/Products', sorter: {path: 'ProductName'}}">
    <StandardListItem title="{ProductName}" description="{ProductID}"/>
</List>

<!-- Binding with type -->
<Text text="{path: 'Price', type: 'sap.ui.model.type.Currency',
            formatOptions: {showMeasure: false}}"/>
```

### Anti-patterns in XML views

**Wrong**: Hardcoded text
```xml
<Button text="Save" press=".onSave"/>
<!-- CORRECT -->
<Button text="{i18n>save}" press=".onSave"/>
```

**Wrong**: Using deprecated controls
```xml
<!-- WRONG -->
<sap.ui.commons:TextField value="{Name}"/>
<!-- CORRECT -->
<Input value="{Name}"/>
```

---

## 5. OData V4 Binding Patterns

### List binding with parameters
```javascript
var oList = this.byId("myTable");
oList.bindItems({
    path: "/Products",
    parameters: {
        $expand: "Category",
        $select: "ProductID,ProductName,Price,Category/Name",
        $orderby: "ProductName asc",
        $count: true
    },
    template: new sap.m.ColumnListItem({ ... })
});
```

### Context binding (Object Page)
```javascript
this.getView().bindElement({
    path: "/Products('001')",
    parameters: {
        $expand: "Items($expand=Material)"
    }
});
```

### Invoking a bound action (V4)
```javascript
var oContext = oTable.getSelectedItem().getBindingContext();
var oActionBinding = oContext.getModel().bindContext(
    "com.namespace.ActionName(...)",
    oContext
);
oActionBinding.invoke().then(
    function () { /* success */ },
    function (oError) { /* error */ }
);
```

### Requesting side effects programmatically
```javascript
// After an action, refresh specific properties or navigation
this.getExtensionAPI().getEditFlow().getView()
    .getBindingContext()
    .requestSideEffects([
        { $NavigationPropertyPath: "Items" },
        { $PropertyPath: "TotalAmount" }
    ]);
```

---

## 6. manifest.json Patterns

### Fiori Elements List Report + Object Page routing

```json
{
  "sap.ui5": {
    "routing": {
      "routes": [
        {
          "pattern": ":?query:",
          "name": "ProductsList",
          "target": "ProductsList"
        },
        {
          "pattern": "Products({key}):?query:",
          "name": "ProductsObjectPage",
          "target": "ProductsObjectPage"
        }
      ],
      "targets": {
        "ProductsList": {
          "type": "Component",
          "id": "ProductsList",
          "name": "sap.fe.templates.ListReport",
          "options": {
            "settings": {
              "contextPath": "/Products",
              "variantManagement": "Page",
              "controlConfiguration": {
                "@com.sap.vocabularies.UI.v1.LineItem": {
                  "tableSettings": {
                    "type": "ResponsiveTable",
                    "enableExport": true,
                    "selectionMode": "Multi"
                  }
                }
              }
            }
          }
        },
        "ProductsObjectPage": {
          "type": "Component",
          "id": "ProductsObjectPage",
          "name": "sap.fe.templates.ObjectPage",
          "options": {
            "settings": {
              "contextPath": "/Products",
              "editableHeaderContent": false
            }
          }
        }
      }
    }
  }
}
```

### Custom section declaration in manifest.json

```json
"content": {
  "body": {
    "sections": {
      "customRevenueSection": {
        "template": "my.app.ext.fragment.RevenueSection",
        "title": "{i18n>revenueSection}",
        "type": "XMLFragment",
        "position": {
          "placement": "After",
          "anchor": "GeneralInfo"
        }
      }
    }
  }
}
```

### Custom column in a table

```json
"controlConfiguration": {
  "@com.sap.vocabularies.UI.v1.LineItem": {
    "columns": {
      "customStatus": {
        "header": "{i18n>statusColumn}",
        "template": "my.app.ext.fragment.StatusColumn",
        "position": {
          "placement": "After",
          "anchor": "DataField::Status"
        },
        "availability": "Default"
      }
    }
  }
}
```

---

## 7. i18n Conventions

### SAP recommended key prefixes

| Prefix | Usage               | Example                        |
|--------|---------------------|--------------------------------|
| `xtit` | Title               | `xtit.appTitle=My Application` |
| `xfld` | Field label         | `xfld.productName=Product Name`|
| `xbut` | Button text         | `xbut.save=Save`               |
| `xmsg` | General message     | `xmsg.saved=Data saved`        |
| `ymsg` | Message with params | `ymsg.deleted={0} deleted`     |
| `xtol` | Tooltip             | `xtol.helpIcon=Show help`      |
| `xcel` | Table column header | `xcel.quantity=Qty`            |
| `xgrp` | Group header        | `xgrp.address=Address`         |
| `notr` | Not translatable    | `notr.dateFormat=yyyy-MM-dd`   |

### Anti-patterns
- `btn1=Save` — non-descriptive key
- Duplicate keys with different values in the same file
- Missing `# XTIT:` comment headers (translation hint for translators)

---

## 8. Legacy API Migration Table

| Legacy (avoid)                          | Modern replacement                                  |
|-----------------------------------------|-----------------------------------------------------|
| `jQuery.sap.log.*`                      | `sap/base/Log`                                      |
| `jQuery.sap.require`                    | `sap.ui.require` / `sap.ui.define`                  |
| `jQuery.sap.declare`                    | `sap.ui.define`                                     |
| `jQuery.sap.delayedCall`               | `setTimeout`                                        |
| `jQuery.sap.clearDelayedCall`          | `clearTimeout`                                      |
| `jQuery.sap.getModulePath`             | `sap.ui.require.toUrl`                              |
| `sap.ui.getCore().byId()`             | `this.byId()` or `Element.getElementById()`         |
| `sap.ui.getCore().getModel()`         | `this.getOwnerComponent().getModel()`               |
| `sap.ui.controller()`                 | `sap.ui.define` with Controller.extend              |
| `sap.ui.xmlview()`                    | `sap.ui.core.mvc.XMLView.create()`                  |
| `sap.ui.xmlfragment()`               | `Fragment.load()` (async)                            |
| `new sap.m.Button()`                  | Import via `sap.ui.define` then `new Button()`      |
| `oModel.read()` (V2 in V4 context)   | `oListBinding.requestContexts()`                    |
| `oModel.create()` (V2)               | `oListBinding.create()`                             |
| `oModel.update()` (V2)               | Property binding + `oContext.setProperty()`         |
| `console.log`                          | `Log.info` / `Log.warning` / `Log.error`            |
