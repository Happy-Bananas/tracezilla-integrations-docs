---
title: Data Mapping
layout: default
parent: tracezilla Fundamentals
nav_order: 80
---

# Data mapping

API fields describe available data; they do not decide what a customer's data
means. Record every mapping policy before enabling writes.

| Concept | External service | tracezilla | Required decision |
|:--|:--|:--|:--|
| SKU identity | Product or variant SKU | `sku_code` | Exact comparison, normalization, blanks, duplicates |
| Product structure | Products and variants | SKU and units | Entity boundary, names, units, weights, conversions |
| Location | Platform-specific location ID | Warehouse/location identity | Explicit source-to-destination relationship |
| Inventory | Quantity at one external location | Available inventory and unit conversion | Source of truth and safe quantity conversion |
| Order identity | Stable platform order ID | External reference | Namespace, uniqueness, duplicate and update policy |
| Customer | Customer/address or webshop account | Partner and location | Resolution and fallback rules |
| Order line | SKU, quantity, price, currency | Outbound SKU line | Discounts, taxes, shipping, refunds, price-per logic |

Identifiers from different namespaces are not interchangeable. Label database
IDs, location numbers, display names, external IDs, and references explicitly.

## Keep mapping separate from transport

Authentication and HTTP clients should not decide business mappings. A
workflow should convert external response data into typed source data, apply a
visible mapper, validate a tracezilla-shaped payload, and pass the result to a
destination service.

This separation lets consultants change customer rules without rewriting
authentication, pagination, retries, or error handling.
