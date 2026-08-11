---
title: SKUs and Products
layout: default
parent: tracezilla Fundamentals
nav_order: 40
---

# SKUs and products

An external commerce platform's product model does not automatically map to a
tracezilla SKU. The integration must define identity, product structure,
units, weights, and conversions explicitly.

## Identity

The initial examples match an external variant SKU to tracezilla's `sku_code`.
That is a workflow policy, not a universal rule.

Decide and test:

- Whether comparison is exact or normalized.
- How whitespace and character case are treated.
- What happens when a source SKU is blank.
- What happens when either system contains duplicate SKU codes.
- Whether existing tracezilla SKUs may be updated or only reported.

Never silently choose one duplicate record as the winner.

## Product structure and units

One external product can contain several variants, while tracezilla represents
SKU data together with operational units and conversion values. Creating a
tracezilla SKU can therefore require decisions beyond copying a code and name:

- Global/display name.
- Unit of measure.
- Lot unit.
- Default unit conversion.
- Net and gross weight factors.
- Categories, tags, or other customer conventions.

Demonstration defaults make an example executable; they are not business
advice. Keep them visible in a mapper and replace them with customer-approved
rules.

## Read versus write behavior

A catalog comparison should remain read-only. A create workflow should:

1. Read the complete source and existing tracezilla identities.
2. Classify missing, existing, blank, and duplicate records.
3. Validate the mapped payload.
4. Preview the decision without writing.
5. Require explicit execution and a conservative initial limit.
6. Return one result per selected source record.

Consult the current [tracezilla API documentation](https://app.tracezilla.com/api/documentation)
for supported SKU fields and endpoint schemas.

This page will introduce SKUs, products, locations, inventory, sales orders,
pagination, and the customer-specific mapping decisions shared by integrations.
