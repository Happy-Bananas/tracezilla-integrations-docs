---
title: Developer Access
layout: default
parent: Setup
grand_parent: Shopify
nav_order: 10
---

# Obtain Shopify developer access

You need access to an organization that can create apps and dev stores. Shopify
currently supports a Shopify Partner account or a merchant organization whose
administrator has granted the required developer permissions.

For independent integration work, a Partner organization is normally the
cleanest boundary. If your company already has one, ask its owner to invite you
instead of creating a second organization.

## Verify your access

1. Sign in to the [Shopify Dev Dashboard](https://dev.shopify.com/dashboard).
2. Select the organization that will own the integration test assets.
3. Confirm that **Apps** and **Stores** are available.
4. Confirm that your role permits app development and dev-store creation.

If either area is unavailable, ask an organization administrator to review
your permissions. Do not use a customer's live store as a substitute for a
missing test environment.

## Ownership matters

The examples use Shopify's client credentials grant for trusted server-to-server
integration. Shopify limits this grant to apps developed by your own
organization and installed in stores that your organization owns. Integrating
an independently owned merchant store requires a different authorization flow
and is outside this initial setup path.

Continue with [Create a dev store](./dev-store.html).

## Official Shopify references

- [Dev stores: requirements](https://shopify.dev/docs/apps/build/dev-dashboard/development-stores#requirements)
- [Client credentials grant: requirements](https://shopify.dev/docs/apps/build/authentication-authorization/access-tokens/client-credentials-grant#requirements)
