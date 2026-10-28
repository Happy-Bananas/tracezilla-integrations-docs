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
[`seeds/woocommerce-sandbox.yaml`](https://github.com/Happy-Bananas/tracezilla-woocommerce-php/blob/main/seeds/woocommerce-sandbox.yaml).
See [Seed tracezilla](../woocommerce/setup.html#seed-tracezilla) for how to run
it.

## Naming rule

Every record owned by the dataset starts with `TZ-`, the same prefix used by
the commerce-platform sandboxes, so that SKUs match across both systems. Do not
use customer names, addresses, product codes, or other production information.

## Version 1 catalog

| SKU | Intended state |
|:--|:--|
| `TZ-SIMPLE-001`, `TZ-SALE-001` | Matched, with stock |
| `TZ-TSHIRT-S`, `TZ-TSHIRT-M` | Matched variations, with stock |
| `TZ-CATALOG-001` | Matched, with stock |
| `TZ-CATALOG-002` | Matched, stored in boxes of 12 (unit conversion) |
| `TZ-OUT-OF-STOCK` | Matched, zero inventory |
| `TZ-TRACEZILLA-ONLY-001` | Exists only in tracezilla |

The rest of the commerce catalog, such as `TZ-CATALOG-003`, is deliberately
missing from tracezilla so the create-SKUs workflow has records to create.

## Shared supporting data

| Record | Purpose |
|:--|:--|
| `TZ-SUPPLIER` partner | Supplier of the stock purchase order |
| `TZ-WEBSHOP` partner | Customer partner for imported webshop orders |
| `TZ-WAREHOUSE` partner, `TZ-WAREHOUSE-MAIN` location | Warehouse that holds the stock |
| `TZ-STOCK-001` purchase order | Received order that puts stock in the warehouse |

Stock quantities differ from the commerce sandbox on purpose, so inventory
synchronization has updates to report.

## Feature coverage

| Feature | Dataset records |
|:--|:--|
| Connection check | No business record is required |
| Catalog comparison | Matched and tracezilla-only SKUs |
| Create missing tracezilla SKUs | Commerce SKUs missing from tracezilla |
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
