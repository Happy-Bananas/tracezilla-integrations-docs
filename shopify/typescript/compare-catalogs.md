---
title: Compare Catalogs
layout: default
parent: TypeScript
grand_parent: Shopify
nav_order: 10
---

# Compare Shopify and tracezilla catalogs

{: .label .label-green }
Read only

This introductory TypeScript command reads both complete catalogs, normalizes
SKU codes, and reports SKUs found in both systems or only one system. Complete
the [TypeScript setup](../typescript.html) first.

## Run the command

```bash
docker compose run --rm app
```

Change the maximum displayed rows per category or return complete JSON:

```bash
docker compose run --rm app --limit=25
docker compose run --rm app --json
```

`--limit` affects display only; totals always cover the complete catalogs.
SKU—not title, name, or internal ID—is the matching identifier.

## Safety and exit status

The command needs Shopify `read_products` and tracezilla read access. It never
writes to either API. Differences return exit code `0`; configuration,
authentication, API, and malformed-response failures return a non-zero code.

## Implementation map

| Responsibility | Source |
|---|---|
| CLI | `src/cli/compare-catalogs.ts` |
| Shopify query | `src/shopify/queries/get-product-variants.ts` |
| Shopify retrieval and mapping | `src/shopify/shopify-catalog-service.ts`, `src/shopify/shopify-variant-mapper.ts` |
| tracezilla retrieval and mapping | `src/tracezilla/tracezilla-catalog-service.ts`, `src/tracezilla/tracezilla-sku-mapper.ts` |
| Comparison | `src/workflows/compare-catalogs.ts` |
| Shared model | `src/shared/catalog-item.ts` |
| Output | `src/output/table-renderer.ts` |

## Verify changes

```bash
docker compose run --rm app npm test
docker compose run --rm app npm run typecheck
```

These checks do not contact Shopify or tracezilla.
