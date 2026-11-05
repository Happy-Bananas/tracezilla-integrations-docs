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

## Name your own company and allow the team

Add two lines to the `.env` file in your integration project folder:

```dotenv
TRACEZILLA_OWN_COMPANY=Your development company
TRACEZILLA_SEED_ALLOWED_TEAM=dev-woocommerce-abc1
```

- `TRACEZILLA_OWN_COMPANY` is the exact partner name of your own company in
  tracezilla. The seed adds its warehouse location to that partner.
- `TRACEZILLA_SEED_ALLOWED_TEAM` is the team slug of your development company,
  the same value as `TRACEZILLA_TEAM_SLUG`. The seed writes only into this
  team. Never set it in a project that works on a customer's production team:
  without it, `--execute` stops with exit code 2 before it contacts tracezilla,
  so the test data cannot end up there by mistake.

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

Check that the first line names your development company, then write the
data. If the command says that `TRACEZILLA_SEED_ALLOWED_TEAM` is not set or
names another team, nothing was written; check the two team settings in
`.env`.

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
