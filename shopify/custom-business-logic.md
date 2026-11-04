---
title: Implement Custom Business Logic
layout: default
parent: Shopify
nav_order: 5
---

# Implement custom business logic

This page shows how to turn a customer's request into a feature that runs
reliably on a schedule. Keep the business rules in your own PHP classes, not in
cron, an API client, or a webhook handler.

## Generate the starting point

Generate a scenario instead of copying framework classes by hand:

With Docker:

```bash
docker compose run --rm php php bin/bifrost-connect scenario:create confirm-credentials --platform=shopify
```

Without Docker:

```bash
php bin/bifrost-connect scenario:create confirm-credentials --platform=shopify
```

The command creates four files under
`custom/Scenarios/Shopify/ConfirmCredentials/`. They are yours to change:

| File | What you do in it |
|---|---|
| `ShopifyQuery.graphql` | Request only the Shopify fields required by the feature |
| `TracezillaRequest.php` | Declare the read-only tracezilla endpoint and query parameters |
| `BusinessRules.php` | Validate inputs and express the customer's decisions in PHP |
| `BusinessRulesTest.php` | Turn the customer story into examples that run without live APIs |

The generated “hello world” is read-only. It requests the Shopify
shop name, reads a one-record page from tracezilla, and reports that both sets
of credentials work. It never prints credentials or access tokens.

Run the generated test and scenario:

With Docker:

```bash
docker compose run --rm php composer test
docker compose run --rm php php bin/bifrost-connect scenario:run confirm-credentials --platform=shopify
```

Without Docker:

```bash
composer test
php bin/bifrost-connect scenario:run confirm-credentials --platform=shopify
```

Scenarios are read-only. Do not put calls that change Shopify or tracezilla in
`BusinessRules.php`; build writes as a workflow command, as described below.

## Start with the business outcome

Write the rule in customer language before writing PHP. For example:

> Every five minutes, import paid Shopify orders into tracezilla. Skip
> cancelled orders, use customer `WEB-B2B` for wholesale orders and `WEB-B2C`
> for other orders, reject unknown SKUs, and never create the same order twice.

Record these decisions:

| Decision | Example |
|---|---|
| Trigger | Cron every five minutes |
| Systems read | Recent Shopify orders and existing tracezilla references |
| System written | tracezilla sales orders |
| Selection | Paid, non-cancelled orders |
| Identity | `SHP` plus the immutable Shopify order ID |
| Customer mapping | Wholesale tag selects `WEB-B2B`; otherwise `WEB-B2C` |
| Retry behavior | Existing external reference is a successful skip |
| Recovery window | Read more than five minutes so runs overlap safely |
| Failure result | Non-zero exit status and a per-order failure reason |

This short specification becomes both the implementation checklist and the
test plan.

## Understand one scheduled run

<pre class="mermaid">
flowchart TB
    Cron[Cron starts bounded reconciliation]
    Command[Console command validates options and execution mode]
    ReadCommerce[Workflow reads changed Shopify records]
    CommerceData[Shopify returns typed commerce data]
    ReadTracezilla[Workflow reads tracezilla context and existing identities]
    TracezillaData[tracezilla returns context and known references]
    ApplyRules[Customer PHP rules select, validate, and map each record]
    Write[Workflow performs approved idempotent writes]
    ItemResult[tracezilla returns created, skipped, or failed]
    Result[Workflow returns a structured result]
    Exit[Command writes output and exit status for cron]

    Cron --> Command --> ReadCommerce --> CommerceData --> ReadTracezilla
    ReadTracezilla --> TracezillaData --> ApplyRules --> Write --> ItemResult
    ItemResult --> Result --> Exit
</pre>

Cron does not contain integration logic. Its only responsibility is to start
the command regularly and make failures observable.

## Put code in the correct layer

| Concern | Location | Example |
|---|---|---|
| Scheduling | Server configuration | Cron expression and overlap lock |
| Command options | `bin/` | Lookback period, limit, dry run, and execution flag |
| Shopify transport | `src/Shopify/` | GraphQL query, authentication, and pagination |
| tracezilla transport | `src/Tracezilla/` | API request, authentication, and response mapping |
| Stable boundaries | `src/Contracts/` | Order reader or sales-order gateway interfaces |
| Customer decisions | Focused rule or mapper class | Customer, currency, SKU, unit, and exclusion rules |
| Orchestration | `src/Workflows/` | Read, decide, deduplicate, write, and collect results |
| Human/JSON output | `src/Output/` | Report the result without changing it |
| Verification | `tests/Unit/` | Business examples using fakes, without live APIs |

