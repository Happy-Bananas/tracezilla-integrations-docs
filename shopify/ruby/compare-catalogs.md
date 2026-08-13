---
title: Compare Catalogs
layout: default
parent: Ruby
grand_parent: Shopify
nav_order: 10
---

# Compare Shopify and tracezilla catalogs

{: .label .label-green }
Read only

## Behavior

The command reads the complete catalogs and compares normalized SKU
codes. Complete the [Ruby setup](../ruby.html) first.

## Run the command

```bash
docker compose run --rm app
```

Change the displayed row limit or produce complete JSON:

```bash
docker compose run --rm app --limit=25
docker compose run --rm app --json
```

SKU is the matching identifier. The output groups matches, Shopify-only SKUs,
and tracezilla-only SKUs. The limit affects display, not comparison totals.

## Options

- `--limit=25` changes the maximum displayed rows per category.
- `--json` returns the complete structured result.

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

## Tests

```bash
docker compose run --rm --entrypoint bundle app exec rake test
```

The tests do not contact either API.
