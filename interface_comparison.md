# SAP API Interface Comparison: Similar Fields

## Overview
This document compares similar fields between two interfaces:
1. **Warehouse Order and Task (A2X)** - Transaction data
2. **Warehouse Available Stock - Read (A2X)** - Master/Stock data

---

## Field-by-Field Comparison

| Field Name | Interface 1: Warehouse Order & Task | Interface 2: Warehouse Available Stock | Data Type | Length | Match Status |
|---|---|---|---|---|---|
| **Warehouse** | EWMWarehouse (Entity) | EWMWarehouse (Entity) | CHAR | 4 | ✅ IDENTICAL |
| **Product/Material** | Product (CHAR, 40) | Product (Entity) | CHAR | 40 | ✅ MATCH (same entity name) |
| **Batch** | Batch (CHAR, 10) | Batch (Entity) | CHAR | 10 | ✅ MATCH (same entity name) |
| **Storage Type** | SourceStorageType (CHAR, 4) | EWMStorageType (Entity) | CHAR | 4 | ✅ RELATED (different contexts) |
| **Storage Type** | DestinationStorageType (CHAR, 4) | EWMStorageType (Entity) | CHAR | 4 | ✅ RELATED (different contexts) |
| **Storage Bin** | SourceStorageBin (CHAR, 18) | EWMStorageBin (Entity) | CHAR | 18 | ✅ RELATED (different contexts) |
| **Storage Bin** | DestinationStorageBin (CHAR, 18) | EWMStorageBin (Entity) | CHAR | 18 | ✅ RELATED (different contexts) |
| **Unit of Measure** | BaseUnit (UNIT, 3) | StockKeepingAltvUnitISOCode (ISO Code) | UNIT | 3 | ✅ RELATED |
| **Unit of Measure** | AlternativeUnit (UNIT, 3) | StockKeepingAltvUnitISOCode (ISO Code) | UNIT | 3 | ✅ RELATED |
| **Quantity** | TargetQuantityInBaseUnit (QUAN, 31) | AvailableEWMStockQty (Entity) | QUAN | 31 | ✅ RELATED (different semantics) |
| **Quantity** | TargetQuantityInAltvUnit (QUAN, 31) | AvailableEWMStockQty (Entity) | QUAN | 31 | ✅ RELATED (different semantics) |
| **Stock Type** | *(Not present)* | EWMStockType (Entity) | N/A | N/A | ❌ GAP in Interface 1 |
| **Handling Unit** | SourceHandlingUnit (CHAR, 20) | HandlingUnitExternalID (Entity) | CHAR | 20 | ✅ RELATED |
| **Handling Unit** | DestinationHandlingUnit (CHAR, 20) | HandlingUnitExternalID (Entity) | CHAR | 20 | ✅ RELATED |

---

## Characteristics Comparison

### Material Characteristics (Product-level)

| Field Name | Interface 1 | Interface 2 | Source | Notes |
|---|---|---|---|---|
| **TOB_TYPE** | TOB_TYPE (Value, CHAR, 3) | TOB_TYPE | Material Characteristics Value API | ✅ IDENTICAL mapping |
| **TOB_TYPE_DES** | *(Not in mapping)* | TOB_TYPE_DESCRIPTION | Material Characteristics Value API | ✅ Description available in Interface 2 |
| **CIG_CODE** | CIG_CODE (Value, CHAR, 3) | *(Not present)* | Material Characteristics Value API | ❌ CIG_CODE only in Interface 1 |

### Batch Characteristics (Batch-level)

| Field Name | Interface 1 | Interface 2 | Source | Notes |
|---|---|---|---|---|
| **GRADE** | GRADE (Value, CHAR, 3) | GRADE | Batch Master Record - Classification | ✅ IDENTICAL mapping |
| **GRADE_DES** | GRADE_DESCRIPTION (CHAR, 30) | GRADE_DESCRIPTION | Batch Master Record - Classification | ✅ IDENTICAL mapping |
| **SIZE** | SIZE (Value, CHAR, 3) | SIZE | Batch Master Record - Classification | ✅ IDENTICAL mapping |
| **SIZE_DES** | SIZE_DESCRIPTION (CHAR, 30) | SIZE_DESCRIPTION | Batch Master Record - Classification | ✅ IDENTICAL mapping |
| **LEAVES** | LEAVES (Value, CHAR, 30) | LEAVES | Batch Master Record - Classification | ✅ IDENTICAL mapping |
| **CQT** | *(Not in first table)* | CQT | Batch Master Record - Classification | ❌ Not in Interface 1 first table |