Do not put customer rules into `ShopifyClient`, `TracezillaClient`, a GraphQL
query, cron, or `bin/bifrost-connect`. Those components should remain
stable when the customer's rules change.

## Implement the feature

Use this order of work:

1. Describe accepted, skipped, invalid, failed, and already-processed cases.
2. Add missing source fields to a named Shopify query and typed data object.
3. Extend a focused reader or gateway contract only if the workflow needs a
   new capability.
4. Implement customer selection and mapping in an explicit PHP rule or mapper
   class.
5. Make the workflow coordinate reads, rules, duplicate checks, writes, and
   structured results.
6. Require an explicit mode, `--dry-run` or `--execute`, and a
   bounded limit.
7. Add the thin entry-point script and register its public name in
   `bin/bifrost-connect`.
8. Unit-test business decisions and retry behavior with fake readers and
   gateways.
9. Run a bounded dry run against test accounts, inspect every decision, and
   perform one approved write.
10. Repeat the same run and verify that it safely reports the record as already
    processed.

The existing individual-order workflow demonstrates this separation:

- `ShopifyOrderService` reads recent orders.
- `ShopifyOrderToTracezillaSalesOrderMapper` contains visible example mapping
  decisions that must be customized.
- `ImportIndividualOrders` handles cancellation, partial input, duplicate
  external references, dry run, writes, and per-item results.
- `bin/import-individual-orders` is the thin composition and output adapter.

## Test the customer rules

Each business sentence should have a focused unit test. At minimum, cover:

- A normal order maps to the expected customer and lines.
- A wholesale order selects the wholesale customer.
- An unpaid or cancelled order is skipped with a stable reason.
- An unsupported currency or unknown SKU is rejected safely.
- An existing external reference is not written again.
- One failed record does not hide the status of other records.
- A dry run performs no writes.

Run the complete suite in the development container:

With Docker:

```bash
docker compose run --rm php composer test
```

Without Docker:

```bash
composer test
```

## Schedule the command

Run the command manually and verify its dry-run output before adding cron. A
Docker-based production schedule can use `flock` to prevent overlapping runs:

```cron
*/5 * * * * flock -n /tmp/tracezilla-orders.lock docker compose -f /opt/customer-integration/compose.yaml run --rm -T php php bin/bifrost-connect orders:import-individual --customer='WEB-B2C' --warehouse=2 --days=1 --limit=100 --execute >> /var/log/tracezilla-orders.log 2>&1
```

When PHP and Composer are installed directly on the host, cron can invoke the
same application without Docker:

```cron
*/5 * * * * flock -n /tmp/tracezilla-orders.lock /usr/bin/php /opt/customer-integration/bin/bifrost-connect orders:import-individual --customer='WEB-B2C' --warehouse=2 --days=1 --limit=100 --execute >> /var/log/tracezilla-orders.log 2>&1
```

Use absolute paths because cron has a minimal environment. Ensure failures are
monitored rather than only written to an unread log.

{: .warning }
The cron entries are examples; do not copy them into production unchanged. Before scheduling a workflow, verify that its pagination,
selection limit, lookback window, and idempotency rule cannot leave older
eligible records permanently behind. Treat the example order command as a
starting point that you harden for each customer.

## Design for reconciliation

A five-minute schedule must not mean “read exactly the last five minutes.” A
run might start late, Shopify might be temporarily unavailable, or the server
might be offline. Read an overlapping period and use a stable identity to make
reprocessing safe.

For orders, an immutable commerce order ID stored as the tracezilla external
reference provides the idempotency boundary. For inventory, compare current
source and destination quantities before writing. For a new workflow, choose a
stable ID that stops duplicates, and decide how a failed item is retried.

## If you add a webhook

The integration has no webhook support. If you add a webhook so a change is
handled sooner, let it call the same workflow as cron, so the rules exist only
once.

```text
Webhook ─┐
         ├──> same workflow and customer rules
Cron ────┘
```

Keep cron running as well. It picks up events whose webhook was missed, failed,
or was only partly processed. Because both call the same workflow, they share
the same duplicate checks and results.
