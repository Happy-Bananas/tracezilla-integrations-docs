---
title: Compare Catalogs
layout: default
parent: PHP
grand_parent: Shopify
nav_order: 10
---

# Compare Shopify and tracezilla catalogs

{: .label .label-green }
Read only

## Behavior

The command retrieves the
complete Shopify variant catalog and tracezilla SKU catalog, normalizes their
SKU codes, and reports differences without writing to either API.

Complete the [PHP installation and configuration](../php.html) before running
this command.

## Run the command

From the cloned `tracezilla-integration-php` repository, run:

### With Docker

```bash
docker compose exec integration php bin/tracezilla-integration catalog:compare
```

### Without Docker

```bash
php bin/compare-catalogs
```

The terminal output contains three categories:

- SKUs present in both systems;
- SKUs present only in Shopify;
- SKUs present only in tracezilla.

SKU is the shared identifier. Product titles, variant names, and internal IDs
do not determine a match.

The output begins by recording the validated application command and its
execution mode:

```text
Command: php bin/compare-catalogs
Mode: READ ONLY
```

This makes saved terminal and future cron logs self-describing. The displayed
command is the PHP entry point; when Docker is used, the surrounding
`docker compose run` invocation remains the responsibility of the shell.

## Options

Display a different maximum number of rows from each category:

```bash
docker compose exec integration php bin/tracezilla-integration catalog:compare --limit=25
```

Return the complete result as machine-readable JSON:

```bash
docker compose exec integration php bin/tracezilla-integration catalog:compare --json
```

JSON output remains valid JSON and includes the same execution context around
the structured workflow result:

```json
{
  "command": "php bin/compare-catalogs --json",
  "mode": "read_only",
  "result": {
    "status": "differences"
  }
}
```

The row limit affects only the terminal display. The comparison and summary
counts always use the complete catalogs.

## Safety and exit status

The command requires Shopify `read_products` access and read access to the
configured tracezilla team. It does not create or update data.

Because the command cannot write, it has no `--execute` or `--confirm` flags.
Commands that can change external data will instead default to `dry_run` and
require both flags before reporting `execute` mode.

Catalog differences are a successful result and return exit code `0`.
Configuration, authentication, transport, or malformed-response errors return
a non-zero exit code.

## Architecture

<pre class="mermaid">
flowchart TB
    subgraph Shopify[Shopify boundary]
        Query[GraphQL query] --> ShopifyService[Catalog service]
        ShopifyClient[API client] --> ShopifyService
        ShopifyService --> ShopifyMapper[Variant mapper]
    end
    subgraph Tracezilla[tracezilla boundary]
        TracezillaClient[API client] --> TracezillaService[Catalog service]
        TracezillaService --> TracezillaMapper[SKU mapper]
    end
    ShopifyMapper --> Model[Shared CatalogItem]
    TracezillaMapper --> Model
    Model --> Workflow[CompareCatalogs workflow]
    Workflow --> Result[CatalogComparisonResult]
</pre>

## Implementation

| Responsibility | Source |
|---|---|
| CLI entry point and options | `bin/compare-catalogs` |
| Shopify GraphQL document | `src/Shopify/Queries/GetProductVariants.php` |
| Shopify catalog pagination and mapping | `src/Shopify/ShopifyCatalogService.php` |
| tracezilla catalog retrieval and mapping | `src/Tracezilla/TracezillaCatalogService.php` |
| Comparison behavior | `src/Workflows/CompareCatalogs.php` |
| Structured result | `src/Shared/CatalogComparisonResult.php` |
| Terminal and JSON rendering | `src/Output/` |

This division is intentional: the entry point assembles the command, clients
handle APIs, services retrieve use-case data, mappers normalize records, and
the workflow contains the comparison rule. See the [PHP architecture guide](../php.html#architecture)
before adapting the command.

## Tests

```bash
docker compose exec integration composer test
```

The tests use in-memory data and do not contact Shopify or tracezilla.
