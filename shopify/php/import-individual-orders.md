---
title: Import Individual Orders
layout: default
parent: PHP
grand_parent: Shopify
nav_order: 60
---

# Import individual Shopify orders

{: .label .label-yellow }
Writes with explicit confirmation

## Behavior

The command reads recently created Shopify orders and maps each selected order
to one tracezilla sales order. It is a dry run by default. Existing tracezilla
orders are detected by the external reference `SHP<Shopify legacy order ID>`.

Complete the [PHP installation and configuration](../php.html) first. The
Shopify app needs the `read_orders` scope, and the tracezilla API key must be
able to read partners, locations, and sales orders. Execution also requires
permission to create sales orders.

## Mapping assumptions

This is an example mapping, not a universal Shopify-to-tracezilla contract:

- only DKK orders are accepted;
- the tracezilla exchange rate is `100`;
- each Shopify SKU must already exist in tracezilla;
- quantities and discounted unit prices are grouped by SKU;
- the selected tracezilla customer supplies the customer relationship;
- the selected warehouse supplies the pickup relationship;
- the Shopify shipping address becomes the delivery location;
- orders enter tracezilla with status `from_edi`;
- post-save, missing-inventory, and missing-lot actions are disabled.

Review
`src/Tracezilla/Mappers/ShopifyOrderToTracezillaSalesOrderMapper.php` before
execution. Adapt currency, reference, dates, price interpretation, partner
roles, order status, and post-save behavior to the customer.

The command rejects an entire order when a line lacks a usable SKU, positive
quantity, numeric price, or matching currency. It does not silently create a
partial sales order. Cancelled orders and orders with more than 250 lines are
skipped; line-item pagination must be implemented before importing the latter.

## Run a dry run

The customer name must exactly identify a tracezilla customer partner. The
warehouse is selected by its tracezilla location number.

### With Docker

```bash
docker compose run --rm php php bin/import-individual-orders \
  --customer='Banana primary webshop' \
  --warehouse=2 \
  --days=3 \
  --limit=10
```

### Without Docker

```bash
php bin/import-individual-orders \
  --customer='Banana primary webshop' \
  --warehouse=2 \
  --days=3 \
  --limit=10
```

The output begins with the reproducible PHP command and mode:

```text
Command: php bin/import-individual-orders --customer='Banana primary webshop' --warehouse=2 --days=3 --limit=10
Mode: DRY RUN
```

## Options

- `--customer='Exact name'` selects the tracezilla customer partner and is
  required.
- `--warehouse=N` selects the tracezilla pickup warehouse by location number
  and is required.
- `--days=N` selects recently created Shopify orders and defaults to `3`.
- `--limit=N` bounds selected orders and defaults to `10`.
- `--json` returns `command`, `mode`, and the complete structured result.
- `--execute --confirm` enables tracezilla writes; both flags are required.

Credentials remain in `.env` and are never included in command output.

## Execute one sandbox order

First ensure the dry run contains exactly the intended customer, warehouse,
address, SKUs, quantities, prices, currency, and external reference. Then test
one order in sandbox accounts:

```bash
docker compose run --rm php php bin/import-individual-orders \
  --customer='Banana primary webshop' \
  --warehouse=2 \
  --days=3 \
  --limit=1 \
  --execute --confirm
```

If either write flag is missing, the command exits before constructing API
clients. Execution reports `Mode: EXECUTE`. Never place the write flags in a
cron entry until dry runs and a controlled sandbox write have been reviewed.

## Results and exit status

Each selected order receives one status:

- `would_create` — valid and missing during dry run;
- `created` — successfully created during explicit execution;
- `skipped` — cancelled, truncated, duplicate, or already imported;
- `invalid` — the example mapping cannot safely represent the order;
- `failed` — tracezilla rejected the creation request.

An empty result and skipped duplicates are successful. Invalid records, failed
writes, configuration errors, authentication errors, and unsafe options return
a non-zero exit code so a scheduler can detect them.

## Architecture

<pre class="mermaid">
flowchart TB
    Query[GetOrders GraphQL query] --> ShopifyClient[ShopifyClient]
    ShopifyClient --> OrderService[ShopifyOrderService]
    OrderService --> ShopifyOrder[ShopifyOrderData]
    TZClient[TracezillaClient] --> TZService[TracezillaSalesOrderService]
    TZService --> Context[Customer, owner, and warehouse context]
    ShopifyOrder --> Workflow[ImportIndividualOrders]
    Context --> Workflow
    Workflow --> Mapper[Shopify order mapper]
    Mapper --> SalesOrder[TracezillaSalesOrderData]
    SalesOrder --> DryRun[Dry-run result]
    SalesOrder --> GuardedWrite[Guarded tracezilla write]
</pre>

## Implementation

| Responsibility | Source |
|---|---|
| CLI arguments, safety flags, command context, and exit status | `bin/import-individual-orders` |
| Shopify order fields | `src/Shopify/Queries/GetOrders.php` |
| Shopify pagination and date filtering | `src/Shopify/ShopifyOrderService.php` |
| Shopify order and line models | `src/Shopify/ShopifyOrderData.php`, `ShopifyOrderLineData.php` |
| Customer, warehouse, duplicates, and guarded creation | `src/Tracezilla/TracezillaSalesOrderService.php` |
| Customer-specific business mapping | `src/Tracezilla/Mappers/ShopifyOrderToTracezillaSalesOrderMapper.php` |
| Dry-run and execution decisions | `src/Workflows/ImportIndividualOrders.php` |
| Structured result | `src/Workflows/ImportIndividualOrdersResult.php` |
| Terminal output | `src/Output/IndividualOrderImportRenderer.php` |

Small reader and gateway contracts keep workflow and mapping tests independent
of live APIs. HTTP clients and services retain authentication, pagination, and
transport behavior.

## Tests

```bash
docker compose run --rm php composer test
```

Tests verify mapping, line aggregation, partial-order rejection, duplicate
protection, limits, dry run, guarded writes, failure continuation, structured
results, and human output without contacting Shopify or tracezilla.
