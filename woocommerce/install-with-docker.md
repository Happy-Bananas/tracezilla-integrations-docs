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

- Docker Desktop or Docker Engine with the Compose plugin
- Git
- [WooCommerce prerequisites](./setup.html): a development store and REST API
  keys
- [tracezilla prerequisites](../tracezilla-fundamentals.html): a development
  company and an OAuth client

You do not need a GitHub account to download or run the project.

## 1. Get the project

Run the following from the directory where you keep development projects.
Replace `YOUR_PROJECT_NAME` with the folder name you want:

```bash
git clone https://github.com/Happy-Bananas/tracezilla-woocommerce-php.git YOUR_PROJECT_NAME
cd YOUR_PROJECT_NAME
git remote rename origin template
```

The public source is named `template` in Git so it cannot be confused with a
future private customer repository. If you later want backup, collaboration,
or deployment through Git, push the project to a **private repository owned by
you or the customer**.

## 2. Add credentials

Create `.env` from the included template:

```bash
cp .env.example .env
```

Open `.env` in your editor and add the WooCommerce and tracezilla values:

```dotenv
WOOCOMMERCE_URL=http://host.docker.internal:8080
WOOCOMMERCE_CONSUMER_KEY=ck_...
WOOCOMMERCE_CONSUMER_SECRET=cs_...

TRACEZILLA_TEAM_SLUG=your-development-company
TRACEZILLA_CLIENT_ID=...
TRACEZILLA_CLIENT_SECRET=...
```

`host.docker.internal:8080` reaches the local sandbox from inside the
container. For an existing store, use its public HTTPS URL. Git ignores `.env`;
never commit it.

## 3. Install the PHP packages

```bash
docker compose run --rm php composer install
```

The first run builds the development container, so it can take a little
while. The container runs as your user, so the files it creates belong to you.
If your user and group IDs are not 1000, set `UID` and `GID` in your shell
first.

## 4. Log in to tracezilla

```bash
docker compose run --rm php php bin/tracezilla-login
```

Enter your tracezilla username and password. The password is used once and
never stored; see [Credentials](../fundamentals/account-and-team.html#oauth-client-and-login-woocommerce).

## 5. Check both connections

```bash
./check-connection
```

The command performs one small read-only request against each service. A
successful result looks like:

```text
Connection check passed.
WooCommerce: connected to http://localhost:8080 (WooCommerce 11.2.1)
tracezilla:  connected to team your-development-company
```

No products, orders, inventory, or settings are changed. Nothing else works
until both lines report `connected`; see [Check Connections](./check-connections.html)
for what to do when one of them fails.

## Next

Fill your development company with the [standard test dataset](../fundamentals/seed.html).
