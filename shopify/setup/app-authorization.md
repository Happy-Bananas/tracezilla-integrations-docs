---
title: App Authorization
layout: default
parent: Setup
grand_parent: Shopify
nav_order: 40
---

# Create and install a Shopify app

The integration needs an app that declares its Admin API permissions and is
installed on the dev store. Creating an app, releasing a version, installing
it, and acquiring an access token are separate operations.

## Choose minimum access scopes

Grant only the access required by the selected workflow:

| Use case | Access scope |
|:--|:--|
| Read products and variants | `read_products` |
| List locations | `read_locations` |
| Read inventory | `read_inventory` |
| Update inventory | `write_inventory` plus required read access |
| Read recent orders | `read_orders` |

Catalog connection validation needs `read_products`; it does not require order
or inventory-write access.

## Create and release an app version

1. Open the [Shopify Dev Dashboard](https://dev.shopify.com/dashboard).
2. Select the organization and open **Apps**.
3. Create an app with a recognizable test name.
4. Open **Versions** and create a version.
5. Select the intended Admin API version.
6. Add the minimum required access scopes.
7. Release the version.

An app needs a released version before it can be installed. When a later
version requests additional scopes, installed stores do not receive them
silently; an administrator must approve the new access.

## Install the app

1. Open the app's **Home** page in the Dev Dashboard.
2. Select **Install app**.
3. Select the dev store created for this project.
4. Review the requested permissions.
5. Confirm the installation.

## Protect the client credentials

Locate the client ID and client secret in the Dev Dashboard. Store them in a
password manager or appropriate secret store.

Never place credentials or access tokens in:

- Git-tracked files.
- Documentation or screenshots.
- Command output saved in support tickets.
- Browser URLs.
- Application logs.

The client credentials grant exchanges the client ID and secret for an access
token. Shopify currently issues these tokens for 24 hours, so an implementation
must reacquire a token when necessary rather than treating it as permanent.

Continue with [Validate the Shopify connection](./connection-validation.html).

## Official Shopify references

- [Create apps using the Dev Dashboard](https://shopify.dev/docs/apps/build/dev-dashboard/create-apps-using-dev-dashboard)
- [Client credentials grant](https://shopify.dev/docs/apps/build/authentication-authorization/access-tokens/client-credentials-grant)
- [Admin API access scopes](https://shopify.dev/docs/api/usage/access-scopes)
