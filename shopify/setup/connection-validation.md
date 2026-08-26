---
title: Validate Connection
layout: default
parent: Shopify Prerequisites
nav_order: 60
---

# Validate the Shopify connection

Validate a narrow authenticated read before running catalog, inventory, or
order workflows. Connection validation is read-only and should not print or
store the acquired access token.

## Validation contract

A platform implementation should:

1. Validate that the shop hostname, client ID, client secret, requested API
   version, and timeouts are configured.
2. Request an access token using the client credentials grant.
3. Send a small GraphQL Admin API query such as `shop { name }`.
4. Confirm that the request authenticated successfully.
5. Read Shopify's `X-Shopify-API-Version` response header.
6. Report whether Shopify used the version requested by the implementation.
7. Return only safe identifiers and status information.

An implementation must treat the access token as a secret. Shopify currently
sets client-credentials access tokens to expire after 24 hours.

## Validate workflow-specific access

A successful shop query proves authentication, not every scope. Validate each
additional permission with the smallest relevant read:

| Workflow prerequisite | Read-only check |
|:--|:--|
| Catalog | Read a small product/variant page |
| Locations | List locations and identify the intended test location |
| Inventory | Read tracked inventory at that location |
| Orders | Read a small permitted order page |

Do not grant write scopes just to make a connection check succeed.

## Safe success result

A useful result contains:

- The intended shop name or permanent hostname.
- Requested API version.
- API version Shopify actually used.
- Whether those versions match.
- Confirmation that the selected read succeeded.

It must not contain the client secret, access token, complete request headers,
or unrelated API payload data.

## Ready to continue

Continue only when:

- The app is installed on the intended dev store.
- Authentication succeeds.
- Shopify uses the intended supported API version.
- The smallest read required by the selected workflow succeeds.
- Returned products or locations belong to the intended test store.

## Implementations

### PHP

From the generated project directory, run:

```bash
./check-connection
```

This starts the Docker development service when needed, waits for dependencies
to be ready, and runs the read-only `connection:check` console command. For an
already-running container, the equivalent command is:

```bash
docker compose exec integration php bin/bifrost-connect connection:check
```

Add `--json` to either form for structured output. The command never displays
the Shopify access token, client secret, or tracezilla API key.

Additional platform instructions will be added as their focused repositories
become available.

## Official Shopify references

- [Client credentials grant](https://shopify.dev/docs/apps/build/authentication-authorization/access-tokens/client-credentials-grant)
- [GraphQL Admin API](https://shopify.dev/docs/api/admin-graphql)
- [Shopify API versioning](https://shopify.dev/docs/api/usage/versioning)
