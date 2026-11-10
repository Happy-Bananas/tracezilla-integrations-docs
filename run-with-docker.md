---
title: Set Up with Docker
layout: default
parent: Shopify
nav_order: 3
---

# Set up your Shopify integration with Docker

This is the quickest way to create and run a Shopify integration with
BifrostConnect. See [Set up without Docker](./manual-installation.html) if you prefer
to use PHP and Composer directly on your computer.

## Requirements

- [Docker](./development-environment.html#with-docker); nothing else on your computer
- [Shopify prerequisites](./shopify/setup.html): a development store and an
  installed app
- [tracezilla prerequisites](./tracezilla-fundamentals.html): a development
  company and an OAuth client

You do not need a GitHub account to download or run the project.

## 1. Create the project

Run the following from the folder where you keep your projects. Replace
`YOUR_INTEGRATION_NAME` with the name of your integration:

Mac and Linux:

```bash
curl -fsSL https://happy-bananas.github.io/tracezilla-integrations-docs/create-project | sh -s -- shopify YOUR_INTEGRATION_NAME
```

Windows (PowerShell):

```powershell
docker run --rm -v "${PWD}:/work" -w /work alpine:3 sh -c "wget -qO- https://happy-bananas.github.io/tracezilla-integrations-docs/create-project | sh -s -- shopify YOUR_INTEGRATION_NAME"
```

The command downloads a fresh copy of the BifrostConnect Shopify template into
that folder and
creates the `.env` configuration file. The project is a plain folder without
Git history; see [Create the project](./getting-started.html#4-create-the-project).
It will not replace a folder that already exists.

The project includes the tracezilla seed, which fills a development company
with test data; see [Seed the Test Dataset](./fundamentals/seed.html). If you
will work directly on the customer's production data, add `--without-seed`
after the project name to leave it out.

## 2. Add credentials

```bash
cd YOUR_INTEGRATION_NAME
```

Open `.env` in your editor and complete the Shopify and tracezilla values. The
tracezilla values are the team slug and the four Resource Owner Password
Credentials (`TRACEZILLA_CLIENT_ID`, `TRACEZILLA_CLIENT_SECRET`,
`TRACEZILLA_USERNAME`, and `TRACEZILLA_PASSWORD`); see
[Credentials](./fundamentals/account-and-team.html).
Never commit this file; Git ignores it automatically.

## 3. Install the PHP packages

```bash
docker compose run --rm php composer install
```

The first run builds the development container, so it can take a little
while.

## 4. Check both connections

Run the helper from the new project folder. It is part of the project, so it
does not exist before step 1:

Mac and Linux:

```bash
./check-connection
```

The helper installs the PHP packages if needed and then performs one small
read-only request against each service.

Windows (PowerShell):

```powershell
docker compose run --rm php php bin/bifrost-connect connection:check
```

A successful result looks like:

```text
Connection check passed.
Shopify: Example Shop (example-shop.myshopify.com)
tracezilla: connected
```

No products, orders, inventory, or settings are changed. Nothing else works
until both connections pass; see [Validate Connection](./shopify/setup/connection-validation.html)
if one of them fails.

## Next

The connections work and the development environment is ready. Fill your
development company with the [standard test dataset](./fundamentals/seed.html),
then continue with [Implement Custom Business Logic](./shopify/custom-business-logic.html).
