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

## 1. Clone

```bash
git clone https://github.com/Happy-Bananas/tracezilla-shopify-php.git tracezilla-integration-php
cd tracezilla-integration-php
cp .env.example .env
```

Add the Shopify test-store and tracezilla test-team credentials to `.env`.
Never commit this file.

## 2. Start

```bash
docker compose up --build
```

Wait until the terminal displays `TRACEZILLA INTEGRATION IS READY`. Keep it
running and open a second terminal in the same directory.

## 3. Generate the first scenario

```bash
docker compose exec integration php bin/tracezilla-integration \
  scenario:create confirm-credentials --platform=shopify
```

This creates four consultant-owned files under
`custom/Scenarios/Shopify/ConfirmCredentials/`: the Shopify request, the
tracezilla request, PHP business rules, and a test.

## 4. Test and run

```bash
docker compose exec integration composer test
docker compose exec integration php bin/tracezilla-integration \
  scenario:run confirm-credentials --platform=shopify
```

The generated scenario is read-only. A successful result confirms that both
API credentials work. You can now copy its shape for the customer's business
rule.

## Next

- [Implement Custom Business Logic](./shopify/php/custom-business-logic.html)
- [Deploy and schedule with cron](./deployment/)
- [Shopify setup](./shopify/setup.html)
