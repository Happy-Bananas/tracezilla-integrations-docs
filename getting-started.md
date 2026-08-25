---
title: Getting Started
layout: default
nav_order: 10
---

# Getting started

The integration is one headless PHP console application:

<pre class="mermaid">
flowchart TB
    Shopify[Shopify]
    Integration[tracezilla Integration]
    Tracezilla[tracezilla]

    Shopify <--> Integration
    Integration <--> Tracezilla
</pre>

It runs manually during development and from cron in production. Customer
business rules are ordinary PHP files generated from a safe starting point.

> **The one-command setup below is optional.** It is the quickest way to get
> started, but it does not do anything you cannot do yourself. See
> [Manual installation](./manual-installation.html) if you prefer to run each
> setup step separately.

## Before you start

Install [Docker Desktop](https://www.docker.com/products/docker-desktop/) or
Docker Engine with the Compose plugin. You also need Git and `curl`.

Use credentials for a Shopify test store and a tracezilla test team while
developing an integration.

## 1. Create the project

```bash
curl -fsSL https://raw.githubusercontent.com/Happy-Bananas/tracezilla-shopify-php/main/create-shopify-project | sh -s -- my-shopify-integration
```

`my-shopify-integration` is simply the name of the new project folder. Choose a
name that describes your customer or integration, for example:

```bash
curl -fsSL https://raw.githubusercontent.com/Happy-Bananas/tracezilla-shopify-php/main/create-shopify-project | sh -s -- happy-hanna-shopify
```

The command downloads a fresh copy of the integration into that folder and
creates the `.env` configuration file. It will not replace a folder that
already exists.

You do not need a GitHub account to create or run the integration. Everything
runs from the new folder on your computer. If you later want backup,
collaboration, or deployment through Git, push the project to a **private
repository owned by you or the customer**. Customer business rules should not
be pushed to the public Happy Bananas repository.

## 2. Add credentials

```bash
cd my-shopify-integration
```

Open `.env` in your editor and complete the Shopify and tracezilla values.
Never commit this file; Git ignores it automatically.

## 3. Check both connections

```bash
./check-connection
```

The first run builds and starts the development container, so it can take a
little while. The command prints progress, waits until the integration is
ready, and then performs one small read-only request against each service.

A successful result looks like:

```text
Connection check passed.
Shopify: Example Shop (example-shop.myshopify.com)
tracezilla: connected
```

No products, orders, inventory, or settings are changed.

## Next

The connections work and the development environment is ready. Continue with
[Create Custom Business Logic](./shopify/php/custom-business-logic.html).
