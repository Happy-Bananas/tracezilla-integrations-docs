---
title: Implement Custom Business Logic
layout: default
parent: PHP
grand_parent: Shopify
nav_order: 5
---

# Implement custom business logic

This guide gives the consultant the helicopter view: how a customer request
becomes a reliable scheduled integration feature without putting business
rules in cron, an API client, or a future webhook handler.

## Generate the starting point

After cloning and configuring the integration, generate a scenario instead of
copying framework classes by hand:

```bash
php bin/tracezilla-integration scenario:create confirm-credentials --platform=shopify
```

With Docker, run the same command inside the integration service:

```bash
docker compose exec integration php bin/tracezilla-integration scenario:create confirm-credentials --platform=shopify
```

The command creates this consultant-owned feature:

```text
custom/Scenarios/Shopify/ConfirmCredentials/
├── ShopifyQuery.graphql
├── TracezillaRequest.php
├── BusinessRules.php
└── BusinessRulesTest.php
```

The generated “hello world” is deliberately read-only. It requests the Shopify
shop name, reads a one-record page from tracezilla, and reports that both sets
of credentials work. It never prints credentials or access tokens.

Run the generated test and scenario:

```bash
composer test
php bin/tracezilla-integration scenario:run confirm-credentials --platform=shopify
```

The four generated files are the normal consultant editing surface:

| File | Consultant responsibility |
|---|---|
| `ShopifyQuery.graphql` | Request only the Shopify fields required by the feature |
| `TracezillaRequest.php` | Declare the read-only tracezilla endpoint and query parameters |
| `BusinessRules.php` | Validate inputs and express the customer's decisions in PHP |
| `BusinessRulesTest.php` | Turn the customer story into examples that run without live APIs |

The initial runner supports read-only scenarios. Write scenarios will be added
behind explicit preview, `--execute`, and `--confirm` safeguards; do not place
write calls inside `BusinessRules.php`.

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
query, cron, or `bin/tracezilla-integration`. Those components should remain
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
6. Default writes to dry run. Require explicit execution flags and a bounded
   limit.
7. Add the thin entry-point script and register its public name in
   `bin/tracezilla-integration`.
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

```bash
docker compose exec integration composer test
```

## Schedule the command

Run the command manually and verify its dry-run output before adding cron. A
Docker-based production schedule can use `flock` to prevent overlapping runs:

```cron
*/5 * * * * flock -n /tmp/tracezilla-orders.lock docker compose -f /opt/customer-integration/compose.yaml exec -T integration php bin/tracezilla-integration orders:import-individual --customer='WEB-B2C' --warehouse=2 --days=1 --limit=100 --execute --confirm >> /var/log/tracezilla-orders.log 2>&1
```

When PHP and Composer are installed directly on the host, cron can invoke the
same application without Docker:

```cron
*/5 * * * * flock -n /tmp/tracezilla-orders.lock /usr/bin/php /opt/customer-integration/bin/tracezilla-integration orders:import-individual --customer='WEB-B2C' --warehouse=2 --days=1 --limit=100 --execute --confirm >> /var/log/tracezilla-orders.log 2>&1
```

Use absolute paths because cron has a minimal environment. Ensure failures are
monitored rather than only written to an unread log.

{: .warning }
The cron entries illustrate the target operating model; do not copy them into
production unchanged. Before scheduling a workflow, verify that its pagination,
selection limit, lookback window, and idempotency rule cannot leave older
eligible records permanently behind. The current example order command is a
starting point for that customer-specific hardening.

## Design for reconciliation

A five-minute schedule must not mean “read exactly the last five minutes.” A
run might start late, Shopify might be temporarily unavailable, or the server
might be offline. Read an overlapping period and use a stable identity to make
reprocessing safe.

For orders, an immutable commerce order ID stored as the tracezilla external
reference provides the idempotency boundary. For inventory, compare current
source and destination quantities before writing. Other workflows must define
their own equivalent identity and retry rule.

## Add webhooks later

A webhook can trigger the same workflow shortly after an event. It should not
introduce a second implementation of the rules.

```text
Webhook ─┐
         ├──> same workflow and customer rules
Cron ────┘
```

Cron remains the recovery mechanism. It finds events whose webhook was missed,
whose first execution failed, or whose processing completed only partially.
Both triggers must therefore share the same idempotency checks and structured
result behavior.
