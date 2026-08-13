---
title: Create tracezilla SKUs
layout: default
parent: PHP
grand_parent: Shopify
nav_order: 20
---

# Create tracezilla SKUs from Shopify

{: .label .label-yellow }
Writes with explicit confirmation

This command reads Shopify product variants and the existing tracezilla SKU
catalog. It previews or creates a tracezilla SKU for each selected Shopify SKU
code that does not already exist.

Complete the [PHP installation and configuration](../php.html) first. Use a
development Shopify store and a test tracezilla team while adapting the example.

## Review the example mapping

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

## Run a dry run

Dry run is the default and processes at most ten Shopify variants:

```bash
docker compose run --rm php php bin/create-tracezilla-skus
```

Choose another processing limit or request JSON:

```bash
docker compose run --rm php php bin/create-tracezilla-skus --limit=25
docker compose run --rm php php bin/create-tracezilla-skus --limit=25 --json
```

Unlike the display limit in Compare Catalogs, this limit controls how many
Shopify variants are processed. The command still reads the complete Shopify
and tracezilla catalogs so it can report source counts and avoid duplicates.

## Understand the decisions

Each selected variant receives one result:

| Status | Meaning |
|---|---|
| `would_create` | The SKU is missing in tracezilla and would be created during execution |
| `created` | tracezilla accepted the create request |
| `skipped` | The SKU already exists or another selected Shopify variant has the same SKU |
| `invalid` | The Shopify variant has no SKU |
| `failed` | The tracezilla create request failed |

Matching uses the trimmed SKU code. Product names and internal IDs do not
determine equality. Re-running the command reads the tracezilla catalog again,
so successfully created SKU codes are skipped on later runs.

## Execute a bounded write

First inspect a dry run. For the first sandbox write, use a limit of one:

```bash
docker compose run --rm php php bin/create-tracezilla-skus --execute --confirm --limit=1
```

Both `--execute` and `--confirm` are required. Supplying `--execute` alone exits
with an error and writes nothing. Increase the limit only after verifying the
created tracezilla record and its unit mapping.

The command requires Shopify `read_products` access and permission to list and
create tracezilla SKUs. It never writes to Shopify.

## Architecture

<pre class="mermaid">
flowchart TB
    Query[GetProductVariants query] --> ShopifyService[ShopifyCatalogService]
    ShopifyClient[ShopifyClient] --> ShopifyService
    ShopifyService --> Variant[ShopifyVariantData]
    TracezillaClient[TracezillaClient] --> TracezillaService[TracezillaCatalogService]
    Variant --> Workflow[CreateTracezillaSkus workflow]
    TracezillaService --> Workflow
    Workflow --> Mapper[ShopifyVariantToTracezillaSkuMapper]
    Mapper --> Payload[TracezillaSkuData]
    Payload --> TracezillaService
    Workflow --> Result[CreateTracezillaSkusResult]
</pre>

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

## Test changes

```bash
docker compose run --rm php composer test
```

The tests use in-memory variants and a fake tracezilla gateway. They verify dry
run, limits, missing and duplicate SKUs, existing records, successful writes,
and isolated failures without contacting either API.
