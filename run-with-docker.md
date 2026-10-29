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

- Docker Desktop or Docker Engine with the Compose plugin
- `curl` and `tar`
- [Shopify prerequisites](./shopify/setup.html): a development store and an
  installed app
- [tracezilla prerequisites](./tracezilla-fundamentals.html): a development
  company and a personal access token

You do not need a GitHub account to download or run the project.

## 1. Create the project

Run the following from the folder where you keep your projects. Replace
`GiantPotatoes` with the name of your project:

```bash
curl -fsSL https://happy-bananas.github.io/tracezilla-integrations-docs/create-project | sh -s -- shopify GiantPotatoes
```

The command downloads a fresh copy of BifrostConnect into that folder and
creates the `.env` configuration file. The project is a plain folder without
Git history; see [Create the project](./getting-started.html#3-create-the-project).
It will not replace a folder that already exists.

## 2. Add credentials

```bash
cd GiantPotatoes
```

Open `.env` in your editor and complete the Shopify and tracezilla values. The
tracezilla values are the team slug and the personal access token
(`TRACEZILLA_API_KEY`); see [Credentials](./fundamentals/account-and-team.html).
Never commit this file; Git ignores it automatically.

## 3. Check both connections

Run the helper from the new project folder. It is part of the project, so it
does not exist before step 1:

```bash
./check-connection
```

The first run builds and starts the development container, so it can take a
little while. The command prints progress, waits until BifrostConnect is ready,
and then performs one small read-only request against each service.

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
