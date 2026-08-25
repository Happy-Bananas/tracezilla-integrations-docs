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

This first iteration defines the data contract only. Import spreadsheets will
be added after they have been generated from and validated against current
tracezilla templates.

## Naming rule

Every record owned by the dataset starts with `TZINT-`. Do not use customer
names, addresses, product codes, or other production information.

## Version 1 catalog

| SKU | Intended state |
|:--|:--|
| `TZINT-MATCHED-001` | Exists in tracezilla and the commerce test store |
| `TZINT-MATCHED-002` | A second normal matched product |
| `TZINT-TRACEZILLA-ONLY-001` | Exists only in tracezilla |
| `TZINT-STOCK-001` | Has a small positive inventory quantity |
| `TZINT-ZERO-STOCK-001` | Has zero available inventory |
| `TZINT-FRACTIONAL-001` | Uses a unit conversion that exposes unsafe rounding |

The commerce-platform dataset will later add its platform-only and invalid
records. Those do not belong in tracezilla by definition.

## Shared supporting data

- One `TZINT` test warehouse/location.
- One `TZINT Webshop` customer partner.
- One `TZINT Supplier` partner.
- A valid owner and required partner locations.
- Small stock quantities for the inventory records.
- Only the units and configuration required by these records.

## Feature coverage

| Feature | Dataset records |
|:--|:--|
| Connection check | No business record is required |
| Catalog comparison | Matched and tracezilla-only SKUs |
| Create missing tracezilla SKUs | The later commerce-only SKU |
| Inventory synchronization | Positive, zero, and fractional cases |
| Order import | Webshop partner, warehouse, owner, and matched SKUs |

## Before import files are published

tracezilla import templates are downloaded from the application and can
contain account-specific structure or identifiers. Version 1 files must be
created from a current empty test team, contain no secrets or production data,
and be tested for both first import and repeated import behavior.

Official references:

- [Import and export tutorials](https://www.tracezilla.com/en/tutorials/topic/import-and-export-of-data)
- [Import SKUs](https://www.tracezilla.com/en/tutorials/import-skus)
- [Import partners](https://www.tracezilla.com/en/tutorials/import-template-for-partners)
- [Import lots](https://www.tracezilla.com/en/tutorials/import-lots)
