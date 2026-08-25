---
title: Shopify Prerequisites
layout: default
nav_order: 30
has_children: true
---

# Shopify prerequisites

Create a safe Shopify development environment, prepare predictable test data,
and authorize the minimum Admin API access needed by the integration.

No prior Shopify development experience is required. The guides are written as
text-first procedures because Shopify may change dashboard layouts and labels
without changing the underlying setup outcome.

## Setup path

Complete these guides in order:

1. [Create or join a Partner account](./setup/partner-account.html).
2. [Create a development store](./setup/development-store.html).
3. [Create products and locations](./setup/products-and-locations.html).
4. [Prepare inventory tracking](./setup/inventory-setup.html) when the selected
   workflow synchronizes inventory.
5. [Create, release, and install an app](./setup/authorize-api.html).
6. [Validate the Shopify connection](./setup/connection-validation.html).

## Expected result

At the end of setup, you should have:

- A Shopify Partner organization.
- A development store with a permanent `myshopify.com` domain.
- At least one active location and one product variant with a unique test SKU.
- An installed app version with the minimum required access scopes.
- A client ID and client secret stored outside version control.
- A successful read-only authentication and API-version check.

Use development data throughout this section. Do not test inventory or order
writes in a customer's production store.

## Official Shopify references

- [Development stores](https://shopify.dev/docs/apps/build/dev-dashboard/development-stores)
- [Create apps using the Dev Dashboard](https://shopify.dev/docs/apps/build/dev-dashboard/create-apps-using-dev-dashboard)
- [Shopify API access scopes](https://shopify.dev/docs/api/usage/access-scopes)
