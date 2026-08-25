---
layout: default
nav_order: 30
parent: Shopify Prerequisites
title: Products & Locations
---

# Products and locations

Before testing the integration, prepare a small product catalog and at least
one location in your Shopify development store.

A **location** is a place where inventory is stored, such as a warehouse,
retail store, or distribution centre. Shopify records inventory for each
product variant separately at each location.

## Understand the relationship

A Shopify product describes the item you sell. A **variant** is a specific
version of that product, and the SKU identifies that version.

```text
Product: T-shirt
└── Variant: Blue / Medium
    ├── SKU: TSHIRT-BLUE-M
    └── Inventory item
        ├── Development Warehouse → Quantity: 25
        └── Development Store     → Quantity: 8
```

The same SKU can therefore have different inventory quantities at different
locations.

## Create or select a location

1. Open the [Shopify Admin](https://admin.shopify.com).
2. Select the development store used for this project.
3. Select **Settings**.
4. Select **Locations**.
5. Use an existing test location or select **Add location**.
6. Enter a clear name such as `Development Warehouse`.
7. Enter the requested address information.
8. Make sure the location is active.
9. Enable online-order fulfilment only when the selected workflow requires it.
10. Save the location.

Do not experiment with locations or inventory in a customer's production
store.

## Create test products

1. In Shopify Admin, select **Products**.
2. Select **Add product**.
3. Enter a product title.
4. Add options such as size or colour when the test needs several variants.
5. Enter a unique SKU for every variant.
6. Save the product.

Use recognizable SKUs such as:

```text
TEST-COFFEE-250G
TEST-COFFEE-1KG
TEST-TEA-GREEN
```

Every variant selected for synchronization with tracezilla must have an SKU.

## Assign inventory to the location

For each test product or variant:

1. Open the product.
2. Select the variant when the product has more than one.
3. Find the **Inventory** section.
4. Make sure inventory is stocked at the development location.
5. Enter a small test quantity.
6. Save the product.

Different quantities make later inventory tests easier to understand. For
example, assign `25` units to the warehouse and `8` units to the store.

## Readiness checklist

- You are working in a Shopify development store.
- At least one active location exists.
- You know which location the integration should use.
- At least one product exists.
- Every test variant has a unique SKU.
- Test inventory is assigned to the chosen location.

Continue with [Inventory Setup](./inventory-setup.html) if the selected
workflow synchronizes inventory. Otherwise continue with
[Authorize API](./authorize-api.html).
