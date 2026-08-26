---
title: Run with Docker
layout: default
parent: Getting Started
nav_order: 10
---

# Run with Docker

This is the quickest way to create and run a BifrostConnect project. See
[Manual Installation](./manual-installation.html) if you prefer to use PHP and
Composer directly on your computer.

## Requirements

- Docker Desktop or Docker Engine with the Compose plugin
- Git
- `curl`
- [tracezilla and Shopify test credentials](./getting-started.html#prerequisites)

You do not need a GitHub account to download or run the project.

## 1. Create the project

```bash
curl -fsSL https://raw.githubusercontent.com/Happy-Bananas/tracezilla-shopify-php/main/create-shopify-project | sh -s -- YOUR_PROJECT_NAME
```

Replace `YOUR_PROJECT_NAME` with your project name, for example:

```bash
curl -fsSL https://raw.githubusercontent.com/Happy-Bananas/tracezilla-shopify-php/main/create-shopify-project | sh -s -- happy-hanna-shopify
```

The command downloads a fresh copy of BifrostConnect into that folder and
creates the `.env` configuration file. It will not replace a folder that
already exists.

Everything runs from the new folder on your computer. If you later want backup,
collaboration, or deployment through Git, push the project to a **private
repository owned by you or the customer**. Customer business rules should not
be pushed to the public Happy Bananas repository.

## 2. Add credentials

```bash
cd YOUR_PROJECT_NAME
```

Open `.env` in your editor and complete the Shopify and tracezilla values.
Never commit this file; Git ignores it automatically.

## 3. Check both connections

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

No products, orders, inventory, or settings are changed.

## Next

The connections work and the development environment is ready. Continue with
[Implement Custom Business Logic](./shopify/custom-business-logic.html).
