---
title: Install with Docker
layout: default
parent: WooCommerce
nav_order: 3
---

# Install the WooCommerce integration with Docker

This is the quickest way to install and run the WooCommerce integration. See
[Install without Docker](./install-without-docker.html) if you prefer to use PHP
and Composer directly on your computer.

## Requirements

- [Docker](../development-environment.html#with-docker); nothing else on your computer
- [WooCommerce prerequisites](./setup.html): a development store and REST API
  keys
- [tracezilla prerequisites](../tracezilla-fundamentals.html): a development
  company and an OAuth client

You do not need a GitHub account to download or run the project.

## 1. Create the project

Run the following from the folder where you keep your projects. Replace
`YOUR_INTEGRATION_NAME` with the name of your integration:

Mac and Linux:

```bash
curl -fsSL https://happy-bananas.github.io/tracezilla-integrations-docs/create-project | sh -s -- woocommerce YOUR_INTEGRATION_NAME
cd YOUR_INTEGRATION_NAME
```

Windows (PowerShell):

```powershell
docker run --rm -v "${PWD}:/work" -w /work alpine:3 sh -c "wget -qO- https://happy-bananas.github.io/tracezilla-integrations-docs/create-project | sh -s -- woocommerce YOUR_INTEGRATION_NAME"
cd YOUR_INTEGRATION_NAME
```

The command creates a plain project folder with its own `.env` file, without
Git history; see [Create the project](../getting-started.html#4-create-the-project).

Add `--with-seed` after the project name to include the tracezilla seed,
which fills your development company with test data; see
[Seed the Test Dataset](../fundamentals/seed.html).

## 2. Add credentials

Open `.env` in your editor. The WooCommerce values already point to the local
sandbox and its [preinstalled key](./setup/rest-api-keys.html); add the
tracezilla values:

```dotenv
WOOCOMMERCE_URL=http://host.docker.internal:8080
WOOCOMMERCE_CONSUMER_KEY=ck_sandbox
WOOCOMMERCE_CONSUMER_SECRET=cs_sandbox

TRACEZILLA_TEAM_SLUG=your-development-company
TRACEZILLA_CLIENT_ID=...
TRACEZILLA_CLIENT_SECRET=...
```

`host.docker.internal:8080` reaches the local sandbox from inside the
container. For an existing store, use its HTTPS URL and its own keys. Git
ignores `.env`; never commit it.

## 3. Install the PHP packages

```bash
docker compose run --rm php composer install
```

The first run builds the development container, so it can take a little
while. The container runs as your user, so the files it creates belong to you:
the create command stored your user and group IDs in `.env` as
`TRACEZILLA_DOCKER_UID` and `TRACEZILLA_DOCKER_GID`. Docker Compose names the
containers after the project folder, so several projects can exist side by
side.

## 4. Log in to tracezilla

```bash
docker compose run --rm php php bin/bifrost-connect tracezilla:login
```

Enter your tracezilla username and password. The password is used once and
never stored; see [Credentials](../fundamentals/account-and-team.html#oauth-client-and-login).

## 5. Check both connections

Make sure Docker and the sandbox are running, then follow
[Check Connections](./check-connections.html). Nothing else works until both
connections pass.

## Next

Fill your development company with the [standard test dataset](../fundamentals/seed.html).
