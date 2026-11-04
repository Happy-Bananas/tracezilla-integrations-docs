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

The seed only talks to tracezilla, so it works the same in every integration.
It is optional: a project has the `tracezilla:seed` command and the dataset,
`seeds/tracezilla.yaml`, only when you create it with `--with-seed`:

```bash
curl -fsSL https://happy-bananas.github.io/tracezilla-integrations-docs/create-project | sh -s -- woocommerce YOUR_INTEGRATION_NAME --with-seed
```

To check whether your project has the seed, run `php bin/bifrost-connect help`
(with Docker, prefixed as below) and look for `tracezilla:seed`.

## Before you start

- A [development company](./development-company.html). Never seed a
  production team.
- [Credentials](./account-and-team.html) with the OAuth client, and a completed
  `tracezilla:login`.
- Your integration installed, as described in
  [Getting Started](../getting-started.html), and created with `--with-seed`.

Add the exact partner name of your own company in tracezilla to the `.env` file
in your integration project folder. The seed adds its warehouse location to
that partner:

```dotenv
TRACEZILLA_OWN_COMPANY=Your development company
```

### Add the seed to an existing project

If you created your project without the seed, create a second, temporary
project of the same platform with `--with-seed`, and copy these into your
project: `bin/seed-tracezilla`, `src/Seed/`, `seeds/`, and `tests/Unit/Seed/`.
Then add the `TRACEZILLA_OWN_COMPANY` line above to `.env`.

## Run the seed

Run these commands in your integration project folder: the folder you created
with `create-project`, under the name you chose. For example:

```bash
cd YOUR_INTEGRATION_NAME
```

The command needs a mode. Start with a dry run, `--dry-run`, which shows what
would be created and writes nothing.

With Docker, in a WooCommerce integration:

```bash
docker compose run --rm php php bin/bifrost-connect tracezilla:seed --dry-run
```

With Docker, in a Shopify integration:

```bash
docker compose exec integration php bin/bifrost-connect tracezilla:seed --dry-run
```

Without Docker:

```bash
php bin/bifrost-connect tracezilla:seed --dry-run
```

Check that the first line names your development company, then write the
data.

With Docker, in a WooCommerce integration:

```bash
docker compose run --rm php php bin/bifrost-connect tracezilla:seed --execute --confirm
```

With Docker, in a Shopify integration:

```bash
docker compose exec integration php bin/bifrost-connect tracezilla:seed --execute --confirm
```

Without Docker:

```bash
php bin/bifrost-connect tracezilla:seed --execute --confirm
```

Both `--execute` and `--confirm` are required. Without a mode, or with
`--dry-run` and `--execute` together, the command refuses to run. Running it
again reports every record as `exists`. To use another dataset, pass the path
of its YAML file as the first argument.

The Shopify Docker commands use `exec`, because the Shopify development
container keeps running after `./check-connection` has started it.

## What the seed does

The seed reads `seeds/tracezilla.yaml` in your integration project and creates
the records that are missing:

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
