---
title: Create tracezilla SKUs
layout: default
parent: Python
grand_parent: Experimental Implementations
nav_order: 20
---

# Create tracezilla SKUs from Shopify

{: .label .label-yellow }
Experimental

{: .label .label-yellow }
Writes with explicit confirmation

## Behavior

The command reads Shopify product variants and existing tracezilla SKUs. It
previews or creates a tracezilla SKU for each selected Shopify SKU code that is
missing from tracezilla.

Complete the [Python installation and configuration](../python.html) first.
Use development and test accounts while adapting the example.

### Mapping

The mapping is deliberately visible in
`src/tracezilla_shopify/create_skus.py`:

| Shopify source | tracezilla field | Example value |
|---|---|---|
| Variant SKU | `sku_code` | Shopify SKU |
| Variant SKU | `global_name` | Shopify SKU |
| Example assumption | `weight_factor_net` | `1.0` |
| Example assumption | `weight_factor_gross` | `1.0` |
| Example assumption | `unit_of_measure` | `pcs` |
| Example assumption | `lot_unit` | `colli` |
| Example assumption | `default_uom_conversion` | `1.0` |

The fixed values are example business assumptions, not universal defaults.
Review them for the customer's units, weights, and conversions before writing.

Each selected variant becomes `would_create`, `created`, `skipped`, `invalid`,
or `failed`. Existing tracezilla SKUs and repeated Shopify SKUs are skipped;
variants without an SKU are invalid. Matching uses trimmed SKU code only.

## Run the command

Dry run is the default and processes at most ten variants:

### With Docker

```bash
docker compose run --rm --entrypoint create-tracezilla-skus app --limit=10
```

### Without Docker

```bash
create-tracezilla-skus --limit=10
```

## Options

Choose another processing limit or request complete JSON:

```bash
docker compose run --rm --entrypoint create-tracezilla-skus app --limit=25
docker compose run --rm --entrypoint create-tracezilla-skus app --limit=25 --json
```

The limit bounds processed Shopify variants, while both complete catalogs are
read to calculate source counts and prevent duplicates.

## Safety and exit status

Inspect a dry run before a bounded sandbox write:

```bash
docker compose run --rm --entrypoint create-tracezilla-skus app --execute --confirm --limit=1
```

Both confirmation flags are required. Supplying only `--execute` fails without
writing. Increase the limit only after inspecting the created record.

The command requires Shopify `read_products` plus tracezilla list/create SKU
permission. It never writes to Shopify. Successful runs without failed writes
return `0`; option, configuration, API, and individual write failures return
non-zero. Successfully created SKU codes are skipped on subsequent runs.

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
| CLI safety, options, and composition | `src/tracezilla_shopify/create_skus_cli.py` |
| Shopify variant retrieval | `src/tracezilla_shopify/shopify/service.py` |
| Customer-specific mapping and decisions | `src/tracezilla_shopify/create_skus.py` |
| Existing-SKU lookup and creation | `src/tracezilla_shopify/tracezilla/service.py` |
| Authenticated HTTP writes | `src/tracezilla_shopify/tracezilla/client.py` |
| Structured summary and items | `src/tracezilla_shopify/create_skus.py` |

The workflow keeps the mapping explicit next to the tracezilla payload; no
generic configuration or rule engine hides the customer-specific assumptions.

## Tests

```bash
docker compose run --rm --entrypoint pytest app
docker compose run --rm --entrypoint mypy app src tests
```

Tests verify dry-run decisions using in-memory collaborators, and strict mypy
checks the source and tests. Run against sandbox accounts before any write.
