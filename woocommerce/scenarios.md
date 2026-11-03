---
title: Scenarios
layout: default
parent: WooCommerce
nav_order: 20
has_children: true
---

# Scenarios

{: .label .label-green }
Read only

A scenario is where your customer's business rules live. It is ordinary PHP in
the project's `custom/` folder, separate from the integration's own code, so
you can change it freely. You create a scenario from a template, adapt it to
the customer, test it, and run it, by hand or from cron.

## Templates

| Template | What it does |
|:--|:--|
| `confirm-credentials` | A hello world: reads the store status and one tracezilla SKU. The default. |
| [`compare-catalogs`](./scenarios/compare-catalogs.html) | Compares the WooCommerce catalog with the tracezilla SKUs |
| [`missing-in-woocommerce`](./scenarios/missing-in-woocommerce.html) | Lists tracezilla SKUs the shop does not sell, to decide on with the customer |

## Create a scenario

Give the scenario a name and choose a template. With Docker (Mac, Linux, and
Windows PowerShell):

```bash
docker compose run --rm php php bin/bifrost-connect scenario:create hello --template=confirm-credentials
```

Without Docker:

```bash
php bin/bifrost-connect scenario:create hello --template=confirm-credentials
```

The name becomes the scenario's folder, here
`custom/Scenarios/WooCommerce/Hello/`. Use lowercase letters, numbers, and
single hyphens. A scenario that already exists is never replaced.

The command creates three files. They are yours to change:

| File | What you do in it |
|:--|:--|
| `DataReader.php` | Read the data the rules need from WooCommerce and tracezilla |
| `BusinessRules.php` | Write the customer's decisions in plain PHP. It receives what `DataReader` read and never calls an API itself. |
| `BusinessRulesTest.php` | Write examples of those decisions as tests that run without either service |

## Test and run it

Run the tests first. They include the tests of every scenario:

```bash
docker compose run --rm php composer test
```

Then run the scenario against the real services:

```bash
docker compose run --rm php php bin/bifrost-connect scenario:run hello
```

Without Docker, leave out `docker compose run --rm php`. With Docker, Docker
and the sandbox must be running. The result is JSON:

```json
{
    "status": "ok",
    "woocommerce": {
        "credentials_valid": true,
        "store_url": "http://localhost:8080",
        "version": "11.2.1"
    },
    "tracezilla": {
        "credentials_valid": true
    },
    "message": "Hello! Both API credentials are working."
}
```

## Make it your own

1. Write the customer's rule in plain language first, for example: "Report
   every WooCommerce product whose SKU does not exist in tracezilla."
2. Start from the template closest to the rule.
3. In `DataReader.php`, read what the rule needs (see below).
4. Write the examples in `BusinessRulesTest.php`, then the rule in
   `BusinessRules.php`, until `composer test` passes.
5. Run the scenario against the sandbox and your development company.

Scenarios are read only. The readers cannot change WooCommerce or tracezilla,
and `BusinessRules.php` must not call an API.

## Read data in DataReader.php

`DataReader::read()` receives two read-only readers and returns an array, which
becomes the input of `BusinessRules::apply()`:

| Reader method | Returns |
|:--|:--|
| `$wooCommerce->get($path, $query)` | One WooCommerce REST response, for example `get('system_status')` |
| `$wooCommerce->getAll($path, $query)` | Every page of a WooCommerce list, for example `getAll('orders', ['status' => 'processing'])` |
| `$wooCommerce->catalog()` | Everything a customer can buy: simple products, and variations instead of their variable product, each with `sku`, `name`, `product_id`, `variation_id`, `status`, `manage_stock`, and `stock_quantity` |
| `$tracezilla->get($path, $query)` | One tracezilla response, for example `get('/skus', ['perPage' => 1])` |
| `$tracezilla->getAll($path, $query)` | Every page of a tracezilla list |
| `$tracezilla->skus()` | All SKUs, each with `id`, `sku_code`, `name`, `active`, `category` (for example `finished_good` or `raw_material`), and `allow_sales` |

WooCommerce paths are relative to `/wp-json/wc/v3/`; see the
[WooCommerce REST API](https://woocommerce.github.io/woocommerce-rest-api-docs/).
tracezilla paths are relative to `/api/v1/{team-slug}/`; see the
[tracezilla API](https://tech.tracezilla.com/docs/rest-api).

## Implementation

| Responsibility | Source |
|:--|:--|
| Commands | `bin/scenario-create`, `bin/scenario-run` |
| Templates | `templates/scenarios/<template>/` |
| Creating a scenario from a template | `src/Scenarios/ScenarioGenerator.php` |
| The read-only readers | `src/Scenarios/WooCommerceReader.php`, `src/Scenarios/TracezillaReader.php` |
| Running a scenario | `src/Scenarios/ScenarioRunner.php` |
