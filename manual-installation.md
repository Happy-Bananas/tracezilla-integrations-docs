---
title: Install without Docker
layout: default
nav_order: 4
parent: Shopify
---

# Install the Shopify integration without Docker

This is the native PHP alternative to [Install with Docker](./run-with-docker.html).
It covers obtaining the project
files and preparing the development environment. It does not explain how to
install system tools.

## Requirements

- PHP 8.3 or newer
- Composer 2
- `curl` and `tar`
- A terminal and code editor
- [Shopify prerequisites](./shopify/setup.html): a development store and an
  installed app
- [tracezilla prerequisites](./tracezilla-fundamentals.html): a development
  company and a personal access token

A GitHub account is not required to download or run the public project.

## Required files

The development project consists of:

- The integration source files, created with the create command.
- A local `.env` configuration created from `.env.example`.
- PHP packages installed by Composer into `vendor/`.

Do not copy `vendor/` from another computer and do not commit `.env`.

## 1. Create the project

Run the following from the folder where you keep your projects. Replace
`GiantPotatoes` with the name of your project:

```bash
curl -fsSL https://happy-bananas.github.io/tracezilla-integrations-docs/create-project | sh -s -- shopify GiantPotatoes
cd GiantPotatoes
```

The command creates a plain project folder with its own `.env` file, without
Git history; see [Create the project](./getting-started.html#3-create-the-project).

## 2. Install the PHP packages

```bash
composer install
```

Composer reads `composer.lock` and installs the tested package versions into
`vendor/`.

## 3. Complete the local configuration

Open `.env` in your editor and add the Shopify and tracezilla credentials. The
tracezilla values are the team slug and the personal access token
(`TRACEZILLA_API_KEY`); see [Credentials](./fundamentals/account-and-team.html).
Git ignores this file automatically.

## 4. Check the credentials

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

[Implement Custom Business Logic](./shopify/custom-business-logic.html).
