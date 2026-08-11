---
title: Locations and Inventory
layout: default
parent: tracezilla Fundamentals
nav_order: 50
---

# Locations and inventory

Inventory synchronization needs an explicit relationship between the
tracezilla warehouse and the external service's location. Display names,
database IDs, location numbers, and external GraphQL IDs are different
identifier namespaces.

## Select the tracezilla source location

Create or select a non-production location that can hold warehouse inventory.
Record its tracezilla location number and verify it resolves to the intended
internal location through the API.

Do not assume that the first supplier or company location is the inventory
source. Confirm where received lots actually appear.

## Available inventory requires received stock

Creating an SKU does not necessarily create available inventory. A realistic
test may require a purchase or receiving flow so stock exists at the selected
warehouse. The inventory workflow page will document the exact test procedure
and consequences.

Use a small known dataset:

| SKU | tracezilla quantity | External quantity | Expected result |
|:--|--:|--:|:--|
| `TEST-COFFEE-250G` | 10 | 3 | Would update |
| `TEST-COFFEE-1KG` | 5 | 5 | Unchanged |
| `TEST-TEA-GREEN` | 7 | Missing | Missing externally |

## Mapping decisions

- Which tracezilla warehouse maps to which external location.
- Which system is the source of truth.
- Which tracezilla quantity represents externally available stock.
- How tracezilla unit conversion becomes an external quantity.
- Whether fractional, negative, reserved, quarantined, or unavailable stock is
  excluded or transformed.
- Whether zero means out of stock and how overselling is configured.

An external platform may require a non-negative whole number. Reject unsafe
fractional mappings rather than rounding silently.

## Safe synchronization

Read and compare both locations before writing. Preview old and new quantities,
require explicit execution, limit the first write, and update only the selected
location. A failure for one SKU must not hide results for the others.
