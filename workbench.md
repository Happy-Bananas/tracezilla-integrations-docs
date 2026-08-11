---
layout: default
title: Integration workbench
nav_order: 4
---

# Integration workbench

The [Tracezilla Integration Workbench](https://github.com/Happy-Bananas/tracezilla-integration-workbench)
is a local browser application for checking credentials before you start an
integration project. It is separate from the language-specific examples and is
not a production connector.

## Run it locally

Clone the repository, then run:

```bash
docker compose up --build
```

Open [http://localhost:8000/](http://localhost:8000/). Stop it with:

```bash
docker compose down
```

## Initial checks

- Shopify: validate client credentials and retrieve up to 10 products or locations.
- Tracezilla: validate an API key and retrieve up to 10 SKUs.

All current checks are read-only. First validate credentials on the relevant
page; successful credentials are then available to the sample buttons for the
rest of that browser session.

## Credential safety

Use the workbench only on a trusted computer. Credentials are held in an
encrypted browser session cookie, expire after 60 minutes, and can be removed
immediately with **Forget credentials**. They are not written to `.env` or shown
again in password fields. The included development server must not be exposed
to the public internet.
