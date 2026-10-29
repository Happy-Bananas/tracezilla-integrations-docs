---
title: Getting Started
layout: default
nav_order: 10
---

# Getting started

Each commerce platform has its own standalone PHP integration. The steps are
the same for every platform; only the platform pages differ.

## 1. Prepare tracezilla

Shared by all integrations:

1. [Create a development company](./fundamentals/development-company.html).
   Never develop against a production team.
2. [Acquire credentials](./fundamentals/account-and-team.html): the team slug
   and either a personal access token (Shopify) or an OAuth client
   (WooCommerce).

## 2. Prepare the commerce platform

Choose the platform you are integrating with and complete its prerequisites:

| Platform | Prerequisites |
|:--|:--|
| Shopify | [Shopify Prerequisites](./shopify/setup.html): development store and app |
| WooCommerce | [WooCommerce Prerequisites](./woocommerce/setup.html): development store and REST API keys |

## 3. Install the integration

Install it **with Docker** (recommended; PHP and its packages run in a prepared
development container) **or without Docker** (PHP and Composer installed on your
computer). Both give the same project.

| Platform | With Docker | Without Docker |
|:--|:--|:--|
| Shopify | [Install with Docker](./run-with-docker.html) | [Install without Docker](./manual-installation.html) |
| WooCommerce | [Install with Docker](./woocommerce/install-with-docker.html) | [Install without Docker](./woocommerce/install-without-docker.html) |

## 4. Check both connections

From the new project folder, run:

```bash
./check-connection
```

Without Docker, run `php bin/bifrost-connect connection:check`.

The command makes one small read-only request to the commerce platform and one
to tracezilla, and reports both. **Nothing else works until both connections
pass**, so fix any reported problem before continuing.

## 5. Next steps

- [Seed the test dataset](./fundamentals/seed.html) into your development
  company.
- Shopify: [Implement Custom Business Logic](./shopify/custom-business-logic.html)
  and the Shopify commands.
- WooCommerce: the [WooCommerce commands](./woocommerce/).
- When it works locally, see [Deployment](./deployment.html).
