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

## Before you start

Install [Docker Desktop](https://www.docker.com/products/docker-desktop/) or
Docker Engine with the Compose plugin. You also need Git and `curl`.

Use credentials for a Shopify test store and a tracezilla test team while
developing an integration.

## 1. Create the project

```bash
curl -fsSL https://raw.githubusercontent.com/Happy-Bananas/tracezilla-shopify-php/main/create-shopify-project | sh -s -- my-shopify-integration
```

This creates `my-shopify-integration/`, copies the safe configuration template
to `.env`, and retains the source repository as a Git remote named `template`.
It refuses to overwrite an existing directory. The
[installer source](https://github.com/Happy-Bananas/tracezilla-shopify-php/blob/main/create-shopify-project)
is available for review before running it.

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

## 4. Generate the first business scenario

```bash
docker compose exec integration php bin/tracezilla-integration \
  scenario:create my-first-scenario --platform=shopify
```

This creates four consultant-owned files under
`custom/Scenarios/Shopify/MyFirstScenario/`: the Shopify request, the
tracezilla request, PHP business rules, and a test.

## 5. Test and run the scenario

```bash
docker compose exec integration composer test
docker compose exec integration php bin/tracezilla-integration \
  scenario:run my-first-scenario --platform=shopify
```

The generated scenario is read-only. Customize its four files for the
customer's business rule after the example passes unchanged.

## Next

- [Implement Custom Business Logic](./shopify/php/custom-business-logic.html)
- [Deploy and schedule with cron](./deployment/)
- [Shopify setup](./shopify/setup.html)
