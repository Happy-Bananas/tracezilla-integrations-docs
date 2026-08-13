---
title: Compare Catalogs
layout: default
parent: Java
grand_parent: Shopify
nav_order: 10
---

# Compare Shopify and tracezilla catalogs

{: .label .label-green }
Read only

## Behavior

The command retrieves both complete catalogs and compares normalized SKU
codes. Complete the [Java setup](../java.html) first.

## Run the command

```bash
docker compose run --rm app
```

Change the displayed row limit or return complete JSON:

```bash
docker compose run --rm app --limit=25
docker compose run --rm app --json
```

SKU is the shared identifier. Results contain matches, Shopify-only SKUs, and
tracezilla-only SKUs. The limit affects display only, never catalog totals.

## Options

- `--limit=25` changes the maximum displayed rows per category.
- `--json` returns the complete structured result.

## Safety and exit status

The command requires Shopify `read_products` and tracezilla read access. It
writes no data. Differences return exit code `0`; configuration,
authentication, API, and malformed-response failures return non-zero.

## Architecture

<pre class="mermaid">
flowchart TB
    subgraph Shopify[Shopify boundary]
        Query[GraphQL query] --> ShopifyCatalog[Catalog reader]
        ShopifyClient[API client] --> ShopifyCatalog
    end
    subgraph Tracezilla[tracezilla boundary]
        TracezillaCatalog[Catalog reader]
    end
    ShopifyCatalog --> Model[Shared CatalogItem]
    TracezillaCatalog --> Model
    Model --> Workflow[CompareCatalogs workflow]
    Workflow --> Output[Table or JSON output]
</pre>

## Implementation

All Java paths below are relative to
`src/main/java/com/happybananas/tracezilla/`.

| Responsibility | Source |
|---|---|
| CLI and output | `Main.java` |
| Shopify query | `shopify/GetProductVariants.java` |
| Shopify API client | `shopify/ShopifyClient.java` |
| Shopify catalog reader | `shopify/ShopifyCatalog.java` |
| tracezilla catalog reader | `tracezilla/TracezillaCatalog.java` |
| Shared model and contract | `shared/CatalogItem.java`, `shared/CatalogReader.java` |
| Comparison | `workflow/CompareCatalogs.java` |

## Tests

```bash
docker compose run --rm --entrypoint mvn app test
```

The tests do not contact Shopify or tracezilla.
