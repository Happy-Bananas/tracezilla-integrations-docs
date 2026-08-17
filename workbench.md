---
layout: default
title: Integration workbench
nav_order: 4
---

# Integration workbench

The [Tracezilla Integration Workbench](https://github.com/Happy-Bananas/tracezilla-integration-workbench)
currently preserves the proven Laravel application from the legacy Shopify
connector. It is a baseline for checking Shopify and Tracezilla credentials
before the workbench is separated further.

{: .important }
The workbench is a Laravel application. Its command entry point is always
`php artisan`, whether it runs inside Docker or directly on a server. Commands
such as `php bin/compare-catalogs` belong to the separate framework-neutral
`tracezilla-shopify-php` repository and are not interchangeable with workbench
commands.

## Run it locally

Clone the repository and enter it:

```bash
git clone https://github.com/Happy-Bananas/tracezilla-integration-workbench.git
cd tracezilla-integration-workbench
```

On the first run, create `.env`, install dependencies, build the assets, and
start the application services:

```bash
cp .env.example .env
docker compose build app
docker compose run --rm --no-deps app composer install
docker compose run --rm --no-deps app npm install
docker compose run --rm --no-deps app npm run build
docker compose up -d db app
docker compose exec app php artisan key:generate
docker compose exec app php artisan migrate --force
```

Open [http://localhost:8000/](http://localhost:8000/). Stop it with:

```bash
docker compose down
```

## Configure credentials

Copy `.env.example` to `.env` and enter the Shopify and Tracezilla credentials
there. The application reads credentials from Laravel's service configuration;
it does not currently accept them through the browser.

After changing `.env`, reload Laravel configuration:

With Docker:

```bash
docker compose exec app php artisan config:clear
```

Without Docker:

```bash
php artisan config:clear
```

The web interface can validate each connection and retrieve small samples of
Shopify products and Tracezilla SKUs.

## Run integration commands from the terminal

Run these commands from the workbench repository or deployed Laravel project
directory. They use the same `.env` configuration as the web interface.

Confirm that the current directory contains the `artisan` file before running
them:

```bash
test -f artisan && echo "Laravel workbench detected"
```

### With Docker

```bash
# List Shopify locations
docker compose exec app php artisan shopify:locations

# Compare Shopify and tracezilla catalogs
docker compose exec app php artisan pull-catalog-from-shopify --limit=10

# Preview creating tracezilla SKUs from Shopify
docker compose exec app php artisan tracezilla:skus-from-shopify --limit=10

# Preview updating Shopify inventory from tracezilla
docker compose exec app php artisan shopify:inventory-from-tracezilla --limit=10

# Preview creating individual tracezilla orders from Shopify
docker compose exec app php artisan tracezilla:orders-from-shopify --limit=10

# Preview collected Shopify orders
docker compose exec app php artisan pull-orders-from-shopify-collected --days=3 --limit=10

# Report tracezilla SKUs missing from Shopify
docker compose exec app php artisan push-catalog-to-shopify --limit=10
```

### Without Docker

Use this form when PHP and the application dependencies are installed directly
on the server, as with a traditional Laravel hosting account:

```bash
# List Shopify locations
php artisan shopify:locations

# Compare Shopify and tracezilla catalogs
php artisan pull-catalog-from-shopify --limit=10

# Preview creating tracezilla SKUs from Shopify
php artisan tracezilla:skus-from-shopify --limit=10

# Preview updating Shopify inventory from tracezilla
php artisan shopify:inventory-from-tracezilla --limit=10

# Preview creating individual tracezilla orders from Shopify
php artisan tracezilla:orders-from-shopify --limit=10

# Preview collected Shopify orders
php artisan pull-orders-from-shopify-collected --days=3 --limit=10

# Report tracezilla SKUs missing from Shopify
php artisan push-catalog-to-shopify --limit=10
```

The SKU creation, inventory synchronization, and individual-order commands are
dry runs by default. Review their output before adding `--execute`. Confirm that
`.env` points to the intended sandbox accounts before enabling any write.

## Preview or create Tracezilla SKUs

From the Tracezilla connection page, select **Import Shopify SKUs into
Tracezilla**.

The page:

- Requires both Shopify and Tracezilla configuration.
- Starts with **Dry run** enabled.
- Processes at most 10 Shopify variants by default.
- Skips SKU codes that already exist in Tracezilla.
- Reports missing and duplicate Shopify SKU codes without creating them.
- Shows a structured summary and per-item output.

Run the dry run first and review every proposed item. If dry run is disabled,
pressing **Run SKU import** opens a confirmation dialog. Tracezilla is updated
only after **OK** is selected.

{: .warning }
The current mapper is a demonstration. It uses the Shopify SKU code as both the
Tracezilla SKU code and global name, with `pcs`, `colli`, and weight/conversion
factors of `1.0`. Review these values against the customer's products and unit
model before executing an import.

## Credential safety

Use the workbench only on a trusted computer. The `.env` file is ignored by Git,
but still contains secrets on your local disk. Never commit or share it. The
included development server must not be exposed to the public internet.
