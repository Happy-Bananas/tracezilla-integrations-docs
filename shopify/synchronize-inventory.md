---
title: Synchronize Inventory
layout: default
parent: Shopify
nav_order: 40
---

# Synchronize Shopify inventory from tracezilla

{: .label .label-yellow }
Writes with explicit confirmation

## Behavior

The command reads available inventory from one tracezilla warehouse and the
current available quantities at one Shopify location. Records are matched by
trimmed SKU code. It reports each record as `would_update`, `updated`,
`unchanged`, `skipped`, or `failed`.

The Shopify location ID and tracezilla warehouse number are required arguments,
so the relationship is visible whenever the command runs.

### Quantity mapping

The example calculates Shopify's available whole-unit quantity as:

```text
(traceable available × default UOM conversion)
+ (non-traceable available × non-traceable UOM conversion)
```

Negative, fractional, or non-finite results fail safely. This formula is an
example business assumption; review the customer's units, traceability, and
sellable-quantity rules before writing.

## Run the command

Find the target GraphQL ID with [List Shopify Locations](list-shopify-locations.html),
then run a bounded preview:

### With Docker

```bash
docker compose exec integration php bin/bifrost-connect inventory:sync \
  --shopify-location=gid://shopify/Location/123 \
  --tracezilla-warehouse=2 \
  --limit=10
```

### Without Docker

```bash
php bin/synchronize-inventory \
  --shopify-location=gid://shopify/Location/123 \
  --tracezilla-warehouse=2 \
  --limit=10
```

Dry run is the default. No inventory is changed.

## Options

- `--shopify-location=gid://...` selects the target Shopify location.
- `--tracezilla-warehouse=N` selects the source warehouse by location number.
- `--limit=N` bounds processed tracezilla records and defaults to `10`.
- `--json` returns the complete structured result.
- `--execute --confirm` explicitly enables writes.

Both location arguments are mandatory to prevent an implicit destination.

## Safety and exit status

After reviewing the dry run, test at most one record in sandbox accounts:

```bash
docker compose exec integration php bin/bifrost-connect inventory:sync \
  --shopify-location=gid://shopify/Location/123 \
  --tracezilla-warehouse=2 \
  --execute --confirm --limit=1
```

Both write flags are required. Shopify's mutation includes the quantity read
earlier as `compareQuantity`, so Shopify rejects a stale concurrent update.
The command requires `read_products`, `read_inventory`, and `write_inventory`.
Failures return non-zero; differences and skipped records are successful.

## Architecture

<pre class="mermaid">
flowchart TB
    subgraph Tracezilla[tracezilla source]
        TZClient[TracezillaClient] --> TZService[TracezillaInventoryService]
        TZService --> TZModel[TracezillaInventory]
    end
    subgraph Shopify[Shopify target]
        Query[GetInventoryItems query] --> ShopifyService[ShopifyInventoryService]
        Mutation[SetInventoryQuantity mutation] --> ShopifyService
        ShopifyClient[ShopifyClient] --> ShopifyService
    end
    TZModel --> Workflow[SynchronizeInventory]
    ShopifyService --> Workflow
    Workflow --> Mapper[Quantity mapper]
    Mapper --> Result[Structured summary and items]
</pre>

## Implementation

| Responsibility | Source |
|---|---|
| CLI arguments, safety flags, and output | `bin/synchronize-inventory` |
| Shopify query and mutation | `src/Shopify/Queries/GetInventoryItems.php`, `src/Shopify/Mutations/SetInventoryQuantity.php` |
| Shopify pagination and guarded write | `src/Shopify/ShopifyInventoryService.php` |
| tracezilla warehouse lookup and pagination | `src/Tracezilla/TracezillaInventoryService.php` |
| Customer-specific quantity rule | `src/Workflows/TracezillaInventoryToShopifyQuantityMapper.php` |
| Matching and update decisions | `src/Workflows/SynchronizeInventory.php` |
| Structured output | `src/Workflows/SynchronizeInventoryResult.php` |

The workflow depends on small source and target contracts, making its business
decisions testable without live APIs. API services retain pagination and
transport-specific behavior.

## Tests

```bash
docker compose exec integration composer test
```

Tests verify quantity conversion, dry-run behavior, guarded execution, and
structured results without contacting either API. A sandbox dry run verifies
credentials, location mapping, and real records before any one-record write.
