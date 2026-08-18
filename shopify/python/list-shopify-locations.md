---
title: List Shopify Locations
layout: default
parent: Python
grand_parent: Shopify Labs
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

Complete the [Python installation and configuration](../python.html) and ensure the app has `read_locations` access.

## Run the command

### With Docker

```bash
docker compose run --rm --entrypoint list-shopify-locations app
```

### Without Docker

```bash
list-shopify-locations
```

An empty location list is reported explicitly and is still successful.

## Options

Return structured JSON:

```bash
docker compose run --rm --entrypoint list-shopify-locations app --json
```

The command follows Shopify pagination until every location is retrieved.

## Safety and exit status

This is read-only and never modifies Shopify or tracezilla. Success, including an empty list, returns `0`. Configuration, authentication, GraphQL, response, and pagination errors return non-zero.

## Architecture

<pre class="mermaid">
flowchart TB
 Query[GetLocations GraphQL query] --> Service[ShopifyLocationService]
 Client[ShopifyClient] --> Service --> Mapper[ShopifyLocationMapper]
 Mapper --> Model[ShopifyLocation] --> Result[Structured result]
 Result --> Output[Table or JSON]
</pre>

## Implementation

| Responsibility | Source |
|---|---|
| CLI and output | `src/tracezilla_shopify/list_locations_cli.py` |
| Query, mapping, and pagination | `src/tracezilla_shopify/shopify/location_service.py` |
| Typed model | `src/tracezilla_shopify/shopify/location.py` |
| Authentication and transport | `src/tracezilla_shopify/shopify/client.py` |

The service owns pagination and mapping, the model represents normalized
location data, and the entry point selects table or JSON output. Business code
therefore does not depend on Shopify's response structure.

## Tests

```bash
docker compose run --rm --entrypoint pytest app
docker compose run --rm --entrypoint mypy app src tests
```

Tests verify mapping, addresses, activity flags, serialization, and pagination
without contacting Shopify. A sandbox run verifies access to expected locations.
