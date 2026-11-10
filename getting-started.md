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
below, replace `PLATFORM` with the platform you integrate with, and
`YOUR_INTEGRATION_NAME` with the name of your integration.

| Platform | `PLATFORM` | ZIP download (Windows without Docker) |
|:--|:--|:--|
| Shopify | `shopify` | [tracezilla-shopify-php](https://github.com/Happy-Bananas/tracezilla-shopify-php/archive/refs/heads/main.zip) |
| WooCommerce | `woocommerce` | [tracezilla-woocommerce-php](https://github.com/Happy-Bananas/tracezilla-woocommerce-php/archive/refs/heads/main.zip) |

### Mac and Linux

```bash
curl -fsSL https://happy-bananas.github.io/tracezilla-integrations-docs/create-project | sh -s -- PLATFORM YOUR_INTEGRATION_NAME
```

### Windows with Docker

In PowerShell, the same command runs in a small container:

```powershell
docker run --rm -v "${PWD}:/work" -w /work alpine:3 sh -c "wget -qO- https://happy-bananas.github.io/tracezilla-integrations-docs/create-project | sh -s -- PLATFORM YOUR_INTEGRATION_NAME"
```

### Windows without Docker

1. Download the ZIP file for your platform from the table above.
2. Extract it in the folder where you keep your projects and rename the
   extracted folder to `YOUR_INTEGRATION_NAME`.
3. In PowerShell, create `.env` in the project folder:

   ```powershell
   cd YOUR_INTEGRATION_NAME
   Copy-Item .env.example .env
   ```

### What you get

The command downloads the latest release of the integration into a new
`YOUR_INTEGRATION_NAME` folder and creates its `.env` file. `TEMPLATE_VERSION`
in the folder records the release; see [Update Your Integration](./update.html).
The project is yours: a plain folder on your computer, with no Git history and
no connection to the BifrostConnect repositories. The command refuses to replace an existing
folder or to create the project inside a Git repository.

The project includes the tracezilla seed, which fills a development company
with test data; see [Seed the Test Dataset](./fundamentals/seed.html). If you
will work directly on the customer's production data, add `--without-seed`
after the project name to leave it out.

If you later want backup, collaboration, or deployment through Git, run
`git init` in the project and push it to a **private repository owned by you or
the customer**. Never push customer business rules to the public BifrostConnect
repositories.

The WooCommerce test store is created the same way; see
[Development store](./woocommerce/setup/store.html).

## 5. Set up your integration

Follow the setup page for your service and your choice of
[Docker or no Docker](./development-environment.html):

| Service | With Docker | Without Docker |
|:--|:--|:--|
| Shopify | [Set up with Docker](./run-with-docker.html) | [Set up without Docker](./manual-installation.html) |
| WooCommerce | [Set up with Docker](./woocommerce/install-with-docker.html) | [Set up without Docker](./woocommerce/install-without-docker.html) |

## 6. Check both connections

{: .important }
With Docker, start Docker first, and the WooCommerce sandbox if you use it.
Neither starts by itself after the computer restarts.

The check needs the PHP packages. The setup page in step 5 installs them; if
you skipped it, run this once from the new project folder. With Docker:

```bash
docker compose run --rm php composer install
```

Without Docker:

```bash
composer install
```

There is no login step: the first command logs in to tracezilla by itself with
the credentials in `.env`.

Then run the check from the same folder:

| | Command |
|:--|:--|
| With Docker, Mac and Linux | `./check-connection` |
| With Docker, Windows | `docker compose run --rm php php bin/bifrost-connect connection:check` |
| Without Docker | `php bin/bifrost-connect connection:check` |

The command makes one small read-only request to the external service and one
to tracezilla. The WooCommerce check always reports both; the Shopify check
stops at the first failure. **Nothing else works until both connections
pass**, so fix any reported problem before continuing; see
[Troubleshooting](./troubleshooting.html).

## 7. Next steps

- [Seed the test dataset](./fundamentals/seed.html) into your development
  company.
- Shopify: [Implement Custom Business Logic](./shopify/custom-business-logic.html)
  and the Shopify commands.
- WooCommerce: your first [scenario](./woocommerce/scenarios.html), then the
  [WooCommerce commands](./woocommerce/).
- When it works locally, see [Deployment](./deployment.html).
