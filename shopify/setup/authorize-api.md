---
layout: default
nav_order: 50
parent: Shopify Prerequisites
title: Authorize API
---

# Create and install a Shopify app

The integration needs an app that declares its Admin API permissions and is
installed on the development store. Creating an app, releasing a version, and
installing that version are separate steps.

## Choose the minimum access scopes

Grant only the access needed by the selected workflows:

| Use case | Shopify access scope |
|:--|:--|
| Read products and variants | `read_products` |
| List locations | `read_locations` |
| Read inventory | `read_inventory` |
| Set inventory | `write_inventory` plus required read access |
| Read recent orders | `read_orders` |

Catalog inspection needs `read_products`; it does not need order or
inventory-write access.

## Create the app

1. Open the [Shopify Dev Dashboard](https://dev.shopify.com/dashboard).
2. Select the organization used for the development store.
3. Open **Apps** and choose **Create app**.
4. Give the app a descriptive name such as `tracezilla integration test`.
5. Open **Versions** and create a version.
6. Select the intended Admin API version.
7. Select the minimum scopes from the table above.
8. Give the version a meaningful name and release it.

If you later change scopes, create and release another app version. A released
version declares what the app requests; it does not silently grant new access
to an existing store installation.

## Install the app

1. Open the app's **Home** page in the Dev Dashboard.
2. Select **Install app**.
3. Select the development store created for this project.
4. Review the requested permissions carefully.
5. Confirm the installation.

If a later version adds scopes, Shopify requires an administrator to approve
the additional access for the installed app.

## Store the credentials

Locate the app's **Client ID** and **Client secret**. Store them in a password
manager or secret store. Do not place credentials in documentation,
screenshots, committed files, or terminal output.

Each implementation repository will document its own configuration names. The
values normally include:

```dotenv
SHOPIFY_SHOP_URL=example-integration-test.myshopify.com
SHOPIFY_CLIENT_ID=secret-value
SHOPIFY_CLIENT_SECRET=secret-value
SHOPIFY_SCOPE=read_products,read_locations
SHOPIFY_API_VERSION=reviewed-version
```

Choose scopes to match the released app version and selected workflow. A local
scope setting cannot grant access that the installed app has not approved.

## Access-token behavior

The current examples use Shopify's client credentials grant for an app owned
by the same organization as the store. Access tokens expire after 24 hours;
implementations must request a replacement when needed and must never print
the token.

## Verify the result

- The app has a released version.
- The version declares only required scopes.
- The app is installed on the intended development store.
- The store approved the active scopes.
- The client ID and secret are outside version control.

Continue with [Validate Connection](./connection-validation.html).

## Official Shopify references

- [Create apps using the Dev Dashboard](https://shopify.dev/docs/apps/build/dev-dashboard/create-apps-using-dev-dashboard)
- [Client credentials grant](https://shopify.dev/docs/apps/build/authentication-authorization/access-tokens/client-credentials-grant)
- [Shopify API access scopes](https://shopify.dev/docs/api/usage/access-scopes)
