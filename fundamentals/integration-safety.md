---
title: Integration Safety
layout: default
parent: tracezilla Fundamentals
nav_order: 100
---

# Integration safety

Start with read-only calls or dry runs. Treat identifiers, units, locations,
currencies, tax behavior, and source-of-truth rules as customer decisions.

Working write examples must provide a preview, a conservative limit, explicit
execution, duplicate protection where applicable, and structured results.

## Production boundary

These examples demonstrate API access, mapping points, previews, and controlled
writes. The adopting system owns scheduling, concurrency, monitoring, alerting,
recovery, credential rotation, retention, and deployment policy.

Before a production write, document:

- Source of truth and every customer-approved mapping.
- Selection window and stable ordering.
- Idempotency or duplicate-detection strategy.
- Behavior for partial and uncertain failures.
- Rate-limit and retry policy supported by the current API contract.
- Reconciliation and rollback procedure.
- Who receives an alert and what evidence is safe to share.
