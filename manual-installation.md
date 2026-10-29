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
- Git
- A terminal and code editor
- [Shopify prerequisites](./shopify/setup.html): a development store and an
  installed app
- [tracezilla prerequisites](./tracezilla-fundamentals.html): a development
  company and a personal access token

A GitHub account is not required to download or run the public project.

## Required files

The development project consists of:

- The integration source files from the public Git repository.
- A local `.env` configuration created from `.env.example`.
- PHP packages installed by Composer into `vendor/`.

Do not copy `vendor/` from another computer and do not commit `.env`.

## 1. Get the source files

Run the following from the directory where you keep development projects.
Replace `YOUR_PROJECT_NAME` with the folder name you want:

```bash
git clone https://github.com/Happy-Bananas/tracezilla-shopify-php.git YOUR_PROJECT_NAME
cd YOUR_PROJECT_NAME
git remote rename origin template
```

This downloads the application into the new folder. The public source is named
`template` in Git so it cannot be confused with a future private customer
repository.

## 2. Install the PHP packages

```bash
composer install
```

Composer reads `composer.lock` and installs the tested package versions into
`vendor/`.

## 3. Create the local configuration

Create `.env` from the included safe template:

```bash
cp .env.example .env
```

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