---

## Key Findings

### ✅ Perfect Matches (Identical Fields & Definitions)
1. **EWMWarehouse** - Warehouse identifier
2. **Product** - Product/Material master
3. **Batch** - Batch identifier
4. **GRADE** / **GRADE_DESCRIPTION** - Grade characteristic
5. **SIZE** / **SIZE_DESCRIPTION** - Size characteristic
6. **LEAVES** - Leaves characteristic
7. **TOB_TYPE** - Tobacco type characteristic

### ⚠️ Related but Different Context
1. **Storage Type** - Interface 1 has Source/Destination separate; Interface 2 has single entity
2. **Storage Bin** - Interface 1 has Source/Destination separate; Interface 2 has single entity
3. **Quantity** - Interface 1 has Target quantities; Interface 2 has Available stock quantities
4. **Unit of Measure** - Interface 1 has Base/Alternative; Interface 2 has ISO codes
5. **Handling Unit** - Interface 1 has Source/Destination; Interface 2 has External ID entity

### ❌ Gaps & Mismatches

| Field | Present In | Missing From | Risk Level |
|---|---|---|---|
| **CIG_CODE** | Interface 1 (Material Char) | Interface 2 | 🔴 HIGH - Product attribute not exposed in stock API |
| **CQT** | Interface 2 (Batch Char) | Interface 1 | 🟡 MEDIUM - Quality attribute in stock not in order/task |
| **EWMStockType** | Interface 2 | Interface 1 | 🟡 MEDIUM - Stock classification missing in order context |
| **EntitledToDisposeParty** | Interface 2 | Interface 1 | 🟡 MEDIUM - Ownership info in stock not in order |
| **EWMStockUsage** | Interface 2 | Interface 1 | 🟡 MEDIUM - Stock usage classification missing in order |

---

## Recommendations

1. **Standardize Storage Location Handling** - Consolidate Source/Destination storage types and bins into a single comprehensive model that supports both transactional and inventory contexts.

2. **Expose CIG_CODE in Stock API** - If Cigar Code is a critical product discriminator, it should be available in the Warehouse Available Stock interface for filtering and reporting.

3. **Align Quantity Semantics** - Document clearly when Target quantities (Order) map to Available quantities (Stock) for reconciliation logic.

4. **Add Stock Classification to Order Context** - Consider adding EWMStockType and EWMStockUsage to the Warehouse Order interface for better traceability.

5. **Create Unified Master Data View** - Establish a product master data entity that consolidates all characteristics (TOB_TYPE, CIG_CODE, GRADE, SIZE, LEAVES, CQT) for consistent reference.

6. **Document Disposal Party Logic** - Define how EntitledToDisposeParty in stock maps to order fulfillment responsibilities.

---

## Data Flow Implications

```
Warehouse Order (Source)
├── Material Characteristics (TOB_TYPE, CIG_CODE) ──→ Product (Interface 2)
├── Batch Characteristics (GRADE, SIZE, LEAVES)  ──→ Batch (Interface 2)
├── Quantity (Target)                            ──→ Quantity (Available) [CONVERSION NEEDED]
├── Source/Dest Storage                          ──→ Single Storage Entity [CONSOLIDATION]
└── Handling Units                               ──→ Handling Unit Entity [MATCH]

Available Stock (Inventory)
├── Stock Type Classification                    ──X→ Order (Interface 1) [GAP]
├── Disposal Party                               ──X→ Order (Interface 1) [GAP]
└── Stock Usage Classification                   ──X→ Order (Interface 1) [GAP]
```

