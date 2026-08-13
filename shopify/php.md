---
title: PHP
layout: default
parent: Shopify
nav_order: 30
---

# Build a Shopify integration with PHP

{: .label .label-green }
Framework neutral

The
[`tracezilla-shopify-php`](https://github.com/Happy-Bananas/tracezilla-shopify-php)
repository is a runnable starting point for consultants creating Shopify and
tracezilla integration commands. It uses PHP 8.3 and Composer without Laravel,
Symfony, or another application framework.

Its first command is the read-only [Compare Catalogs](./workflows/compare-catalogs.html)
workflow. The same organization is intended for later catalog, inventory, and
order commands.

## Clone and start the project

You need Git and Docker with the Docker Compose plugin. PHP and Composer do not
need to be installed on the host.

Clone only the PHP implementation:

```bash
git clone https://github.com/Happy-Bananas/tracezilla-shopify-php.git
cd tracezilla-shopify-php
```

Create the local configuration file:

```bash
cp .env.example .env
```

Complete [Shopify Setup](./setup.html) and
[tracezilla authentication](../fundamentals/authentication.html), then add the
test-store and test-team credentials to `.env`. Never commit this file.

Build the PHP container and install the locked Composer dependencies:

```bash
docker compose build
docker compose run --rm php composer install
```

The source directory is mounted into the container, so code changes are
available immediately. Rebuild only after changing the Dockerfile or container
dependencies.

## Execute Compare Catalogs

Run the default command:

```bash
docker compose run --rm php php bin/compare-catalogs
```

The command retrieves the complete Shopify variant catalog and the complete
tracezilla SKU catalog. It compares normalized SKU codes and prints at most ten
rows from each result category.

Change only the number of displayed rows:

```bash
docker compose run --rm php php bin/compare-catalogs --limit=25
```

Return complete machine-readable results:

```bash
docker compose run --rm php php bin/compare-catalogs --json
```

Catalog differences are a successful result and return exit code `0`.
Configuration, authentication, transport, or invalid-response errors return a
non-zero exit code. The command does not write to Shopify or tracezilla.

## Run the tests

```bash
docker compose run --rm php composer test
```

Tests use in-memory data and do not contact either API. Run them before and
after changing a mapper, service, or workflow.

## Architecture

The example separates API details from business behavior:

```text
Shopify GraphQL query ─┐
Shopify API client ────┼─> Shopify catalog service ─> Shopify mapper ─┐
                                                                     ├─> CompareCatalogs ─> result ─> output
tracezilla API client ───> tracezilla catalog service ─> SKU mapper ─┘
```

| Layer | Location | Responsibility |
|---|---|---|
| Entry point | `bin/compare-catalogs` | Reads CLI options, constructs dependencies, runs one workflow, selects output, and returns an exit code |
| Configuration | `src/Configuration.php` | Validates environment variables and normalizes endpoint configuration |
| Queries | `src/Shopify/Queries/` | Stores GraphQL documents separately from HTTP and business logic |
| Clients | `src/Shopify/ShopifyClient.php`, `src/Tracezilla/TracezillaClient.php` | Own authentication, URLs, headers, HTTP transport, JSON decoding, timeouts, and safe API errors |
| Services | `ShopifyCatalogService`, `TracezillaCatalogService` | Retrieve use-case data, follow pagination, and pass individual API records to mappers |
| Mappers | `src/*/Mappers/` | Convert service-specific API records into shared application models |
| Shared models | `src/Shared/` | Represent concepts used by both systems; `CatalogItem` contains the normalized SKU used for comparison |
| Workflow | `src/Workflows/CompareCatalogs.php` | Applies the business rule without knowing about HTTP, GraphQL, Docker, or `.env` |
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

## Create another command

Use the existing command as a template, but keep each responsibility in its
own layer:

1. Write down the read/write behavior, mapping rule, required scopes, and safe
   result categories.
2. Add or update a named GraphQL query when Shopify fields are required.
3. Add client behavior only when a new HTTP operation is needed.
4. Create service methods that handle pagination and return mapped models.
5. Add shared or workflow-specific models rather than passing raw arrays
   through the application.
6. Put transformation rules in mappers.
7. Implement a workflow that depends on interfaces or focused services.
8. Add a small file in `bin/` that constructs dependencies and invokes the
   workflow.
9. Add unit tests before running against test accounts.
10. For writes, default to dry run, display the intended changes, impose a
    limit, and require explicit confirmation.

Do not copy authentication, pagination, or mapping code into each command.
Extend the existing clients and services so later commands reuse the same
tested boundaries.

## Use the code in another application

The `src/` classes are ordinary Composer-autoloaded PHP. A consultant can reuse
them in Laravel, Symfony, WordPress, a queue worker, or a scheduled CLI. The
host application should replace only composition, configuration, and output;
the clients, services, mappers, and workflow rules can remain unchanged.

The optional
[`tracezilla-integration-workbench`](https://github.com/Happy-Bananas/tracezilla-integration-workbench)
is a Laravel interface for credential checks and controlled experiments. It is
not the canonical PHP template.
