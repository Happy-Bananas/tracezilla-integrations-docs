---
title: Set Up without Docker
layout: default
nav_order: 4
parent: Shopify
---

# Set up your Shopify integration without Docker

This is the alternative to [Set up with Docker](./run-with-docker.html).

## Requirements

Your computer must already have these installed and working. Installing and
configuring them is your responsibility and is not covered here; see
[Without Docker](./development-environment.html#without-docker) for links to the
official guides.

- PHP 8.3 or newer
- Composer 2
- [Shopify prerequisites](./shopify/setup.html): a development store and an
  installed app
- [tracezilla prerequisites](./tracezilla-fundamentals.html): a development
  company and an OAuth client

A GitHub account is not required to download or run the public project.

## 1. Create the project

Run the following from the folder where you keep your projects. Replace
`YOUR_INTEGRATION_NAME` with the name of your integration:

```bash
curl -fsSL https://happy-bananas.github.io/tracezilla-integrations-docs/create-project | sh -s -- shopify YOUR_INTEGRATION_NAME
cd YOUR_INTEGRATION_NAME
```

On Windows, download and extract the project as described in
[Windows without Docker](./getting-started.html#windows-without-docker).

The command creates a plain project folder with its own `.env` file, without
Git history; see [Create the project](./getting-started.html#4-create-the-project).

The project includes the tracezilla seed, which fills a development company
with test data; see [Seed the Test Dataset](./fundamentals/seed.html). If you
will work directly on the customer's production data, add `--without-seed`
after the project name to leave it out.

## 2. Download the libraries

```bash
composer install
```

This downloads the libraries the project uses into its `vendor/` folder.


## 3. Complete the local configuration

Open `.env` in your editor and add the Shopify and tracezilla credentials. The
tracezilla values are the team slug and the OAuth client
(`TRACEZILLA_CLIENT_ID` and `TRACEZILLA_CLIENT_SECRET`); see
[Credentials](./fundamentals/account-and-team.html).
Git ignores this file automatically.

## 4. Log in to tracezilla

```bash
php bin/bifrost-connect tracezilla:login
```

Enter your tracezilla username and password. The password is used once and
never stored; see [Credentials](./fundamentals/account-and-team.html#oauth-client-and-login).

## 5. Check the credentials

```bash
php bin/bifrost-connect connection:check
```

The command makes one small read-only request to each service. A successful
result looks like:

```text
Connection check passed.
Shopify: Example Shop (example-shop.myshopify.com)
tracezilla: connected
```

No products, orders, inventory, or settings are changed. Nothing else works
until both connections pass.

## Next

Fill your development company with the [standard test dataset](./fundamentals/seed.html),
then continue with [Implement Custom Business Logic](./shopify/custom-business-logic.html).
