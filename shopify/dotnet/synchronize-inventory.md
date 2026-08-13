---
title: Synchronize Inventory
layout: default
parent: C# / .NET
grand_parent: Shopify
nav_order: 40
---

# Synchronize Shopify inventory from tracezilla

{: .label .label-yellow }
Writes with explicit confirmation

## Behavior

The command reads available inventory from one tracezilla warehouse and current available quantities at one Shopify location. It matches trimmed SKU codes and reports `would_update`, `updated`, `unchanged`, `skipped`, or `failed`.

Complete the [C# / .NET installation and configuration](../dotnet.html) first. Both location arguments are required, keeping the source-to-target relationship visible.

### Quantity mapping

The example calculates Shopify availability as:

```text
(traceable available × default UOM conversion)
+ (non-traceable available × non-traceable UOM conversion)
```

Negative, fractional, or non-finite results fail safely. Review this example business rule for each customer.

## Run the command

Find the target GraphQL ID with List Shopify Locations, then preview:

```bash
docker compose run --rm app synchronize-inventory \\\n  --shopify-location=gid://shopify/Location/123 --tracezilla-warehouse=2 --limit=10
```

Dry run is the default and changes nothing.

## Options

- `--shopify-location=gid://...` selects the Shopify target.
- `--tracezilla-warehouse=N` selects the tracezilla source.
- `--limit=N` bounds processing and defaults to 10.
- `--execute --confirm` enables writes explicitly.

## Safety and exit status

Review a dry run before adding `--execute --confirm --limit=1`. Both flags are required. Shopify writes include the previously read quantity as `compareQuantity`, rejecting stale concurrent changes. The command needs `read_products`, `read_inventory`, and `write_inventory`. Failures return non-zero.

## Architecture

<pre class="mermaid">
flowchart TB
 subgraph Tracezilla[tracezilla source]
  TZClient[API client] --> TZService[Inventory service] --> TZModel[Inventory data]
 end
 subgraph Shopify[Shopify target]
  Query[Inventory query] --> ShopifyService[Inventory service]
  Mutation[Set quantity mutation] --> ShopifyService
 end
 TZModel --> Workflow[SynchronizeInventory]
 ShopifyService --> Workflow --> Mapper[Quantity mapping]
 Mapper --> Result[Structured result]
</pre>

## Implementation

| Responsibility | Source |
|---|---|
| CLI arguments, safety, and output | `src/TracezillaShopify/Program.cs` |
| Matching, mapping, and decisions | `src/TracezillaShopify/Workflows/SynchronizeInventory.cs` |
| Shopify pagination and guarded writes | `src/TracezillaShopify/Shopify/ShopifyInventoryService.cs` |
| tracezilla warehouse inventory | `src/TracezillaShopify/Tracezilla/TracezillaInventoryService.cs` |

The workflow owns business decisions while the API services own transport and pagination.

## Tests

```bash
docker compose run --rm --entrypoint dotnet app test tests/TracezillaShopify.Tests --no-restore
```

Tests verify conversion, dry-run behavior, and guarded updates without live APIs. A sandbox dry run verifies credentials and location mapping before any bounded write.

