---
title: List Shopify Locations
layout: default
parent: Java
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

Complete the [Java installation and configuration](../java.html) and ensure the app has `read_locations` access.

## Run the command

### With Docker

```bash
docker compose run --rm app list-shopify-locations
```

### Without Docker

```bash
set -a
source .env
set +a
mvn package
java -jar target/tracezilla-shopify-java-0.1.0.jar list-shopify-locations
```

An empty location list is reported explicitly and is still successful.

## Options

Return structured JSON:

```bash
docker compose run --rm app list-shopify-locations --json
```

The command follows Shopify pagination until every location is retrieved.

## Safety and exit status

This is read-only and never modifies Shopify or tracezilla. Success, including an empty list, returns `0`. Configuration, authentication, GraphQL, response, and pagination errors return non-zero.

## Architecture

<pre class="mermaid">
flowchart TB
 Query[GetLocations GraphQL query] --> Service[ShopifyLocationService]
 Client[ShopifyClient] --> Service --> Mapper[Location mapping]
 Mapper --> Model[ShopifyLocation] --> Result[Structured result]
 Result --> Output[Table or JSON]
</pre>

## Implementation

| Responsibility | Source |
|---|---|
| CLI composition and output | `src/main/java/com/happybananas/tracezilla/Main.java` |
| Query, mapping, and pagination | `src/main/java/com/happybananas/tracezilla/shopify/ShopifyLocationService.java` |
| Typed model | `src/main/java/com/happybananas/tracezilla/shopify/ShopifyLocation.java` |
| Authentication and transport | `src/main/java/com/happybananas/tracezilla/shopify/ShopifyClient.java` |

The service owns pagination and mapping, the record represents normalized
location data, and the entry point selects table or JSON output. Business code
therefore does not depend on Shopify's response structure.

## Tests

```bash
docker compose run --rm --entrypoint mvn app test
```

Tests verify mapping without contacting Shopify. The sandbox run verifies
pagination, serialization, and access to the expected locations.
