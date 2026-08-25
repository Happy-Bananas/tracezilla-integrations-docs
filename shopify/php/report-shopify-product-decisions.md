---
title: Report Shopify Product Decisions
layout: default
parent: PHP
grand_parent: Shopify
nav_order: 15
---

# Report tracezilla SKUs needing a Shopify decision

{: .label .label-green }
Read only

## Behavior

The command reads the complete Shopify variant catalog and tracezilla SKU
catalog. It reports each tracezilla SKU for which no Shopify variant has the
same trimmed SKU code. It never creates or changes a product, variant, or SKU.

Complete the [PHP installation and configuration](../php.html) before running
this command. The Shopify app needs `read_products` access, and the tracezilla
API key must be able to list SKUs.

This report is narrower than [Compare Catalogs](compare-catalogs.html). Compare
Catalogs diagnoses differences in both directions; this command turns only the
tracezilla-only category into a bounded consultant decision list containing the
SKU, tracezilla ID, and available name.

## Run the command

From your project directory, run:

### With Docker

```bash
docker compose exec integration php bin/tracezilla-integration catalog:report-shopify-decisions
```

### Without Docker

```bash
php bin/report-tracezilla-skus-needing-shopify-decision
```

The output begins with the validated PHP command and execution mode:

```text
Command: php bin/report-tracezilla-skus-needing-shopify-decision
Mode: READ ONLY
```

## Decisions and options

For every reported SKU, decide whether to:

- create a new Shopify product and variant;
- add a variant to an existing Shopify product;
- map or change the SKU code; or
- intentionally exclude the tracezilla SKU from Shopify.

Display at most 25 candidates while retaining the complete candidate count:

```bash
docker compose exec integration php bin/tracezilla-integration catalog:report-shopify-decisions --limit=25
```

The default display limit is `10`. Unlike Compare Catalogs JSON, this command's
JSON candidate list is also bounded by `--limit` so unattended output remains
manageable:

```bash
docker compose exec integration php bin/tracezilla-integration catalog:report-shopify-decisions --limit=25 --json
```

JSON remains valid and includes the execution context around the result:

```json
{
  "command": "php bin/report-tracezilla-skus-needing-shopify-decision --limit=25 --json",
  "mode": "read_only",
  "result": {
    "status": "decisions_required",
    "candidate_count": 42,
    "display_limit": 25,
    "candidates": []
  }
}
```

## Product-creation boundary

The available catalogs cannot safely determine how a tracezilla SKU should be
represented in Shopify. A future write workflow must explicitly define:

- whether the SKU belongs to a new product or an existing product's variant;
- product title, handle, vendor, product type, options, and variant values;
- draft or active status and publication channels;
- price, tax, shipping, and inventory-tracking behavior; and
- images, descriptions, and other storefront content.

The report deliberately stops at the decision boundary instead of inventing
these customer-specific rules.

## Safety and exit status

The command is read-only and has no `--execute` or `--confirm` flags. It reads
both complete catalogs to calculate an accurate candidate count, while the
limit bounds rendered candidate details. Credentials remain in `.env` and are
not included in output.

A successful report—including one with no candidates—returns exit code `0`.
Invalid limits, configuration, authentication, transport, pagination, or
malformed-response errors return a non-zero exit code. Finding candidates is a
successful report, not a command failure.

## Architecture

<pre class="mermaid">
flowchart TB
    ShopifyClient[ShopifyClient] --> ShopifyService[ShopifyCatalogService]
    ShopifyService --> ShopifyItems[Shopify CatalogItems]
    TracezillaClient[TracezillaClient] --> TracezillaService[TracezillaCatalogService]
    TracezillaService --> TracezillaItems[tracezilla CatalogItems]
    ShopifyItems --> Workflow[ReportTracezillaSkusNeedingShopifyDecision]
    TracezillaItems --> Workflow
    Workflow --> Result[ShopifyProductDecisionReportResult]
    Result --> Table[Terminal report]
    Result --> Json[Bounded JSON]
</pre>

## Implementation

| Responsibility | Source |
|---|---|
| CLI options, execution context, composition, and exit status | `bin/report-tracezilla-skus-needing-shopify-decision` |
| Shopify catalog retrieval and mapping | `src/Shopify/ShopifyCatalogService.php`, `src/Shopify/Mappers/ShopifyVariantMapper.php` |
| tracezilla catalog retrieval and mapping | `src/Tracezilla/TracezillaCatalogService.php`, `src/Tracezilla/Mappers/TracezillaSkuMapper.php` |
| Candidate selection and ordering | `src/Workflows/ReportTracezillaSkusNeedingShopifyDecision.php` |
| Bounded structured result | `src/Workflows/ShopifyProductDecisionReportResult.php` |
| Human-readable output | `src/Output/ShopifyProductDecisionReportRenderer.php` |
| Shared command and mode envelope | `src/Output/CommandExecution.php` |

The workflow depends on the same small `CatalogReader` contract as catalog
comparison. Both readers can therefore be replaced by in-memory fakes in tests;
API clients and pagination remain outside the decision rule.

## Tests

```bash
docker compose exec integration composer test
```

Tests verify filtering, deterministic SKU ordering, the display limit, source
metadata, empty results, structured status, and human guidance without
contacting Shopify or tracezilla.
