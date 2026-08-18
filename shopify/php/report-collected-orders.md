---
title: Report Collected Orders
layout: default
parent: PHP
grand_parent: Shopify
nav_order: 50
---

# Report collected Shopify orders

{: .label .label-green }
Read only

## Behavior

The command reads recently created Shopify orders and groups their reportable
lines by business date, currency, and SKU. It totals quantity and discounted
revenue for each group. It never changes Shopify or tracezilla.

Complete the [PHP installation and configuration](../php.html) first. The
Shopify app needs the `read_orders` scope.

Orders are skipped when they are cancelled or contain more than the 250 lines
returned by the example query. Lines are skipped when the SKU, positive
quantity, numeric unit price, or currency is missing. These explicit skips
prevent a plausible-looking but incomplete report.

## Run the command

The following example reports at most ten orders created during the previous
three days. `Europe/Copenhagen` defines when one reporting day ends and the
next begins.

### With Docker

```bash
docker compose run --rm php php bin/report-collected-orders \
  --days=3 \
  --timezone=Europe/Copenhagen \
  --limit=10
```

### Without Docker

```bash
php bin/report-collected-orders \
  --days=3 \
  --timezone=Europe/Copenhagen \
  --limit=10
```

## Options

- `--days=N` selects orders created during the previous number of days and
  defaults to `3`.
- `--timezone=Area/City` defines the business date and defaults to `UTC`.
- `--limit=N` bounds the number of orders included and defaults to `10`.
- `--json` returns the complete structured report for automation.

Use an IANA timezone such as `Europe/Copenhagen`, not a fixed abbreviation.
Invalid or non-positive values fail with a non-zero exit code.

## Output and safety

The terminal table contains date, currency, SKU, quantity, and revenue. Its
summary distinguishes orders returned by Shopify, orders selected by the
limit, skipped orders, and skipped lines.

For machine-readable output:

```bash
docker compose run --rm php php bin/report-collected-orders \
  --days=3 --timezone=Europe/Copenhagen --limit=10 --json
```

The limit is applied after Shopify returns orders from the requested date
window. Keep both `--days` and `--limit` small while validating a store. A
successful report, including an empty report, returns exit code `0`; invalid
configuration, options, authentication, or API responses return non-zero.

## Architecture

<pre class="mermaid">
flowchart TB
    Query[GetOrders GraphQL query] --> ShopifyClient[ShopifyClient]
    ShopifyClient --> OrderService[ShopifyOrderService]
    OrderService --> OrderModel[ShopifyOrderData and line data]
    OrderModel --> Workflow[BuildCollectedOrderReport]
    Options[Days, timezone, and limit] --> Workflow
    Workflow --> Result[CollectedOrderReportResult]
    Result --> Table[Terminal table]
    Result --> Json[JSON output]
</pre>

## Implementation

| Responsibility | Source |
|---|---|
| CLI options, composition, output, and exit status | `bin/report-collected-orders` |
| Shopify order fields | `src/Shopify/Queries/GetOrders.php` |
| Pagination and date filtering | `src/Shopify/ShopifyOrderService.php` |
| Normalized order data | `src/Shopify/ShopifyOrderData.php`, `ShopifyOrderLineData.php` |
| Grouping, validation, and totals | `src/Workflows/BuildCollectedOrderReport.php` |
| Validated command options | `src/Workflows/CollectedOrderReportOptions.php` |
| Structured result | `src/Workflows/CollectedOrderReportResult.php` |
| Human-readable output | `src/Output/CollectedOrderReportRenderer.php` |

The workflow depends on the small `ShopifyOrderReader` contract. Its grouping
and skip rules can therefore be tested with in-memory orders rather than live
API credentials.

## Tests

```bash
docker compose run --rm php composer test
```

Tests verify timezone grouping, revenue totals, limits, skip rules, option
validation, structured results, and terminal output without contacting
Shopify or tracezilla.
