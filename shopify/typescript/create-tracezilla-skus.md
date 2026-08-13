---
title: Create tracezilla SKUs
layout: default
parent: TypeScript
grand_parent: Shopify
nav_order: 20
---

# Create tracezilla SKUs from Shopify

{: .label .label-yellow }
Writes with explicit confirmation

## Behavior

The command reads Shopify product variants and the existing tracezilla SKU
catalog. It previews or creates a tracezilla SKU for each selected Shopify SKU
code that does not already exist.

Complete the [TypeScript installation and configuration](../typescript.html)
first. Use a development Shopify store and a test tracezilla team.

### Mapping

The mapping is deliberately visible in
`src/workflows/create-tracezilla-skus.ts`:

| Shopify source | tracezilla field | Example value |
|---|---|---|
| Variant SKU | `sku_code` | Shopify SKU |
| Variant SKU | `global_name` | Shopify SKU |
| Example assumption | `weight_factor_net` | `1.0` |
| Example assumption | `weight_factor_gross` | `1.0` |
| Example assumption | `unit_of_measure` | `pcs` |
| Example assumption | `lot_unit` | `colli` |
| Example assumption | `default_uom_conversion` | `1.0` |

These fixed values demonstrate where the customer's product, unit, weight, and
conversion rules belong; they are not universal defaults. Review the mapping
before enabling writes.

Each variant is reported as `would_create`, `created`, `skipped`, `invalid`, or
`failed`. Existing tracezilla SKU codes and repeated Shopify SKU codes are
skipped. Variants without an SKU are invalid. Matching uses the trimmed SKU
code, not names or internal IDs.

## Run the command

Dry run is the default and processes at most ten Shopify variants:

```bash
docker compose run --rm --entrypoint npm app run create-skus -- --limit=10
```

## Options

Choose another processing limit or request complete JSON:

```bash
docker compose run --rm --entrypoint npm app run create-skus -- --limit=25
docker compose run --rm --entrypoint npm app run create-skus -- --limit=25 --json
```

The limit controls how many Shopify variants are processed. The command still
reads the complete Shopify and tracezilla catalogs to report totals and avoid
duplicates.

## Safety and exit status

Inspect a dry run first. For the first sandbox write, use a limit of one:

```bash
docker compose run --rm --entrypoint npm app run create-skus -- --execute --confirm --limit=1
```

Both `--execute` and `--confirm` are required. `--execute` alone exits with an
error and writes nothing. Increase the limit only after inspecting the created
record and its unit mapping.

The command requires Shopify `read_products` and permission to list and create
tracezilla SKUs. It never writes to Shopify. Runs without failed writes return
`0`; invalid options, configuration or API errors, and failed writes return
non-zero. A later run reads tracezilla again and skips successfully created SKUs.

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
| CLI safety, options, and composition | `src/cli/create-tracezilla-skus.ts` |
| Shopify variant retrieval | `src/shopify/shopify-catalog-service.ts` |
| Customer-specific mapping and decisions | `src/workflows/create-tracezilla-skus.ts` |
| Existing-SKU lookup and creation | `src/tracezilla/tracezilla-catalog-service.ts` |
| Authenticated HTTP writes | `src/tracezilla/tracezilla-client.ts` |
| Structured summary and items | `src/workflows/create-tracezilla-skus.ts` |

The workflow is the customization point for the example mapping. There is no
generic rule engine: business assumptions remain explicit beside the payload.

## Tests

```bash
docker compose run --rm --entrypoint npm app test
docker compose run --rm --entrypoint npm app run typecheck
```

Tests verify dry-run decisions without contacting either API, while the strict
typecheck verifies all source and test code. Use a sandbox dry run to verify
the configured accounts before any bounded write.
