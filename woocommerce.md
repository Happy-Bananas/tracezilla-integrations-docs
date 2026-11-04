---
title: WooCommerce
layout: default
nav_order: 50
has_children: true
permalink: /woocommerce/
---

# WooCommerce

Connect WooCommerce with tracezilla using the standalone PHP integration
`tracezilla-woocommerce-php`. It accesses WooCommerce through its REST API and
does not need to be installed as a WordPress plugin.

## Start here

1. Prepare [tracezilla](../tracezilla-fundamentals.html): a development company
   and an OAuth client.
2. Complete the [WooCommerce prerequisites](./setup.html): a development store
   and REST API keys.
3. Set up your integration [with Docker](./install-with-docker.html) or
   [without Docker](./install-without-docker.html).
4. [Check both connections](./check-connections.html). Nothing else works until
   this passes.
5. [Seed the test dataset](../fundamentals/seed.html) into your development
   company.
6. Create and run your first [scenario](./scenarios.html).

## Where to run the commands

You work in two folders. Their names are the ones you chose when you created
them, so yours may differ from the examples:

| Folder | Created with | Run there |
|:--|:--|:--|
| Your integration project, for example `YOUR_INTEGRATION_NAME` | `create-project woocommerce YOUR_INTEGRATION_NAME` | Every `bin/bifrost-connect` command, `./check-connection`, `composer test`, and everything that starts with `docker compose run --rm php` |
| The WooCommerce sandbox, for example `woocommerce-sandbox` | `create-project woocommerce-sandbox woocommerce-sandbox` | `docker compose up -d` and the other sandbox commands on [Development Store](./setup/store.html) |

Change to the right folder before you run a command, for example:

```bash
cd YOUR_INTEGRATION_NAME
```

Each folder has its own `compose.yaml`, so a `docker compose` command acts on
the folder you are in. In the wrong folder it starts the wrong containers, or
fails with `no configuration file provided`.

## Coverage

Every command runs through one entry point, `bin/bifrost-connect`, in your
integration project folder. To list the commands:

With Docker:

```bash
docker compose run --rm php php bin/bifrost-connect help
```

Without Docker:

```bash
php bin/bifrost-connect help
```

| Command | Purpose | Status |
|:--|:--|:--|
| `connection:check` | [Check Connections](./check-connections.html) | Available |
| `tracezilla:login` | [Log in to tracezilla](../fundamentals/account-and-team.html#oauth-client-and-login) | Available |
| `tracezilla:seed` | [Seed the Test Dataset](../fundamentals/seed.html) | Available |
| `scenario:create`, `scenario:run` | [Scenarios](./scenarios.html) | Available |
| Template `compare-catalogs` | [Compare Catalogs](./scenarios/compare-catalogs.html) | Available |
| Template `missing-in-woocommerce` | [Missing in WooCommerce](./scenarios/missing-in-woocommerce.html) | Available |
| Template `create-tracezilla-skus` | [Create tracezilla SKUs](./scenarios/create-tracezilla-skus.html) | Available |
| Your own catalog, inventory, and order rules | [Scenarios](./scenarios.html) | Write them as scenarios |

Commands that write take a run lock, so scheduled runs never overlap. A run
that finds the lock taken is skipped with exit code `75` and changes nothing.
