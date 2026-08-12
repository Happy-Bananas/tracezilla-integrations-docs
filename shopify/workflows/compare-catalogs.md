---
title: Compare Catalogs
layout: default
parent: Workflows
grand_parent: Shopify
nav_order: 10
---

# Compare Shopify and tracezilla catalogs

{: .label .label-green }
Read only

Compare Catalogs is the introductory integration workflow shared by all
implementation platforms. It reads both complete catalogs, normalizes their
SKU codes, and reports matches and differences without writing to either API.

## Comparison rule

SKU code is the shared identifier. Product titles, variant names, and internal
IDs do not determine a match.

The result separates SKU codes into:

- present in both systems;
- present only in Shopify;
- present only in tracezilla.

A successful comparison returns exit code `0` even when differences exist.
Configuration, authentication, malformed-response, and API failures return a
non-zero exit code.

## PHP implementation

The framework-neutral
[`tracezilla-shopify-php`](https://github.com/Happy-Bananas/tracezilla-shopify-php)
implementation includes Docker setup, API clients, pagination services,
response mappers, a comparison workflow, table and JSON output, and automated
tests.

After completing the Shopify and tracezilla setup, follow the repository
README to configure `.env` and run:

```bash
docker compose run --rm php php bin/compare-catalogs
```

The default terminal view shows at most ten rows from each result category,
while comparison and summary counts use the complete catalogs.

## Other implementations

TypeScript, Python, and Make.com versions are planned. They will follow the
same comparison rule and result categories.
