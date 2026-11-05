---
title: Seed the Test Dataset
layout: default
parent: tracezilla Prerequisites
nav_order: 36
---

# Seed the test dataset

{: .label .label-yellow }
Writes with `--execute`

Populate your development company with the
[standard test dataset](./test-data.html): partners, a warehouse on your own
company, SKUs that match the external-service sandboxes, and received stock.

The seed only talks to tracezilla, so it works the same in every integration.
Every new project includes the `tracezilla:seed` command, unless it was created
with `--without-seed`. To check, run `php bin/bifrost-connect help` (with
Docker, prefixed as below) and look for `tracezilla:seed`.

Seed only a [development company](./development-company.html), never a
production team.

The seed needs no settings of its own. It adds its warehouse to your own
company: partner number 1, which tracezilla creates together with every company.

## Run the seed

Run these commands in your integration project folder: the folder you created
with `create-project`, under the name you chose. For example:

```bash
cd YOUR_INTEGRATION_NAME
```

Start with a dry run, which shows what would be created and writes nothing.

With Docker:

```bash
docker compose run --rm php php bin/bifrost-connect tracezilla:seed --dry-run
```

Without Docker:

```bash
php bin/bifrost-connect tracezilla:seed --dry-run
```

Check the first lines: they name the team, and your own company as partner
number 1, for example:

```text
Dry run for tracezilla team [dev-woocommerce-abc1]

partner         exists         own_company                         Banana McShopface (partner number 1)
```

If both are your development company, write the data.

With Docker:

```bash
docker compose run --rm php php bin/bifrost-connect tracezilla:seed --execute
```

Without Docker:

```bash
php bin/bifrost-connect tracezilla:seed --execute
```

## Use your own test data

The standard dataset comes with the integration's shared code
([bifrost-core](https://github.com/Happy-Bananas/bifrost-core/blob/main/seeds/tracezilla.yaml)).
To seed other records, save your own file as `seeds/tracezilla.yaml` in the
project folder; the seed then uses it instead. You can also name a file on the
command line, for example `tracezilla:seed seeds/extra.yaml --dry-run`.
