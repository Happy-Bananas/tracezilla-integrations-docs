---
title: Validate Connection
layout: default
parent: tracezilla Fundamentals
nav_order: 30
---

# Validate the tracezilla connection

Validate the base URL, team slug, and token independently before combining
tracezilla with Shopify, WooCommerce, or another service.

## Validation contract

A platform implementation should:

1. Reject a missing or malformed base URL, team slug, token, or timeout before
   making a request.
2. Construct the intended team API base path.
3. Send the Bearer token only to the configured tracezilla origin.
4. Make one small authenticated read permitted by the token.
5. Report success without displaying the token or unnecessary business data.
6. Convert authentication, authorization, timeout, and invalid-response
   failures into safe actionable messages.

When no dedicated authenticated metadata endpoint is available for the chosen
API contract, use a bounded list read such as one SKU and discard its business
payload. A successful response confirms authentication and access to that
resource; it does not prove permission for every later workflow.

## Validate workflow-specific access

Before a workflow runs, perform the smallest relevant read:

| Workflow prerequisite | Read-only validation |
|:--|:--|
| Catalog | Read a bounded SKU page |
| Inventory | Resolve the intended warehouse and read bounded inventory |
| Order import | Resolve the intended partner/location and inspect existing external references |

Write permission must be tested through a controlled workflow preview and
explicitly confirmed execution—not by creating disposable records during a
generic connection test.

## Safe success result

Return only evidence useful to the consultant:

- Connection succeeded.
- The configured team slug or another non-secret team identifier.
- The resource whose read permission was validated.
- Optional timing or request ID when tracezilla supplies it.

Do not return the token, Authorization header, complete configuration, or the
discarded business record.

## Ready to continue

- The selected team is the intended test team.
- A minimal authenticated read succeeds.
- The token can read every prerequisite resource for the selected workflow.
- Logs and displayed results contain no credentials.

## Implementations

{: .label .label-yellow }
Planned

Platform-specific commands will be linked after their focused repositories are
verified.
