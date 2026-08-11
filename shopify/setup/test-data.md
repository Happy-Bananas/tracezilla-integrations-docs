---
title: Products and Locations
layout: default
parent: Setup
grand_parent: Shopify
nav_order: 30
---

# Prepare products and locations

Create a small, recognizable dataset before testing an integration. Shopify
stores inventory for a variant's inventory item at a particular location:

```text
Product: Coffee
└── Variant: 250 g
    ├── SKU: TEST-COFFEE-250G
    └── Inventory item
        ├── Development Warehouse → Quantity: 25
        └── Test Store            → Quantity: 8
```

## Create or select a location

1. Open the dev store's Shopify Admin.
2. Open **Settings → Locations**.
3. Select an existing test location or create one named, for example,
   `Development Warehouse`.
4. Make sure it is active.
5. Enable fulfilment behavior only when the selected order workflow needs it.

Location identifiers belong to one specific store. Never reuse an identifier
copied from a production store or another dev store.

## Create test products

Create a few products with deliberately simple variants. Give every test
variant a unique SKU, for example:

```text
TEST-COFFEE-250G
TEST-COFFEE-1KG
TEST-TEA-GREEN
```

SKU matching is a consultant-selected integration rule, not a Shopify
guarantee. Empty or duplicate SKUs must be handled explicitly by each workflow.

## Prepare inventory when needed

For an inventory workflow:

1. Enable quantity tracking for each selected variant.
2. Make sure the development location stocks the variant.
3. Enter small, known quantities.
4. Use quantities that make expected comparisons obvious.

Example test plan:

| SKU | Shopify quantity | Expected comparison |
|:--|--:|:--|
| `TEST-COFFEE-250G` | 3 | Different from tracezilla |
| `TEST-COFFEE-1KG` | 5 | Equal to tracezilla |
| `TEST-TEA-GREEN` | Missing | Missing in Shopify |

Do not enable inventory writes merely to validate the initial catalog
connection.

## Readiness checklist

- You are working in a dev store.
- At least one active location exists.
- At least one product variant has a unique test SKU.
- Inventory tracking and location stocking are enabled only where needed.
- The expected test values are recorded before running a comparison.

Continue with [Create and install an app](./app-authorization.html).
