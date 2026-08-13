---
title: Create tracezilla SKUs
layout: default
parent: Java
grand_parent: Shopify
nav_order: 20
---

# Create tracezilla SKUs from Shopify

{: .label .label-yellow }
Writes with explicit confirmation

## Behavior

The command reads Shopify product variants and existing tracezilla SKUs. It
previews or creates a tracezilla SKU for each selected Shopify SKU code that is
missing from tracezilla.

Complete the [Java installation and configuration](../java.html) first. Use a
development Shopify store and a test tracezilla team.

### Mapping

The mapping is deliberately visible in
`src/main/java/com/happybananas/tracezilla/workflow/CreateTracezillaSkus.java`:

| Shopify source | tracezilla field | Example value |
|---|---|---|
| Variant SKU | `sku_code` | Shopify SKU |
| Variant SKU | `global_name` | Shopify SKU |
| Example assumption | `weight_factor_net` | `1.0` |
| Example assumption | `weight_factor_gross` | `1.0` |
| Example assumption | `unit_of_measure` | `pcs` |
| Example assumption | `lot_unit` | `colli` |
| Example assumption | `default_uom_conversion` | `1.0` |

These values demonstrate the API payload; they are not universal defaults.
Review the customer's units, weights, and conversions before enabling writes.

Each variant is reported as `would_create`, `created`, `skipped`, `invalid`, or
`failed`. Existing tracezilla SKUs and duplicate Shopify SKUs are skipped.
Variants without an SKU are invalid. Matching uses the trimmed SKU code.

## Run the command

Dry run is the default and processes at most ten variants:

```bash
docker compose run --rm app create-tracezilla-skus --limit=10
```

## Options

Choose another processing limit or request structured JSON:

```bash
docker compose run --rm app create-tracezilla-skus --limit=25
docker compose run --rm app create-tracezilla-skus --limit=25 --json
```

The limit bounds processing, while complete catalogs are read to report source
counts and detect existing SKU codes.

## Safety and exit status

Inspect a dry run first, then use a one-record sandbox write:

```bash
docker compose run --rm app create-tracezilla-skus --execute --confirm --limit=1
```

Both safety flags are required; `--execute` alone fails without writing.
Increase the limit only after inspecting the created record and its unit
mapping.

The command requires Shopify `read_products` plus tracezilla list/create SKU
permission. It never writes to Shopify. Runs without failed writes return `0`;
invalid options, configuration or API errors, and failed writes return non-zero.
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
| CLI safety, options, composition, and output | `src/main/java/com/happybananas/tracezilla/Main.java` |
| Shopify variant retrieval | `src/main/java/com/happybananas/tracezilla/shopify/ShopifyCatalog.java` |
| Customer-specific mapping and decisions | `src/main/java/com/happybananas/tracezilla/workflow/CreateTracezillaSkus.java` |
| Existing-SKU lookup and creation | `src/main/java/com/happybananas/tracezilla/tracezilla/TracezillaCatalog.java` |
| Structured summary and items | `src/main/java/com/happybananas/tracezilla/workflow/CreateTracezillaSkus.java` |

The compact Java example combines each API client with its catalog boundary.
The workflow still keeps business mapping separate from API communication.

## Tests

```bash
docker compose run --rm --entrypoint mvn app test
```

Tests and compilation run without live APIs. Verify configured accounts with a
sandbox dry run before any bounded write.
