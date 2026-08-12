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

```bash
docker compose exec app php artisan config:clear
```

The web interface can validate each connection and retrieve small samples of
Shopify products and Tracezilla SKUs.

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
