---
title: Compare Catalogs
layout: default
parent: Python
grand_parent: Shopify
nav_order: 10
---

# Compare Shopify and tracezilla catalogs

{: .label .label-green }
Read only

This introductory Python command reads the complete Shopify variant and
tracezilla SKU catalogs and compares their normalized SKU codes. Complete the
[Python setup](../python.html) first.

## Run the command

```bash
docker compose run --rm app
```

Control displayed rows or request the complete JSON result:

```bash
docker compose run --rm app --limit=25
docker compose run --rm app --json
```

The result separates SKUs present in both systems, only Shopify, or only
tracezilla. `--limit` affects display only; all records contribute to totals.

## Safety and exit status

Only Shopify `read_products` and tracezilla read access are required. The
command performs no writes. Catalog differences return `0`; configuration,
authentication, API, and invalid-response errors return a non-zero exit code.

## Implementation map

| Responsibility | Source |
|---|---|
| CLI | `src/tracezilla_shopify/cli.py` |
| Shopify query | `src/tracezilla_shopify/shopify/query.py` |
| Shopify client, service, mapper | `src/tracezilla_shopify/shopify/` |
| tracezilla client, service, mapper | `src/tracezilla_shopify/tracezilla/` |
| Shared model | `src/tracezilla_shopify/shared.py` |
| Comparison | `src/tracezilla_shopify/workflow.py` |
| Output | `src/tracezilla_shopify/output.py` |

## Verify changes

```bash
docker compose run --rm --entrypoint pytest app
docker compose run --rm --entrypoint mypy app src tests
```

Neither check contacts a live API.
