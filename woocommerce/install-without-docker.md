---
title: Install without Docker
layout: default
parent: WooCommerce
nav_order: 4
---

# Install the WooCommerce integration without Docker

This is the native PHP alternative to [Install with Docker](./install-with-docker.html).
It does not explain how to install system tools.

## Requirements

- PHP 8.3 or newer
- Composer 2
- Git
- [WooCommerce prerequisites](./setup.html): a development store and REST API
  keys
- [tracezilla prerequisites](../tracezilla-fundamentals.html): a development
  company and an OAuth client

## 1. Get the project

Run the following from the directory where you keep development projects.
Replace `YOUR_PROJECT_NAME` with the folder name you want:

```bash
git clone https://github.com/Happy-Bananas/tracezilla-woocommerce-php.git YOUR_PROJECT_NAME
cd YOUR_PROJECT_NAME
git remote rename origin template
```

The public source is named `template` in Git so it cannot be confused with a
future private customer repository.

## 2. Install the PHP packages

```bash
composer install
```

Composer reads `composer.lock` and installs the tested package versions into
`vendor/`. Do not copy `vendor/` from another computer.

## 3. Add credentials

```bash
cp .env.example .env
```

Open `.env` in your editor and add the WooCommerce and tracezilla values:

```dotenv
WOOCOMMERCE_URL=http://localhost:8080
WOOCOMMERCE_CONSUMER_KEY=ck_...
WOOCOMMERCE_CONSUMER_SECRET=cs_...

TRACEZILLA_TEAM_SLUG=your-development-company
TRACEZILLA_CLIENT_ID=...
TRACEZILLA_CLIENT_SECRET=...
```

Without Docker, the local sandbox is reached at `http://localhost:8080`. For an
existing store, use its public HTTPS URL. Git ignores `.env`; never commit it.

## 4. Log in to tracezilla

```bash
php bin/tracezilla-login
```

Enter your tracezilla username and password. The password is used once and
never stored; see [Credentials](../fundamentals/account-and-team.html#oauth-client-and-login-woocommerce).

## 5. Check both connections

```bash
php bin/check-connection
```

A successful result looks like:

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
