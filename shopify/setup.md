---
title: Setup
layout: default
parent: Shopify
nav_order: 10
has_children: true
---

# Shopify setup

Create a safe Shopify testing environment, prepare predictable catalog data,
and authorize only the Admin API access required by the selected workflow.

The instructions use Shopify's current **Dev Dashboard** and **dev store**
terminology. Shopify changes interface labels periodically; verify the outcome
of each step rather than relying on an exact screen layout.

## Setup path

Complete these guides in order:

1. [Obtain developer access](./setup/developer-access.html).
2. [Create a dev store](./setup/dev-store.html).
3. [Prepare products and locations](./setup/test-data.html).
4. [Create and install an app](./setup/app-authorization.html).
5. [Validate the Shopify connection](./setup/connection-validation.html).

## Expected result

At the end of setup, you should have:

- Access to a Shopify organization with app-development permission.
- A dev store containing no customer production data.
- Its permanent `myshopify.com` hostname.
- At least one active location and one product variant with a unique test SKU.
- An installed app version with the minimum required access scopes.
- A client ID and client secret stored outside version control.
- A successful read-only authentication and API-version check.

Do not test catalog, inventory, or order writes in a customer's production
store while adapting an example.

## Official Shopify references

- [Dev stores](https://shopify.dev/docs/apps/build/dev-dashboard/development-stores)
- [Create apps using the Dev Dashboard](https://shopify.dev/docs/apps/build/dev-dashboard/create-apps-using-dev-dashboard)
- [Shopify API access scopes](https://shopify.dev/docs/api/usage/access-scopes)

This section will cover development stores, products and locations, inventory
preparation, API authorization, and safe connection validation.
