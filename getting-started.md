---
title: Getting Started
layout: default
nav_order: 10
---

# Getting started

Each external service has its own standalone PHP integration. The steps are
the same for every service; only the service pages differ.

## 1. Set up your development environment

Choose to work [with Docker or without Docker](./development-environment.html).
With Docker, Docker is the only thing you install.

## 2. Prepare tracezilla

Shared by all integrations:

1. [Create a development company](./fundamentals/development-company.html).
   Never develop against a production team.
2. [Acquire credentials](./fundamentals/account-and-team.html): the team slug
   and an OAuth client.

## 3. Prepare the external service

Choose the service you are integrating with and complete its prerequisites:

| Service | Prerequisites |
|:--|:--|
| Shopify | [Shopify Prerequisites](./shopify/setup.html): development store and app |
| WooCommerce | [WooCommerce Prerequisites](./woocommerce/setup.html): development store and REST API keys |

## 4. Create the project

Open a terminal in the folder where you keep your projects. In the commands
below, replace `woocommerce` with `shopify` for a Shopify project, and
`YOUR_INTEGRATION_NAME` with the name of your integration.

### Mac and Linux

```bash
curl -fsSL https://happy-bananas.github.io/tracezilla-integrations-docs/create-project | sh -s -- woocommerce YOUR_INTEGRATION_NAME
```

### Windows with Docker

In PowerShell, the same command runs in a small container:

```powershell
docker run --rm -v "${PWD}:/work" -w /work alpine:3 sh -c "wget -qO- https://happy-bananas.github.io/tracezilla-integrations-docs/create-project | sh -s -- woocommerce YOUR_INTEGRATION_NAME"
```

### Windows without Docker

1. Download the integration as a ZIP file:
   [WooCommerce](https://github.com/Happy-Bananas/tracezilla-woocommerce-php/archive/refs/heads/main.zip)
   or [Shopify](https://github.com/Happy-Bananas/tracezilla-shopify-php/archive/refs/heads/main.zip).
2. Extract it in the folder where you keep your projects and rename the
   extracted folder to `YOUR_INTEGRATION_NAME`.
3. In PowerShell, create `.env` in the project folder:

   ```powershell
   cd YOUR_INTEGRATION_NAME
   Copy-Item .env.example .env
   ```

### What you get

The command downloads the latest integration into a new `YOUR_INTEGRATION_NAME` folder
and creates its `.env` file. The project is
yours: a plain folder on your computer, with no Git history and no connection
to the BifrostConnect repositories. The command refuses to replace an existing
folder or to create the project inside a Git repository.

If you later want backup, collaboration, or deployment through Git, run
`git init` in the project and push it to a **private repository owned by you or
the customer**. Never push customer business rules to the public BifrostConnect
repositories.

The WooCommerce test store is created the same way; see
[Development store](./woocommerce/setup/store.html).

## 5. Install the integration

Follow the install page for your service and your choice of
[Docker or no Docker](./development-environment.html):

| Service | With Docker | Without Docker |
|:--|:--|:--|
| Shopify | [Install with Docker](./run-with-docker.html) | [Install without Docker](./manual-installation.html) |
| WooCommerce | [Install with Docker](./woocommerce/install-with-docker.html) | [Install without Docker](./woocommerce/install-without-docker.html) |

## 6. Check both connections

{: .important }
With Docker, start Docker first, and the WooCommerce sandbox if you use it.
Neither starts by itself after the computer restarts.

From the new project folder, run:

| | Command |
|:--|:--|
| With Docker, Mac and Linux | `./check-connection` |
| With Docker, Windows | See Check Connections for your service |
| Without Docker | `php bin/bifrost-connect connection:check` |

The command makes one small read-only request to the external service and one
to tracezilla, and reports both. **Nothing else works until both connections
pass**, so fix any reported problem before continuing.

## 7. Next steps

- [Seed the test dataset](./fundamentals/seed.html) into your development
  company.
- Shopify: [Implement Custom Business Logic](./shopify/custom-business-logic.html)
  and the Shopify commands.
- WooCommerce: your first [scenario](./woocommerce/scenarios.html), then the
  [WooCommerce commands](./woocommerce/).
- When it works locally, see [Deployment](./deployment.html).
