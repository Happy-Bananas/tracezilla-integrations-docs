---
title: Standard Test Dataset
layout: default
parent: tracezilla Prerequisites
nav_order: 35
---

# Standard test dataset

The standard dataset gives developers a shared, non-production starting point.
It is intentionally a little broader than any single feature so catalog,
inventory, and order examples can use the same recognizable records.

The dataset is described in a YAML seed file and applied through the tracezilla
API. The first seed file ships with the WooCommerce PHP integration:
[`seeds/tracezilla.yaml`](https://github.com/Happy-Bananas/tracezilla-woocommerce-php/blob/main/seeds/tracezilla.yaml).
See [Seed the Test Dataset](./seed.html) for how to run it.

## Naming rule

Every record owned by the dataset starts with `TZ-`, the same prefix used by
the external-service sandboxes, so that SKUs match across both systems. Do not
use customer names, addresses, product codes, or other production information.

## Version 1 catalog

SKUs follow the pattern `TZ-<CATEGORY>-<NNN>`, and variations add `-<NN>`.
tracezilla allows at most 17 characters in an SKU code. The sandboxes contain
200 food products in ten categories: `FRUIT`, `VEG`, `DAIRY`, `BAKERY`, `MEAT`,
`FISH`, `COFFEE`, `TEA`, `PANTRY`, and `SNACK`. tracezilla receives a
subset:

| SKU | Product | Intended state |
|:--|:--|:--|
| `TZ-FRUIT-001` | Fairtrade Bananas 1 kg | Matched, with stock |
| `TZ-FRUIT-002` | Golden Pineapple | Matched, with stock, on sale in the shop |
| `TZ-FRUIT-003` | Ripe Mango | Matched, zero inventory |
| `TZ-COFFEE-001-01`, `TZ-COFFEE-001-02` | House Blend Coffee Beans 250 g and 1 kg | Matched variations, with stock |
| `TZ-DAIRY-001` | Aged Gouda 500 g | Matched, with stock |
| `TZ-DAIRY-002` | Organic Whole Milk 1 L | Matched, stored in crates of 12 (unit conversion) |
| `TZ-RAW-001` | Raw Cocoa Beans 25 kg | Raw material that exists only in tracezilla |

The rest of the sandbox catalog, such as `TZ-FRUIT-004`, is deliberately
missing from tracezilla so the create-SKUs workflow has records to create.

## Shared supporting data

| Record | Purpose |
|:--|:--|
| `TZ-SUPPLIER` partner | Supplier of the stock purchase order |
| `TZ-WEBSHOP` partner | Customer partner for imported webshop orders |
| `TZ-WAREHOUSE-MAIN` location on your own company | Warehouse that holds the stock |
| `TZ-STOCK-001` purchase order | Received order that puts stock in the warehouse |

Stock quantities differ from the sandbox on purpose, so inventory
synchronization has updates to report.

## Feature coverage

| Feature | Dataset records |
|:--|:--|
| Connection check | No business record is required |
| Catalog comparison | Matched and tracezilla-only SKUs |
| Create missing tracezilla SKUs | Sandbox SKUs missing from tracezilla |
| Inventory synchronization | Positive, zero, and unit-conversion cases |
| Order import | Webshop partner, warehouse, and matched SKUs |

## Seed file format

The seed file keeps tracezilla API field names, so any field from the
[tracezilla API documentation](https://app.tracezilla.com/api/documentation)
can be added to a record. Records are identified by `ref` (partners and
locations), `sku_code` (SKUs), and `ext_ref` (purchase orders). The seed only
creates records that are missing; it never updates or deletes existing ones.

Official references:

- [Import and export tutorials](https://www.tracezilla.com/en/tutorials/topic/import-and-export-of-data)
- [Import SKUs](https://www.tracezilla.com/en/tutorials/import-skus)
- [Import partners](https://www.tracezilla.com/en/tutorials/import-template-for-partners)
- [Import lots](https://www.tracezilla.com/en/tutorials/import-lots)
