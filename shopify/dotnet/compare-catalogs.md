---
title: Compare Catalogs
layout: default
parent: C# / .NET
grand_parent: Shopify
nav_order: 10
---

# Compare Shopify and tracezilla catalogs

{: .label .label-green }
Read only

## Behavior

The command reads both complete catalogs and compares normalized SKU
codes. Complete the [C# / .NET setup](../dotnet.html) first.

## Run the command

```bash
docker compose run --rm app
```

Change the displayed limit or request complete JSON:

```bash
docker compose run --rm app --limit=25
docker compose run --rm app --json
```

Matches use SKU—not titles or internal IDs. The limit changes only displayed
rows; summary counts always use the complete catalogs.

## Options

- `--limit=25` changes the maximum displayed rows per category.
- `--json` returns the complete structured result.

## Safety and exit status

The command needs Shopify `read_products` and tracezilla read access. It does
not write data. Differences return exit code `0`; configuration,
authentication, API, and malformed-response failures return non-zero.

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
| CLI composition | `src/TracezillaShopify/Program.cs` |
| Shopify query | `src/TracezillaShopify/Shopify/GetProductVariants.cs` |
| Shopify client, service, mapper | `src/TracezillaShopify/Shopify/` |
| tracezilla client, service, mapper | `src/TracezillaShopify/Tracezilla/` |
| Shared model | `src/TracezillaShopify/Shared/CatalogItem.cs` |
| Comparison | `src/TracezillaShopify/Workflows/CompareCatalogs.cs` |
| Output | `src/TracezillaShopify/Output/TableRenderer.cs` |

## Tests

```bash
docker compose run --rm --entrypoint dotnet app test tests/TracezillaShopify.Tests --no-restore
```

The tests do not contact live APIs.
