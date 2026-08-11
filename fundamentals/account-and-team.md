---
title: Account and Team
layout: default
parent: tracezilla Fundamentals
nav_order: 10
---

# Create or select a safe tracezilla team

Use a demo or otherwise isolated team while learning, testing mappings, and
validating write operations. This keeps test partners, SKUs, orders, and
inventory separate from customer production records.

## Prepare the account

1. Register for or sign in to tracezilla.
2. Select the team that will own the integration test data.
3. Complete the company details required by the selected workflows.
4. Confirm that the account contains no customer production data.
5. Record any expiration or access-review date associated with the test
   environment.

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

## Create only the test data you need

Different workflows have different prerequisites:

| Workflow | Typical tracezilla test data |
|:--|:--|
| Catalog comparison | A small set of known SKUs |
| Create missing SKUs | A team where test SKUs may safely be created |
| Inventory synchronization | A warehouse location and received inventory |
| Order import | A webshop/customer partner, locations, owner, and matching SKUs |

The relevant workflow page owns the detailed setup. Do not populate partners,
orders, or stock globally merely because another example needs them.

## Readiness checklist

- You are signed in to the intended non-production team.
- You know the exact team slug.
- Team and company details are sufficient for the selected workflow.
- The team contains only deliberate test data.

Continue with [Authentication and Secrets](./authentication.html).
