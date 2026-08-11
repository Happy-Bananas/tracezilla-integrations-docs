---
layout: default
nav_order: 20
parent: Setup
grand_parent: Shopify
title: Development Store
---

# Create a Shopify development store

A development store provides a safe Shopify environment for test products,
locations, orders, and app installations. Do not use a customer's production
store while learning or adapting an integration.

## Create the store

1. Open the [Shopify Dev Dashboard](https://dev.shopify.com/dashboard).
2. Select the organization created in the previous guide.
3. Open **Stores**.
4. Choose **Create store**.
5. Give the store a recognizable name that identifies it as integration test
   data.
6. Choose a development-store configuration or plan suitable for testing.
7. Enable generated test data if Shopify offers it and the test case needs
   sample catalog content. The next guide also explains how to create
   predictable test data manually.
8. Create the store and wait for Shopify Admin to open.

Shopify may change the labels used for development-store types and plans. The
important outcome is a test store owned by the intended organization, not a
live merchant store.

## Record the store domain

Find the permanent `myshopify.com` domain in the store details or domain
settings. Record only the hostname, for example:

```text
example-integration-test.myshopify.com
```

Do not substitute a custom storefront domain. Implementations use the
permanent Shopify domain to construct Admin API URLs.

## Verify the result

Confirm that:

- The store appears in the Dev Dashboard.
- You can open its Shopify Admin.
- You know its permanent `myshopify.com` domain.
- The store contains no customer production data.

Next, [create products and locations](./products-and-locations.html).

## Official Shopify reference

[Create and use development stores](https://shopify.dev/docs/apps/build/dev-dashboard/development-stores)
