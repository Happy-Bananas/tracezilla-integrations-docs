---
title: Compare Catalogs
layout: default
parent: C# / .NET
grand_parent: Shopify
nav_order: 10
---

# Compare Shopify and tracezilla catalogs

{: .label .label-green }
Read only

This .NET command reads both complete catalogs and compares normalized SKU
codes. Complete the [C# / .NET setup](../dotnet.html) first.

## Run the command

```bash
docker compose run --rm app
```

Change the displayed limit or request complete JSON:

```bash
docker compose run --rm app --limit=25
docker compose run --rm app --json
```

Matches use SKU—not titles or internal IDs. The limit changes only displayed
rows; summary counts always use the complete catalogs.

## Safety and exit status

The command needs Shopify `read_products` and tracezilla read access. It does
not write data. Differences return exit code `0`; configuration,
authentication, API, and malformed-response failures return non-zero.

## Implementation map

| Responsibility | Source |
|---|---|
| CLI composition | `src/TracezillaShopify/Program.cs` |
| Shopify query | `src/TracezillaShopify/Shopify/GetProductVariants.cs` |
| Shopify client, service, mapper | `src/TracezillaShopify/Shopify/` |
| tracezilla client, service, mapper | `src/TracezillaShopify/Tracezilla/` |
| Shared model | `src/TracezillaShopify/Shared/CatalogItem.cs` |
| Comparison | `src/TracezillaShopify/Workflows/CompareCatalogs.cs` |
| Output | `src/TracezillaShopify/Output/TableRenderer.cs` |

## Verify changes

```bash
docker compose run --rm --entrypoint dotnet app test tests/TracezillaShopify.Tests --no-restore
```

The tests do not contact live APIs.
