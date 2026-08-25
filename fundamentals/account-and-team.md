---
title: Safe Test Team
layout: default
parent: tracezilla Fundamentals
nav_order: 10
---

# Prepare a safe tracezilla test team

Use a demo or otherwise isolated team while learning, testing mappings, and
validating write operations. This keeps test partners, SKUs, orders, and
inventory separate from customer production records.

Use the official [tracezilla tutorials](https://www.tracezilla.com/en/tutorials)
to create and configure the account. This page only defines the integration
safety requirements.

Before development, confirm that:

- The selected team contains no customer production data.
- The developer is allowed to create and remove test records.
- Company settings are sufficient for the feature being tested.
- Everyone can recognize the team as a test environment.
- Any demo expiration or access-review date is recorded.

If an integration project needs a demo extension or account assistance,
[contact tracezilla](https://www.tracezilla.com/en/contact-us).

## Identify the team slug

Open a page inside the intended team and identify the team-specific slug in
the browser URL. The API client uses this slug as part of its base path:

```text
https://app.tracezilla.com/api/v1/{team-slug}
```

The slug selects a team; it is not a credential. Do not assume that it is
identical to the displayed company name.

## Choose the test data

Different workflows have different prerequisites:

| Workflow | Typical tracezilla test data |
|:--|:--|
| Catalog comparison | A small set of known SKUs |
| Create missing SKUs | A team where test SKUs may safely be created |
| Inventory synchronization | A warehouse location and received inventory |
| Order import | A webshop/customer partner, locations, owner, and matching SKUs |

For the quickest shared setup, use the
[standard test dataset](./test-data.html). A developer working on one narrow
feature may instead prepare only that feature's prerequisites.

## Readiness checklist

- You are signed in to the intended non-production team.
- You know the exact team slug.
- Team and company details are sufficient for the selected workflow.
- The team contains only deliberate test data.

Continue with [Authentication and Secrets](./authentication.html).
