---
title: Compare Catalogs
layout: default
parent: Java
grand_parent: Shopify
nav_order: 10
---

# Compare Shopify and tracezilla catalogs

{: .label .label-green }
Read only

This Java command retrieves both complete catalogs and compares normalized SKU
codes. Complete the [Java setup](../java.html) first.

## Run the command

```bash
docker compose run --rm app
```

Change the displayed row limit or return complete JSON:

```bash
docker compose run --rm app --limit=25
docker compose run --rm app --json
```

SKU is the shared identifier. Results contain matches, Shopify-only SKUs, and
tracezilla-only SKUs. The limit affects display only, never catalog totals.

## Safety and exit status

The command requires Shopify `read_products` and tracezilla read access. It
writes no data. Differences return exit code `0`; configuration,
authentication, API, and malformed-response failures return non-zero.

## Implementation map

All Java paths below are relative to
`src/main/java/com/happybananas/tracezilla/`.

| Responsibility | Source |
|---|---|
| CLI and output | `Main.java` |
| Shopify query | `shopify/GetProductVariants.java` |
| Shopify API client | `shopify/ShopifyClient.java` |
| Shopify catalog reader | `shopify/ShopifyCatalog.java` |
| tracezilla catalog reader | `tracezilla/TracezillaCatalog.java` |
| Shared model and contract | `shared/CatalogItem.java`, `shared/CatalogReader.java` |
| Comparison | `workflow/CompareCatalogs.java` |

## Verify changes

```bash
docker compose run --rm --entrypoint mvn app test
```

The tests do not contact Shopify or tracezilla.
