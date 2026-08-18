---
title: Create tracezilla SKUs
layout: default
parent: C# / .NET
grand_parent: Shopify Labs
nav_order: 20
---

# Create tracezilla SKUs from Shopify

{: .label .label-yellow }
Experimental

{: .label .label-yellow }
Writes with explicit confirmation

## Behavior

The command reads Shopify variants and existing tracezilla SKUs. It previews or
creates a tracezilla SKU for each selected Shopify SKU code missing from
tracezilla.

Complete the [C# / .NET installation and configuration](../dotnet.html) first.
Use development and test accounts while adapting the example.

### Mapping

The mapping is deliberately visible in
`src/TracezillaShopify/Workflows/CreateTracezillaSkus.cs`:

| Shopify source | tracezilla field | Example value |
|---|---|---|
| Variant SKU | `sku_code` | Shopify SKU |
| Variant SKU | `global_name` | Shopify SKU |
| Example assumption | `weight_factor_net` | `1.0` |
| Example assumption | `weight_factor_gross` | `1.0` |
| Example assumption | `unit_of_measure` | `pcs` |
| Example assumption | `lot_unit` | `colli` |
| Example assumption | `default_uom_conversion` | `1.0` |

The fixed values are customer-specific example assumptions, not defaults.
Review the customer's units, weights, and conversion rules before writing.

Each variant becomes `would_create`, `created`, `skipped`, `invalid`, or
`failed`. Existing tracezilla SKUs and duplicate Shopify SKUs are skipped;
variants without SKUs are invalid. Matching uses trimmed SKU code only.

## Run the command

Dry run is the default and processes at most ten variants:

### With Docker

```bash
docker compose run --rm app create-tracezilla-skus --limit=10
```

### Without Docker

```bash
set -a
source .env
set +a
dotnet run --project src/TracezillaShopify -- create-tracezilla-skus --limit=10
```

## Options

Choose another processing limit or request structured JSON:

```bash
docker compose run --rm app create-tracezilla-skus --limit=25
docker compose run --rm app create-tracezilla-skus --limit=25 --json
```

The limit bounds processing. Complete catalogs are still read for counts and
duplicate protection.

## Safety and exit status

After a dry run, perform at most one sandbox write:

```bash
docker compose run --rm app create-tracezilla-skus --execute --confirm --limit=1
```

Both flags are required. `--execute` alone exits without writing. Inspect the
created record and unit mapping before increasing the limit.

The command requires Shopify `read_products` and tracezilla list/create SKU
permission. It never writes Shopify data. Runs without failed writes return
`0`; option, configuration, API, and individual write failures return non-zero.
Later runs skip successfully created SKU codes.

## Architecture

<pre class="mermaid">
flowchart TB
    subgraph Shopify[Shopify boundary]
        Query[GraphQL query] --> ShopifyService[Catalog service]
        ShopifyClient[API client] --> ShopifyService
        ShopifyService --> Variant[Shopify variant data]
    end
    subgraph Tracezilla[tracezilla boundary]
        TracezillaClient[API client] --> TracezillaService[SKU service]
        Payload[tracezilla SKU payload] --> TracezillaService
    end
    Variant --> Workflow[CreateTracezillaSkus workflow]
    TracezillaService --> Workflow
    Workflow --> Payload
    Workflow --> Result[Structured summary and items]
</pre>

## Implementation

| Responsibility | Source |
|---|---|
| CLI safety, options, composition, and JSON | `src/TracezillaShopify/Program.cs` |
| Shopify variant retrieval | `src/TracezillaShopify/Shopify/ShopifyCatalog.cs` |
| Customer-specific mapping and decisions | `src/TracezillaShopify/Workflows/CreateTracezillaSkus.cs` |
| Existing-SKU lookup and creation | `src/TracezillaShopify/Tracezilla/TracezillaCatalog.cs` |
| Authenticated HTTP writes | `src/TracezillaShopify/Tracezilla/TracezillaClient.cs` |
| Structured summary and items | `src/TracezillaShopify/Workflows/CreateTracezillaSkus.cs` |

The workflow keeps the example business mapping explicit beside the payload,
without a generic mapping configuration layer.

## Tests

```bash
docker compose run --rm --entrypoint dotnet app test tests/TracezillaShopify.Tests --no-restore
```

Tests and compilation run without live APIs. Verify configured accounts with a
sandbox dry run before any bounded write.
