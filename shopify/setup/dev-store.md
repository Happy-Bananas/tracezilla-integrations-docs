---
title: Dev Store
layout: default
parent: Setup
grand_parent: Shopify
nav_order: 20
---

# Create a Shopify dev store

A dev store is a testing environment owned and controlled by your organization.
It lets you test products, locations, orders, and app installations without
affecting a live merchant store.

## Create the store

1. Open the [Shopify Dev Dashboard](https://dev.shopify.com/dashboard).
2. Select the organization used for this project.
3. Select **Stores**.
4. Select **Create store**.
5. Give the store a name that clearly identifies it as integration test data.
6. Select a plan appropriate for the features you need to test.
7. Create the store, then sign in to its Shopify Admin.

Shopify may offer additional plan options or feature previews. Enable only the
features needed by the integration scenario; feature-preview stores can have
additional limitations.

## Record the permanent hostname

Find the store's permanent `myshopify.com` domain and record only its hostname:

```text
example-integration-test.myshopify.com
```

Use this hostname when constructing Admin API URLs. Do not substitute a custom
storefront domain.

## Verify the result

- The store appears under **Stores** in the intended Dev Dashboard organization.
- You can open its Shopify Admin.
- You know its permanent `myshopify.com` hostname.
- It contains no merchant production data.

Continue with [Prepare products and locations](./test-data.html).

## Official Shopify reference

[Create and use dev stores](https://shopify.dev/docs/apps/build/dev-dashboard/development-stores)
