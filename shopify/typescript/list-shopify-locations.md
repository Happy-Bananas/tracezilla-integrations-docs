---
title: List Shopify Locations
layout: default
parent: TypeScript
grand_parent: Experimental Implementations
nav_order: 30
---

# List Shopify locations

{: .label .label-yellow }
Experimental

{: .label .label-green }
Read only

## Behavior

The command retrieves every location visible to the configured Shopify app.
For each location it reports:

- GraphQL and legacy IDs;
- location name and address;
- active or inactive status;
- whether it has active inventory;
- whether it fulfils online orders.

The GraphQL ID identifies the location in later inventory commands.

Complete the [TypeScript installation and configuration](../typescript.html) and ensure the app has `read_locations` access.

## Run the command

### With Docker

```bash
docker compose run --rm --entrypoint npm app run locations --
```

### Without Docker

```bash
npm run locations --
```

An empty location list is reported explicitly and is still successful.

## Options

Return structured JSON:

```bash
docker compose run --rm --entrypoint npm app run locations -- --json
```

The command follows Shopify pagination until every location is retrieved.

## Safety and exit status

This is read-only and never modifies Shopify or tracezilla. Success, including an empty list, returns `0`. Configuration, authentication, GraphQL, response, and pagination errors return non-zero.

## Architecture

<pre class="mermaid">
flowchart TB
 Query[GetLocations GraphQL query] --> Service[ShopifyLocationService]
 Client[ShopifyClient] --> Service --> Mapper[ShopifyLocationMapper]
 Mapper --> Model[ShopifyLocation] --> Workflow[ListShopifyLocations]
 Workflow --> Result[Structured result] --> Output[Table or JSON]
</pre>

## Implementation

| Responsibility | Source |
|---|---|
| CLI and output | `src/cli/list-shopify-locations.ts` |
| Query | `src/shopify/queries/get-locations.ts` |
| Pagination | `src/shopify/shopify-location-service.ts` |
| Mapping and model | `src/shopify/shopify-location-mapper.ts`, `src/shopify/shopify-location.ts` |
| Workflow | `src/workflows/list-shopify-locations.ts` |

The service owns pagination, the mapper validates and translates Shopify
fields, and the workflow returns structured application data. The entry point
only composes these pieces and selects table or JSON output.

## Tests

```bash
docker compose run --rm --entrypoint npm app test
docker compose run --rm --entrypoint npm app run typecheck
```

Tests verify mapping, addresses, activity flags, serialization, and pagination
without contacting Shopify. A sandbox run verifies access to expected locations.
