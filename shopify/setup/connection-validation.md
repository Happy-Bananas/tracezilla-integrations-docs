---
title: Validate Connection
layout: default
parent: Shopify Prerequisites
nav_order: 60
grand_parent: Shopify
---

# Validate the Shopify connection

Check the connection before you run catalog, inventory, or order commands. The
check is read-only and never prints the Shopify access token, the client
secret, or the tracezilla tokens.

## Run the check

From the project folder, run:

With Docker:

```bash
./check-connection
```

The helper installs the PHP packages if needed and runs `connection:check`. On
Windows (PowerShell), run it directly:

```powershell
docker compose run --rm php php bin/bifrost-connect connection:check
```

Without Docker:

```bash
php bin/bifrost-connect connection:check
```

Add `--json` to any of these for structured output.

A successful result looks like:

```text
Connection check passed.
Shopify: Example Shop (example-shop.myshopify.com)
tracezilla: connected
```

## What the check does

1. Checks that the Shopify and tracezilla settings in `.env` are present.
2. Requests a Shopify access token using the client credentials grant. The
   token expires after 24 hours, so the integration requests a new one on
   every run.
3. Reads the shop name and permanent domain with a small GraphQL Admin API
   query.
4. Reads one SKU from tracezilla, renewing the tracezilla token if needed.

The check stops at the first failure and prints its reason. Make sure the shop
name and domain belong to your development store.

## Check access for each command

A successful check proves that authentication works, not that the app has
every scope a command needs. Before you rely on a command, run its read-only
form once against the development store:

| Command needs | Read-only way to check it |
|:--|:--|
| Catalog | [Compare Catalogs](../compare-catalogs.html) |
| Locations | [List Shopify Locations](../list-shopify-locations.html) |
| Inventory | A dry run of [Synchronize Inventory](../synchronize-inventory.html) |
| Orders | [Report Collected Orders](../report-collected-orders.html) |

Do not grant write scopes just to make a check succeed.

## Ready to continue

Continue when:

- The app is installed on your development store.
- The connection check passes and names your development store.
- The read-only check for the command you need succeeds.

## Official Shopify references

- [Client credentials grant](https://shopify.dev/docs/apps/build/authentication-authorization/access-tokens/client-credentials-grant)
- [GraphQL Admin API](https://shopify.dev/docs/api/admin-graphql)
- [Shopify API versioning](https://shopify.dev/docs/api/usage/versioning)
