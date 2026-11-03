---
title: Scenarios
layout: default
parent: WooCommerce
nav_order: 20
---

# Scenarios

{: .label .label-green }
Read only

A scenario is where the customer's own business rules live. It is ordinary PHP
in the project's `custom/` folder, separate from the framework, so you can
change it without touching the integration's own code.

## Create the hello world scenario

Start with the read-only "hello world". It reads the WooCommerce store's
status and one tracezilla SKU, and confirms that both sets of credentials work.

With Docker (Mac, Linux, and Windows PowerShell):

```bash
docker compose run --rm php php bin/bifrost-connect scenario:create confirm-credentials
```

Without Docker:

```bash
php bin/bifrost-connect scenario:create confirm-credentials
```

The command creates four files in
`custom/Scenarios/WooCommerce/ConfirmCredentials/`. They are yours to change:

| File | Purpose |
|:--|:--|
| `WooCommerceRequest.php` | The WooCommerce REST request to read, for example `system_status` or `products` |
| `TracezillaRequest.php` | The tracezilla request to read, for example `/skus` |
| `BusinessRules.php` | The customer's decisions, in plain PHP. It receives both responses and never calls an API itself. |
| `BusinessRulesTest.php` | Examples of those decisions that run without contacting either service |

Scenario names use lowercase letters, numbers, and single hyphens. A name that
already exists is never replaced.

## Test and run it

Run the tests first. They include the tests of every scenario:

```bash
docker compose run --rm php composer test
```

Then run the scenario against the real services:

```bash
docker compose run --rm php php bin/bifrost-connect scenario:run confirm-credentials
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
2. Change the two request files to read the data the rule needs. See the
   [WooCommerce REST API](https://woocommerce.github.io/woocommerce-rest-api-docs/)
   and the [tracezilla API](https://tech.tracezilla.com/docs/rest-api-v1/).
3. Write the examples in `BusinessRulesTest.php`, then the rule in
   `BusinessRules.php`, until `composer test` passes.
4. Run the scenario against the sandbox.

Scenarios are read only. Do not put calls that change WooCommerce or tracezilla
in `BusinessRules.php`.

## Implementation

| Responsibility | Source |
|:--|:--|
| Commands | `bin/scenario-create`, `bin/scenario-run` |
| Names and folders | `src/Scenarios/Scenario.php` |
| The generated files | `src/Scenarios/ScenarioGenerator.php` |
| Running a scenario | `src/Scenarios/ScenarioRunner.php` |
