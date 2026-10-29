---
title: Seed the Test Dataset
layout: default
parent: tracezilla Prerequisites
nav_order: 36
---

# Seed the test dataset

{: .label .label-yellow }
Writes with explicit confirmation

Populate your development company with the
[standard test dataset](./test-data.html): partners, a warehouse on your own
company, SKUs that match the external-service sandboxes, and received stock.

The seed command and its YAML file currently ship with the WooCommerce PHP
integration, `tracezilla-woocommerce-php`. The data itself is
platform-neutral and can be used with any integration.

## Before you start

- A [development company](./development-company.html). Never seed a
  production team.
- [Credentials](./account-and-team.html) with the OAuth client, and a completed
  `tracezilla:login`.
- The WooCommerce PHP integration installed with
  [Docker](../woocommerce/install-with-docker.html) or
  [without Docker](../woocommerce/install-without-docker.html).

Add the exact partner name of your own company in tracezilla to `.env`. The
seed adds its warehouse location to that partner:

```dotenv
TRACEZILLA_OWN_COMPANY=Your development company
```

## Run the seed

Preview what would be created. Nothing is written:

### With Docker

```bash
docker compose run --rm php php bin/bifrost-connect tracezilla:seed
```

### Without Docker

```bash
php bin/bifrost-connect tracezilla:seed
```

Check that the first line names your development company, then write the
data:

```bash
docker compose run --rm php php bin/bifrost-connect tracezilla:seed --execute --confirm
```

Both `--execute` and `--confirm` are required. Running the command again
reports every record as `exists`. To use another dataset, pass the path of its
YAML file as the first argument.

## What the seed does

The seed reads [`seeds/tracezilla.yaml`](https://github.com/Happy-Bananas/tracezilla-woocommerce-php/blob/main/seeds/tracezilla.yaml)
and creates the records that are missing:

1. Locations on your own company (the warehouse).
2. Partners and their locations.
3. SKUs.
4. Purchase orders, received into the warehouse.

Records are identified by `ref` (partners and locations), `sku_code` (SKUs),
and `ext_ref` (purchase orders). Existing records are reported as `exists` and
never changed.

The seed handles some tracezilla defaults for you:

- New partners are owned by the user who runs the seed, unless the YAML sets
  `owned_by_id`. Order import needs the customer partner to have an owner.
- tracezilla derives an SKU's lot unit from its unit of measure when the SKU is
  created. The seed then stores the units from the YAML file and verifies them.

## Exit status

Runs without failed or invalid records return exit code `0`. Configuration
errors, API errors, and failed or invalid records return a non-zero exit code.

The seed takes the integration's run lock. If another command that writes is
already running, the seed is skipped with exit code `75` and changes nothing;
run it again when the other command has finished.
