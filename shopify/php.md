---
title: PHP
layout: default
parent: Shopify
nav_order: 30
has_children: true
---

# Build a Shopify integration with PHP

{: .label .label-green }
Framework neutral

This guide continues with the project created in
[Getting Started](../getting-started.html). That project is the consultant's
runnable starting point for Shopify and tracezilla integration commands. It
uses PHP 8.3 and Composer without Laravel, Symfony, or another application
framework.

{: .important }
This BifrostConnect application communicates directly with Shopify and
tracezilla. Its entry points live under `bin/`, so workflows are executed
through one console command.

## BifrostConnect at a glance

The normal production driver is cron. Cron starts a bounded workflow; the
BifrostConnect reads from Shopify and tracezilla, applies the
customer's PHP business rules, performs approved writes, and returns a
structured result and exit status.

<pre class="mermaid">
flowchart TB
    Cron[Cron schedule] --> Command[Console command]
    Command --> Workflow[Headless PHP workflow]
    Workflow <--> Shopify[Shopify API]
    Workflow <--> Tracezilla[tracezilla API]
    Rules[Customer business rules in PHP] --> Workflow
    Workflow --> Result[Result, log, and exit code]
</pre>

The consultant owns the schedule and customer rules. The application owns API
authentication, pagination, mapping, safety checks, duplicate prevention, and
workflow results. Shopify and tracezilla remain unaware of cron and of each
other.

Webhooks can later request a faster run, but they do not replace cron. A
periodic reconciliation must still revisit a safe overlap window so a missed
webhook, temporary API failure, or partially completed run is recovered. Safe
retries depend on stable external references and idempotent workflow rules.

Read [Implement custom business logic](./php/custom-business-logic.html) for a
practical guide to adding a customer feature and operating it from cron.

The seven catalog, inventory, and order commands are documented as child pages
beneath PHP. Start with the read-only
[Compare Catalogs](./php/compare-catalogs.html) command to validate catalog
access and SKU matching.

## Continue from your project

Complete [Getting Started](../getting-started.html) before using the commands
on this page. Run them from the project folder you chose there. In command
examples, uppercase names such as `YOUR_SCENARIO_NAME` are placeholders that
you replace with your own value.

## Available commands

- [Compare Catalogs](./php/compare-catalogs.html) — read and compare Shopify
  variants and tracezilla SKUs without changing either system.
- [Report Shopify Product Decisions](./php/report-shopify-product-decisions.html)
  — list tracezilla SKUs that need an explicit Shopify product or variant
  decision.
- [Create tracezilla SKUs](./php/create-tracezilla-skus.html) — preview or
  create missing tracezilla SKUs from Shopify variants.
- [List Shopify Locations](./php/list-shopify-locations.html) — inspect the
  locations and IDs available to the configured Shopify app.
- [Synchronize Inventory](./php/synchronize-inventory.html) — preview or write
  Shopify inventory quantities calculated from a tracezilla warehouse.
- [Report Collected Orders](./php/report-collected-orders.html) — summarize
  recent Shopify sales by business date, currency, and SKU without writing.
- [Import Individual Orders](./php/import-individual-orders.html) — preview or
  create one tracezilla sales order for each selected Shopify order.

## Run the tests

```bash
docker compose exec integration composer test
```

Tests use in-memory data and do not contact either API. Run them before and
after changing a mapper, service, or workflow.

## Architecture

The example separates API details from business behavior:

<pre class="mermaid">
flowchart TB
    subgraph Shopify[Shopify boundary]
        ShopifyQuery[GraphQL query] --> ShopifyService[Catalog service]
        ShopifyClient[API client] --> ShopifyService
        ShopifyService --> ShopifyMapper[Variant mapper]
    end

    subgraph Tracezilla[tracezilla boundary]
        TracezillaClient[API client] --> TracezillaService[Catalog service]
        TracezillaService --> TracezillaMapper[SKU mapper]
    end

    ShopifyMapper --> SharedModel[Shared CatalogItem]
    TracezillaMapper --> SharedModel
    SharedModel --> Workflow[CompareCatalogs workflow]
    Workflow --> Result[CatalogComparisonResult]
    Result --> Table[Table output]
    Result --> Json[JSON output]
</pre>

| Layer | Location | Responsibility |
|---|---|---|
| Entry points | `bin/` | Read CLI options, construct dependencies, run one workflow, select output, and return an exit code |
| Configuration | `src/Configuration.php` | Validates environment variables and normalizes endpoint configuration |
| Queries | `src/Shopify/Queries/` | Stores GraphQL documents separately from HTTP and business logic |
| Clients | `src/Shopify/ShopifyClient.php`, `src/Tracezilla/TracezillaClient.php` | Own authentication, URLs, headers, HTTP transport, JSON decoding, timeouts, and safe API errors |
| Services | `ShopifyCatalogService`, `TracezillaCatalogService` | Retrieve use-case data, follow pagination, and pass individual API records to mappers |
| Mappers | `src/*/Mappers/` | Convert service-specific API records into shared application models |
| Shared models | `src/Shared/` | Represent concepts used by both systems; `CatalogItem` contains the normalized SKU used for comparison |
| Workflows | `src/Workflows/` | Apply business rules without knowing about HTTP, GraphQL, Docker, or `.env` |
| Result and output | `CatalogComparisonResult`, `src/Output/` | Keep workflow results structured and render them for humans or automation |
| Tests | `tests/Unit/` | Verify mapping, comparison, and output without live credentials |

### Clients

A client is the lowest service-specific boundary. `ShopifyClient` obtains a
client-credentials access token and executes GraphQL requests.
`TracezillaClient` supplies the tracezilla team URL and bearer token. Clients
should not decide how products, inventory, or orders map between systems.

### GraphQL queries

`GetProductVariants` contains only the query document. Keeping queries in
named classes makes requested fields and Shopify API changes easy to review.
The catalog service supplies pagination variables and interprets the returned
connection.

### Services

Services answer an application-oriented question such as “read the Shopify
catalog.” They coordinate clients, pagination, validation, and mappers. A
service returns useful application objects rather than exposing a raw HTTP
response to the workflow.

### Mappers

Shopify and tracezilla describe catalog records differently. Each system has
its own mapper, but both produce `CatalogItem`. The workflow therefore compares
SKU codes without depending on Shopify GraphQL fields or tracezilla response
keys. Customer-specific transformation rules belong in explicit mappers, not
in clients or entry-point scripts.

### Workflows

A workflow coordinates application behavior. `CompareCatalogs` accepts two
objects implementing `CatalogReader`, reads them, indexes normalized items by
SKU, and returns structured differences. Constructor dependencies make it easy
to reuse in a framework and replace live readers with fakes during tests.

## Create another scenario

Generate a platform-specific starting point:

```bash
php bin/bifrost-connect scenario:create YOUR_SCENARIO_NAME --platform=shopify
```

Edit the generated GraphQL request, tracezilla request, business rules, and
test. Authentication, locking, retries, history, and console execution remain
framework responsibilities. See
[Implement Custom Business Logic](./php/custom-business-logic.html) for the
complete workflow.
