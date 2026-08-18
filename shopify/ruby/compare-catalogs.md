---
title: Compare Catalogs
layout: default
parent: Ruby
grand_parent: Shopify Labs
nav_order: 10
---

# Compare Shopify and tracezilla catalogs

{: .label .label-yellow }
Experimental

{: .label .label-green }
Read only

## Behavior

The command retrieves the complete Shopify variant catalog and tracezilla SKU
catalog, normalizes their SKU codes, and reports differences without writing to
either API.

Complete the [Ruby installation and configuration](../ruby.html) before
running this command.

## Run the command

From the cloned `tracezilla-shopify-ruby` repository, run:

### With Docker

```bash
docker compose run --rm app
```

### Without Docker

```bash
bundle exec ruby bin/compare_catalogs
```

The terminal output contains three categories:

- SKUs present in both systems;
- SKUs present only in Shopify;
- SKUs present only in tracezilla.

SKU is the shared identifier. Product titles, variant names, and internal IDs
do not determine a match.

## Options

Display a different maximum number of rows from each category:

```bash
docker compose run --rm app --limit=25
```

Return the complete result as machine-readable JSON:

```bash
docker compose run --rm app --json
```

The row limit affects only terminal display. Comparison and summary counts use
the complete catalogs.

## Safety and exit status

The command requires Shopify `read_products` and tracezilla read access and
never writes data. Differences return exit code `0`; configuration,
authentication, API, and malformed-response errors return non-zero.

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
    Workflow --> Output[Table or JSON output]
</pre>

## Implementation

| Responsibility | Source |
|---|---|
| CLI | `bin/compare_catalogs` |
| Shopify query and boundary | `lib/tracezilla_shopify/shopify/` |
| tracezilla boundary | `lib/tracezilla_shopify/tracezilla/` |
| Shared model | `lib/tracezilla_shopify/catalog_item.rb` |
| Comparison | `lib/tracezilla_shopify/compare_catalogs.rb` |
| Output | `lib/tracezilla_shopify/table_renderer.rb` |

The entry point assembles the command, clients handle APIs, services retrieve
use-case data, mappers normalize records, and the workflow owns comparison.
See the [Ruby architecture guide](../ruby.html#architecture) before adapting
the command.

## Tests

```bash
docker compose run --rm --entrypoint bundle app exec rake test
```

The tests use local fixtures and do not contact Shopify or tracezilla.
