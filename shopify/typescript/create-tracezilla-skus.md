---
title: Create tracezilla SKUs
layout: default
parent: TypeScript
grand_parent: Shopify
nav_order: 20
---
# Create tracezilla SKUs from Shopify
{: .label .label-yellow }
Writes with explicit confirmation
## Behavior
Reads Shopify variants and existing tracezilla SKUs, then reports `would_create`, `created`, `skipped`, `invalid`, or `failed`. The mapper visibly uses example values `pcs`, `colli`, and `1.0`; adapt them to the customer.
## Run the command
```bash
docker compose run --rm --entrypoint npm app run create-skus -- --limit=10
```
## Options
Use `--limit=N` to bound processing and `--json` for JSON output.
## Safety and exit status
Dry run is the default. A write requires both flags:
```bash
docker compose run --rm --entrypoint npm app run create-skus -- --execute --confirm --limit=1
```
Failures return non-zero; the command never writes to Shopify.
## Architecture
<pre class="mermaid">
flowchart TB
 subgraph Shopify[Shopify boundary]
  Query[GraphQL query] --> Service[Catalog service] --> Variant[Variant data]
 end
 subgraph Tracezilla[tracezilla boundary]
  Client[API client] --> Target[SKU service]
  Payload[SKU payload] --> Target
 end
 Variant --> Workflow[CreateTracezillaSkus]
 Target --> Workflow --> Payload
</pre>
## Implementation
Entry point: `src/cli/create-tracezilla-skus.ts`; workflow and mapping: `src/workflows/create-tracezilla-skus.ts`; API services: `src/shopify/` and `src/tracezilla/`.
## Tests
```bash
docker compose run --rm --entrypoint npm app test
docker compose run --rm --entrypoint npm app run typecheck
```
