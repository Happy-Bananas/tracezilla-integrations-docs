---
title: Create tracezilla SKUs
layout: default
parent: Ruby
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

Complete the [Ruby installation and configuration](../ruby.html) first. Use a
development Shopify store and a test tracezilla team.

### Mapping

The mapping is deliberately visible in
`lib/tracezilla_shopify/create_tracezilla_skus.rb`:

| Shopify source | tracezilla field | Example value |
|---|---|---|
| Variant SKU | `sku_code` | Shopify SKU |
| Variant SKU | `global_name` | Shopify SKU |
| Example assumption | `weight_factor_net` | `1.0` |
| Example assumption | `weight_factor_gross` | `1.0` |
| Example assumption | `unit_of_measure` | `pcs` |
| Example assumption | `lot_unit` | `colli` |
| Example assumption | `default_uom_conversion` | `1.0` |

These are visible example assumptions rather than universal defaults. Review
the customer's product units, weights, and conversions before enabling writes.

Each variant is reported as `would_create`, `created`, `skipped`, `invalid`, or
`failed`. Existing tracezilla SKUs and duplicate Shopify SKUs are skipped.
Variants without an SKU are invalid. Matching uses the trimmed SKU code.

## Run the command

Dry run is the default and processes at most ten variants:

### With Docker

```bash
docker compose run --rm --entrypoint bundle app exec ruby bin/create_tracezilla_skus --limit=10
```

### Without Docker

```bash
bundle exec ruby bin/create_tracezilla_skus --limit=10
```

## Options

Choose another processing limit or request JSON:

```bash
docker compose run --rm --entrypoint bundle app exec ruby bin/create_tracezilla_skus --limit=25
docker compose run --rm --entrypoint bundle app exec ruby bin/create_tracezilla_skus --limit=25 --json
```

The limit bounds processing, while complete catalogs are read to report source
counts and detect existing SKU codes.

## Safety and exit status

Inspect a dry run first, then use a one-record sandbox write:

```bash
docker compose run --rm --entrypoint bundle app exec ruby bin/create_tracezilla_skus --execute --confirm --limit=1
```

Both safety flags are required; `--execute` alone fails without writing.
Increase the limit only after inspecting the record and its unit mapping.

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
| CLI safety, options, and composition | `bin/create_tracezilla_skus` |
| Shopify variant retrieval | `lib/tracezilla_shopify/shopify/catalog_service.rb` |
| Customer-specific mapping and decisions | `lib/tracezilla_shopify/create_tracezilla_skus.rb` |
| Existing-SKU lookup and creation | `lib/tracezilla_shopify/tracezilla/catalog_service.rb` |
| Authenticated HTTP writes | `lib/tracezilla_shopify/tracezilla/client.rb` |
| Structured summary and items | `lib/tracezilla_shopify/create_tracezilla_skus.rb` |

Ruby's compact implementation keeps mapping and workflow decisions together.
The business assumptions remain explicit beside the payload.

## Tests

```bash
docker compose run --rm --entrypoint bundle app exec rake test
```

Tests verify dry-run decisions without live APIs. Use a sandbox dry run to
verify credentials and mappings before any bounded write.
