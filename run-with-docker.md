---
title: Install with Docker
layout: default
parent: Shopify
nav_order: 3
---

# Install the Shopify integration with Docker

This is the quickest way to create and run a BifrostConnect project for
Shopify. See [Install without Docker](./manual-installation.html) if you prefer
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

The command downloads a fresh copy of BifrostConnect into that folder and
creates the `.env` configuration file. The project is a plain folder without
Git history; see [Create the project](./getting-started.html#4-create-the-project).
It will not replace a folder that already exists.

## 2. Add credentials

```bash
cd YOUR_INTEGRATION_NAME
```

Open `.env` in your editor and complete the Shopify and tracezilla values. The
tracezilla values are the team slug and the OAuth client
(`TRACEZILLA_CLIENT_ID` and `TRACEZILLA_CLIENT_SECRET`); see
[Credentials](./fundamentals/account-and-team.html).
Never commit this file; Git ignores it automatically.

## 3. Log in to tracezilla

Install the PHP packages in the development container, then log in once:

```bash
docker compose run --rm integration composer install
docker compose run --rm integration php bin/bifrost-connect tracezilla:login
```

The first command builds the development container, so it can take a little
while. Enter your tracezilla username and password. The password is used once and
never stored; see [Credentials](./fundamentals/account-and-team.html#oauth-client-and-login).

## 4. Check both connections

Run the helper from the new project folder. It is part of the project, so it
does not exist before step 1:

Mac and Linux:

```bash
./check-connection
```

The first run builds and starts the development container, so it can take a
little while. The command prints progress, waits until BifrostConnect is ready,
and then performs one small read-only request against each service.

Windows (PowerShell): start the container, wait until
`docker compose logs integration` shows `BIFROSTCONNECT IS READY`, and then run
the check:

```powershell
docker compose up --build -d integration
docker compose logs -f integration
docker compose exec integration php bin/bifrost-connect connection:check
```

Press Ctrl+C to stop following the log once it reports that it is ready.

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

The connections work and the development environment is ready. Continue with
[Implement Custom Business Logic](./shopify/custom-business-logic.html).
