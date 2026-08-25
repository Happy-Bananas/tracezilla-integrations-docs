---
layout: default
nav_order: 40
parent: Shopify Prerequisites
title: Inventory Setup
---

# Inventory setup

This optional guide prepares a Shopify development store for inventory
synchronization with tracezilla.

The goal is a small and predictable test setup:

```text
tracezilla inventory
        ↓
matched by SKU
        ↓
Shopify inventory item
        ↓
selected Shopify location
```

During the current example workflow, tracezilla is the source of truth for the
available quantity. Use test products and locations—not production inventory.

## Before you begin

Make sure you already have:

- A Shopify development store.
- At least one active Shopify location.
- A few test products with unique SKUs.
- Matching test SKUs available in tracezilla.

## Step 1: Choose a development location

1. Open the dev store's Shopify Admin.
2. Select **Settings → Locations**.
3. Open the location that should receive synchronized inventory.
4. Confirm that it is active.
5. Record its name so you can recognize it in API or command output later.

Shopify can store the same inventory item at several locations. The inventory
item ID identifies **what** should be updated; the location ID identifies
**where** it should be updated.

```text
Inventory item ID + Location ID = one inventory level
```

Do not copy a location ID from a production or different store.

## Step 2: Enable inventory tracking

Repeat for every test variant:

1. Open the product and select the variant.
2. Find the **Inventory** section.
3. Enable **Track quantity** or the equivalent inventory-tracking control.
4. Confirm that the variant has a unique SKU.
5. Save the product.

A location assignment and SKU do not by themselves mean Shopify tracks the
quantity.

## Step 3: Stock the variant at the location

1. Open the variant's inventory settings.
2. Find the locations that stock it.
3. Make sure the selected development location stocks the variant.
4. Enter a small starting quantity, such as `3`.
5. Save the change.

Use a Shopify quantity that differs from tracezilla so the later preview is
easy to verify:

```text
Shopify available quantity:    3
tracezilla available quantity: 10
Expected preview:              would update 3 → 10
```

## Step 4: Decide out-of-stock behavior

For a straightforward test, leave **Continue selling when out of stock**
disabled. This option controls selling behavior; it does not prevent an API
client from reading or updating the quantity.

## Step 5: Grant inventory permissions

The inventory workflow uses:

| Scope | Purpose |
|:--|:--|
| `read_products` | Read variants and SKUs |
| `read_locations` | Read Shopify locations |
| `read_inventory` | Read inventory items and quantities |
| `write_inventory` | Update quantities after explicit execution |

Having `write_inventory` permission does not cause a write by itself. A safe
implementation remains a preview until execution is explicitly enabled.

## Readiness checklist

- The chosen location is active.
- Every selected variant has inventory tracking enabled.
- Every selected variant has a unique SKU.
- Each variant is stocked at the chosen location.
- Shopify and tracezilla contain deliberately chosen test quantities.
- The app will request the four inventory scopes above.

Continue with [Authorize API](./authorize-api.html).
