---
title: Create tracezilla SKUs
layout: default
parent: Shopify
nav_order: 20
---

# Create tracezilla SKUs from Shopify

{: .label .label-yellow }
Writes with explicit confirmation

## Behavior

The command reads Shopify product variants and the existing tracezilla SKU
catalog. It previews or creates a tracezilla SKU for each selected Shopify SKU
code that does not already exist.

### Mapping

The mapping is deliberately visible in
`src/Tracezilla/Mappers/ShopifyVariantToTracezillaSkuMapper.php`:

| Shopify source | tracezilla field | Example value |
|---|---|---|
| Variant SKU | `sku_code` | Shopify SKU |
| Variant SKU | `global_name` | Shopify SKU |
| Example assumption | `weight_factor_net` | `1.0` |
| Example assumption | `weight_factor_gross` | `1.0` |
| Example assumption | `unit_of_measure` | `pcs` |
| Example assumption | `lot_unit` | `colli` |
| Example assumption | `default_uom_conversion` | `1.0` |

The fixed values are not universal defaults. They demonstrate where the
customer's product, unit, weight, and conversion rules belong. Review and edit
the mapper before enabling writes.

Each selected variant is reported as `would_create`, `created`, `skipped`,
`invalid`, or `failed`. Existing tracezilla SKU codes and repeated Shopify SKU
codes are skipped. Variants without an SKU are invalid. Matching uses the
trimmed SKU code, not product names or internal IDs.

## Run the command

Dry run is the default and processes at most ten Shopify variants:

### With Docker

```bash
docker compose exec integration php bin/bifrost-connect catalog:create-tracezilla-skus
```

### Without Docker

```bash
php bin/create-tracezilla-skus
```

## Options

Choose another processing limit or request complete JSON:

```bash
docker compose exec integration php bin/bifrost-connect catalog:create-tracezilla-skus --limit=25
docker compose exec integration php bin/bifrost-connect catalog:create-tracezilla-skus --limit=25 --json
```

Unlike the display limit in Compare Catalogs, this limit controls how many
Shopify variants are processed. The command still reads the complete Shopify
and tracezilla catalogs so it can report source counts and avoid duplicates.

## Safety and exit status

First inspect a dry run. For the first sandbox write, use a limit of one:

```bash
docker compose exec integration php bin/bifrost-connect catalog:create-tracezilla-skus --execute --confirm --limit=1
```

Both `--execute` and `--confirm` are required. Supplying `--execute` alone exits
with an error and writes nothing. Increase the limit only after verifying the
created tracezilla record and its unit mapping.

The command requires Shopify `read_products` access and permission to list and
create tracezilla SKUs. It never writes to Shopify.

Runs containing no failed writes return exit code `0`. Invalid options,
configuration or API errors, and results containing failed writes return a
non-zero exit code. Re-running the command reads tracezilla again, so
successfully created SKU codes are skipped on later runs.

## Architecture

<pre class="mermaid">
flowchart TB
    subgraph Shopify[Shopify boundary]
        Query[GraphQL query] --> ShopifyService[Catalog service]
        ShopifyClient[API client] --> ShopifyService
        ShopifyService --> Variant[ShopifyVariantData]
    end
    subgraph Tracezilla[tracezilla boundary]
        TracezillaClient[API client] --> TracezillaService[SKU service]
        Payload[TracezillaSkuData] --> TracezillaService
    end
    Variant --> Workflow[CreateTracezillaSkus workflow]
    TracezillaService --> Workflow
    Workflow --> Mapper[ShopifyVariantToTracezillaSkuMapper]
    Mapper --> Payload[TracezillaSkuData]
    Workflow --> Result[CreateTracezillaSkusResult]
</pre>

## Implementation

| Responsibility | Source |
|---|---|
| CLI safety, options, and composition | `bin/create-tracezilla-skus` |
| Shopify variant retrieval | `src/Shopify/ShopifyCatalogService.php` |
| Shopify input model | `src/Shopify/ShopifyVariantData.php` |
| Customer-specific mapping | `src/Tracezilla/Mappers/ShopifyVariantToTracezillaSkuMapper.php` |
| Validated tracezilla payload | `src/Tracezilla/TracezillaSkuData.php` |
| Existing-SKU lookup and creation | `src/Tracezilla/TracezillaCatalogService.php` |
| Decisions and duplicate protection | `src/Workflows/CreateTracezillaSkus.php` |
| Structured summary and items | `src/Workflows/CreateTracezillaSkusResult.php` |
| Terminal output | `src/Output/SkuCreationRenderer.php` |

The mapper is the intended customization point. There is no generic mapping
configuration or rules engine: the example keeps business assumptions explicit
and close to the code that creates the API payload.

## Tests

```bash
docker compose exec integration composer test
```

The tests use in-memory variants and a fake tracezilla gateway. They verify dry
run, limits, missing and duplicate SKUs, existing records, successful writes,
and isolated failures without contacting either API.
