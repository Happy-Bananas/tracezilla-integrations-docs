---
title: Install without Docker
layout: default
parent: WooCommerce
nav_order: 4
---

# Install the WooCommerce integration without Docker

This is the alternative to [Install with Docker](./install-with-docker.html).

## Requirements

Your computer must already have these installed and working. Installing and
configuring them is your responsibility and is not covered here; see
[Without Docker](../your-computer.html#without-docker) for links to the
official guides.

- PHP 8.3 or newer
- Composer 2
- [WooCommerce prerequisites](./setup.html): a development store and REST API
  keys
- [tracezilla prerequisites](../tracezilla-fundamentals.html): a development
  company and an OAuth client

## 1. Create the project

Run the following from the folder where you keep your projects. Replace
`GiantPotatoes` with the name of your project:

```bash
curl -fsSL https://happy-bananas.github.io/tracezilla-integrations-docs/create-project | sh -s -- woocommerce GiantPotatoes
cd GiantPotatoes
```

On Windows, download and extract the project as described in
[Windows without Docker](../getting-started.html#windows-without-docker).

The command creates a plain project folder with its own `.env` file, without
Git history; see [Create the project](../getting-started.html#4-create-the-project).

## 2. Download the libraries

```bash
composer install
```

This downloads the libraries the project uses into its `vendor/` folder.


## 3. Add credentials

Open `.env` in your editor. Set `WOOCOMMERCE_URL` as shown; the key is the
sandbox's [preinstalled key](./setup/rest-api-keys.html). Add the tracezilla
values:

```dotenv
WOOCOMMERCE_URL=http://localhost:8080
WOOCOMMERCE_CONSUMER_KEY=ck_sandbox
WOOCOMMERCE_CONSUMER_SECRET=cs_sandbox

TRACEZILLA_TEAM_SLUG=your-development-company
TRACEZILLA_CLIENT_ID=...
TRACEZILLA_CLIENT_SECRET=...
```

Without Docker, the local sandbox is reached at `http://localhost:8080`. For an
existing store, use its HTTPS URL and its own keys. Git ignores `.env`; never
commit it.

## 4. Log in to tracezilla

```bash
php bin/bifrost-connect tracezilla:login
```

Enter your tracezilla username and password. The password is used once and
never stored; see [Credentials](../fundamentals/account-and-team.html#oauth-client-and-login).

## 5. Check both connections

Make sure your WordPress store is running, then follow
[Check Connections](./check-connections.html). Nothing else works until both
connections pass.

## Next

Fill your development company with the [standard test dataset](../fundamentals/seed.html).
