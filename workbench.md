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

Clone the repository, then run:

```bash
docker compose up --build
```

Open [http://localhost:8000/](http://localhost:8000/). Stop it with:

```bash
docker compose down
```

## Configure credentials

Copy `.env.example` to `.env` and enter the Shopify and Tracezilla credentials
there. The application reads credentials from Laravel's service configuration;
it does not currently accept them through the browser.

The web interface can validate each connection and retrieve small samples of
Shopify products and Tracezilla SKUs.

## Credential safety

Use the workbench only on a trusted computer. The `.env` file is ignored by Git,
but still contains secrets on your local disk. Never commit or share it. The
included development server must not be exposed to the public internet.
