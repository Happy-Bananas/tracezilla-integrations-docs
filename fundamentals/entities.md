---
title: Catalog Prerequisites
layout: default
parent: tracezilla Prerequisites
nav_order: 40
---

# Catalog prerequisites

The integration needs a small, known catalog; it does not need a realistic
customer catalog. Learn how tracezilla SKUs work from the official
[SKU tutorial](https://www.tracezilla.com/en/tutorials/stock-keeping-units-sku)
and use the official
[SKU import guide](https://www.tracezilla.com/en/tutorials/import-skus) when
preparing several records.

For integration testing, provide:

- A few active SKUs with stable `TZ-` codes.
- At least one SKU that will also exist in the external service.
- At least one tracezilla-only SKU for difference reporting.
- Units and weight conversions that are easy to verify by hand.
- No real customer product names, prices, or identifiers.

An external service's product model does not automatically map to a
tracezilla SKU. You must define identity, product structure, units, weights,
and conversions explicitly.

## Identity

The examples match an external variant SKU to tracezilla's `sku_code`. Change
this if your customer identifies products differently.

Decide and test:

- Whether comparison is exact or normalized.
- How whitespace and character case are treated.
- What happens when a source SKU is blank.
- What happens when either system contains duplicate SKU codes.
- Whether existing tracezilla SKUs may be updated or only reported.

Never silently choose one duplicate record as the winner.

## Mapping decisions

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

Keep a catalog comparison read-only. When you write a command that creates
SKUs, make it:

1. Read the complete source and the existing tracezilla SKUs.
2. Classify missing, existing, blank, and duplicate records.
3. Validate the mapped data.
4. Preview the decision without writing.
5. Write only with `--execute --confirm`, and start with a small limit.
6. Report one result per selected source record.

Consult the current [tracezilla API documentation](https://app.tracezilla.com/api/documentation)
for supported SKU fields and endpoint schemas.
