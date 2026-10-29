---
title: Getting Started
layout: default
nav_order: 10
---

# Getting started

Each external service has its own standalone PHP integration. The steps are
the same for every service; only the service pages differ.

## 1. Prepare tracezilla

Shared by all integrations:

1. [Create a development company](./fundamentals/development-company.html).
   Never develop against a production team.
2. [Acquire credentials](./fundamentals/account-and-team.html): the team slug
   and either a personal access token (Shopify) or an OAuth client
   (WooCommerce).

## 2. Prepare the external service

Choose the service you are integrating with and complete its prerequisites:

| Service | Prerequisites |
|:--|:--|
| Shopify | [Shopify Prerequisites](./shopify/setup.html): development store and app |
| WooCommerce | [WooCommerce Prerequisites](./woocommerce/setup.html): development store and REST API keys |

## 3. Create the project

Open a terminal in the folder where you keep your projects and run the command
for your service. Replace `GiantPotatoes` with the name of your project:

Shopify:

```bash
curl -fsSL https://happy-bananas.github.io/tracezilla-integrations-docs/create-project | sh -s -- shopify GiantPotatoes
```

WooCommerce:

```bash
curl -fsSL https://happy-bananas.github.io/tracezilla-integrations-docs/create-project | sh -s -- woocommerce GiantPotatoes
```

The command needs only `curl` and `tar`. It downloads the latest integration
into a new `GiantPotatoes` folder and creates its `.env` file. The project is
yours: a plain folder on your computer, with no Git history and no connection
to the BifrostConnect repositories. The command refuses to replace an existing
folder or to create the project inside a Git repository.

If you later want backup, collaboration, or deployment through Git, run
`git init` in the project and push it to a **private repository owned by you or
the customer**. Never push customer business rules to the public BifrostConnect
repositories.

The WooCommerce test store is created the same way; see
[Development store](./woocommerce/setup/store.html).

## 4. Install the integration

Install it **with Docker** (recommended; PHP and its packages run in a prepared
development container) **or without Docker** (PHP and Composer installed on your
computer). Both give the same project.

| Service | With Docker | Without Docker |
|:--|:--|:--|
| Shopify | [Install with Docker](./run-with-docker.html) | [Install without Docker](./manual-installation.html) |
| WooCommerce | [Install with Docker](./woocommerce/install-with-docker.html) | [Install without Docker](./woocommerce/install-without-docker.html) |

## 5. Check both connections

From the new project folder, run:

```bash
./check-connection
```

Without Docker, run `php bin/bifrost-connect connection:check`.

The command makes one small read-only request to the external service and one
to tracezilla, and reports both. **Nothing else works until both connections
pass**, so fix any reported problem before continuing.

## 6. Next steps

- [Seed the test dataset](./fundamentals/seed.html) into your development
  company.
- Shopify: [Implement Custom Business Logic](./shopify/custom-business-logic.html)
  and the Shopify commands.
- WooCommerce: the [WooCommerce commands](./woocommerce/).
- When it works locally, see [Deployment](./deployment.html).
