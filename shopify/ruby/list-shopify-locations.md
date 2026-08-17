---
title: List Shopify Locations
layout: default
parent: Ruby
grand_parent: Shopify
nav_order: 30
---

# List Shopify locations

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

Complete the [Ruby installation and configuration](../ruby.html) and ensure the app has `read_locations` access.

## Run the command

### With Docker

```bash
docker compose run --rm --entrypoint bundle app exec ruby bin/list_shopify_locations
```

### Without Docker

```bash
bundle exec ruby bin/list_shopify_locations
```

An empty location list is reported explicitly and is still successful.

## Options

Return structured JSON:

```bash
docker compose run --rm --entrypoint bundle app exec ruby bin/list_shopify_locations --json
```

The command follows Shopify pagination until every location is retrieved.

## Safety and exit status

This is read-only and never modifies Shopify or tracezilla. Success, including an empty list, returns `0`. Configuration, authentication, GraphQL, response, and pagination errors return non-zero.

## Architecture

<pre class="mermaid">
flowchart TB
 Query[GetLocations GraphQL query] --> Service[LocationService]
 Client[Shopify client] --> Service --> Mapper[LocationMapper]
 Mapper --> Model[Location] --> Result[Structured result]
 Result --> Output[Table or JSON]
</pre>

## Implementation

| Responsibility | Source |
|---|---|
| CLI and output | `bin/list_shopify_locations` |
| Query, mapping, and pagination | `lib/tracezilla_shopify/shopify/location_service.rb` |
| Typed model | `lib/tracezilla_shopify/shopify/location.rb` |
| Authentication and transport | `lib/tracezilla_shopify/shopify/client.rb` |

The service owns pagination and mapping, the model represents normalized
location data, and the entry point selects table or JSON output. Business code
therefore does not depend on Shopify's response structure.

## Tests

```bash
docker compose run --rm --entrypoint bundle app exec rake test
```

Tests verify mapping, addresses, activity flags, serialization, and pagination
without contacting Shopify. A sandbox run verifies access to expected locations.
