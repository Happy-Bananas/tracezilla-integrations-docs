---
title: Compare Catalogs
layout: default
parent: Java
grand_parent: Experimental Implementations
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

Complete the [Java installation and configuration](../java.html) before
running this command.

## Run the command

From the cloned `tracezilla-shopify-java` repository, run:

### With Docker

```bash
docker compose run --rm app
```

### Without Docker

```bash
set -a
source .env
set +a
mvn package
java -jar target/tracezilla-shopify-java-0.1.0.jar
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

The entry point assembles the command, clients handle APIs, catalog readers
retrieve and normalize use-case data, and the workflow owns comparison. See
the [Java architecture guide](../java.html#architecture) before adapting the
command.

## Tests

```bash
docker compose run --rm --entrypoint mvn app test
```

The tests use local fixtures and do not contact Shopify or tracezilla.
