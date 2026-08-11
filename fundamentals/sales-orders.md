---
title: Partners and Sales Orders
layout: default
parent: tracezilla Fundamentals
nav_order: 60
---

# Partners and sales orders

Importing an external order requires more context than its lines. The
integration must resolve customer, pickup, delivery, ownership, warehouse, SKU,
price, currency, and identity rules before creating a tracezilla sales order.

## Webshop partner pattern

An integration can represent the webshop as a tracezilla customer partner. A
workflow may use that partner's primary location and owner when creating sales
orders. This is one useful example pattern—not a requirement for every
customer.

Customer-specific decisions include:

- Partner resolution by stable ID, configured name, or another mapping.
- Primary-location and address fallbacks.
- Required owner.
- Warehouse partner and pickup location.
- Payment term and payment-provider text.

Do not hard-code a demonstration partner name as a universal value.

## Stable order identity

Map the external platform's stable order ID to a tracezilla external reference
with a documented namespace or prefix. Before creating an order, read existing
references and skip a duplicate.

Display order numbers can be useful to humans but may not be the best stable
identity. Keep the external stable ID and display name separately where the
API model permits it.

## Line and money decisions

- SKU matching and behavior for missing SKUs.
- Current versus original quantity.
- Discounts, refunds, shipping, taxes, and rounding.
- Unit price and price-per unit.
- Currency and exchange-rate policy.
- Cancelled, test, fulfilled, and partially fulfilled order selection.
- Orders whose line collection requires additional pagination.

Fail a mapping that the example does not safely understand. Do not silently
drop monetary or quantity information.

## Retry safety

A disconnected response can be uncertain: tracezilla may have accepted the
create before the client timed out. Before retrying, query the stable external
reference. Blind retries can create duplicates unless the endpoint and request
use a documented idempotency mechanism.

Consult the current [tracezilla API documentation](https://app.tracezilla.com/api/documentation)
for sales-order request and response schemas.
