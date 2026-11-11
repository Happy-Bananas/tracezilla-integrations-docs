---
title: Shopify Prerequisites
layout: default
nav_order: 2
has_children: true
parent: Shopify
---

# Shopify prerequisites

Before your integration can connect, Shopify needs a development store and an
app installed on it. This page lists what the integration needs; how to click
your way there is in Shopify's own guides, linked below, because Shopify's
dashboard changes often.

Use a development store throughout. Do not test inventory or order writes in
a customer's production store.

## What you need

### 1. A development store

Create one in the [Shopify Dev Dashboard](https://dev.shopify.com/dashboard),
in the same organization that will own the app. Note its permanent
`myshopify.com` domain, not a custom storefront domain.

Shopify's guide:
[Create and use development stores](https://shopify.dev/docs/apps/build/dev-dashboard/development-stores).

### 2. Test products and a location

- At least one active location, for example `Development Warehouse`.
- A few products whose variants have unique SKUs. Use SKUs from the
  [standard test dataset](../fundamentals/test-data.html), so they match what
  the [seed](../fundamentals/seed.html) creates in tracezilla:

  ```text
  TZ-COFFEE-001-01    House Blend Coffee Beans, 250 g
  TZ-COFFEE-001-02    House Blend Coffee Beans, 1 kg
  TZ-FRUIT-001        Fairtrade Bananas 1 kg
  ```

- For inventory sync: quantity tracking turned on for each variant, the
  variant stocked at the location, and a quantity that differs from
  tracezilla's, so the dry run has something to show.

### 3. An app with the scopes your commands use

Create the app in the Dev Dashboard, in the same organization as the store.
The integration logs in with Shopify's client credentials grant, which works
only for an app owned by the store's organization.

Release an app version with the Admin API version from `.env.example` and only
the scopes your commands need:

| Command | Scopes |
|:--|:--|
| Connection check, catalog commands | `read_products` |
| `shopify:locations` | `read_locations` |
| `inventory:sync` | `read_products`, `read_locations`, `read_inventory`, `write_inventory` |
| Order commands | `read_orders` |

Then install the app on the development store. A new app version with more
scopes must be approved on the store again.

Shopify's guides:
[Create apps using the Dev Dashboard](https://shopify.dev/docs/apps/build/dev-dashboard/create-apps-using-dev-dashboard),
[Access scopes](https://shopify.dev/docs/api/usage/access-scopes),
[Client credentials grant](https://shopify.dev/docs/apps/build/authentication-authorization/access-tokens/client-credentials-grant).

### 4. The values for `.env`

From the store and the app's settings:

```dotenv
SHOPIFY_SHOP_URL=example-integration-test.myshopify.com
SHOPIFY_CLIENT_ID=...
SHOPIFY_CLIENT_SECRET=...
SHOPIFY_SCOPE=read_products,read_locations
SHOPIFY_API_VERSION=2025-10
```

`SHOPIFY_SCOPE` lists the scopes of the released app version, separated by
commas. The client secret is a secret: keep it out of Git, screenshots, and
terminal output.

## Expected result

At the end, you have:

- A development store with a permanent `myshopify.com` domain.
- At least one active location and one product variant with a test SKU.
- An app, owned by the store's organization, installed with the scopes your
  commands need.
- Its client ID and secret in `.env`, and nowhere in Git.
- A passing [connection check](./setup/connection-validation.html).
