---
title: tracezilla Fundamentals
layout: default
nav_order: 20
has_children: true
---

# tracezilla fundamentals

What an integration developer needs before connecting another service to
tracezilla.

The official [tracezilla tutorials](https://www.tracezilla.com/en/tutorials)
explain how to operate tracezilla. This section does not repeat them. It covers
the safe test environment, API access, data prerequisites, mapping decisions,
and execution rules shared by integrations.

## Recommended path

1. [Prepare a safe test team](./fundamentals/account-and-team.html).
2. [Create and protect API credentials](./fundamentals/authentication.html).
3. Prepare the [standard test dataset](./fundamentals/test-data.html), or only
   the prerequisites needed by the feature:
   - [Catalog prerequisites](./fundamentals/entities.html)
   - [Inventory prerequisites](./fundamentals/locations-and-inventory.html)
   - [Order prerequisites](./fundamentals/sales-orders.html)
4. Review [pagination](./fundamentals/pagination.html),
   [data mapping](./fundamentals/data-mapping.html), and
   [synchronization results](./fundamentals/synchronization-results.html)
   before implementing a complete workflow.

The [tracezilla tutorials](https://www.tracezilla.com/en/tutorials) are the
canonical product guide. The public
[tracezilla API documentation](https://app.tracezilla.com/api/documentation)
is the canonical endpoint reference.
