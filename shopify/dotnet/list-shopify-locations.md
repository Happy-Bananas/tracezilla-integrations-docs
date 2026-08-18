---
title: List Shopify Locations
layout: default
parent: C# / .NET
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

Complete the [C# / .NET installation and configuration](../dotnet.html) and ensure the app has `read_locations` access.

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
dotnet run --project src/TracezillaShopify -- list-shopify-locations
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
| CLI composition and output | `src/TracezillaShopify/Program.cs` |
| Query, mapping, model, and pagination | `src/TracezillaShopify/Shopify/ShopifyLocations.cs` |
| Authentication and transport | `src/TracezillaShopify/Shopify/ShopifyClient.cs` |

The service owns pagination and mapping, the records represent normalized
location data, and the entry point selects table or JSON output. Business code
therefore does not depend on Shopify's response structure.

## Tests

```bash
docker compose run --rm --entrypoint dotnet app test tests/TracezillaShopify.Tests --no-restore
```

Tests verify mapping, addresses, activity flags, serialization, and pagination
without contacting Shopify. A sandbox run verifies access to expected locations.
