---
title: List Shopify Locations
layout: default
parent: Shopify
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

The GraphQL location ID identifies the inventory location in later Shopify
inventory commands.

## Run the command

From your project directory, run:

### With Docker

```bash
docker compose exec integration php bin/bifrost-connect shopify:locations
```

### Without Docker

```bash
php bin/list-shopify-locations
```

The terminal table lists every returned location. If the app cannot see any
locations, the command reports that explicitly and still exits successfully.

## Options

Return the complete structured result as JSON:

```bash
docker compose exec integration php bin/bifrost-connect shopify:locations --json
```

There is no processing limit. The command follows Shopify pagination until all
locations have been retrieved; stores normally have a small number of locations.

## Safety and exit status

The command is read-only and never modifies Shopify or tracezilla. It requires
Shopify `read_locations` access.

A successful response—including an empty location list—returns exit code `0`.
Configuration, authentication, GraphQL, malformed-response, and invalid
pagination errors return a non-zero exit code.

## Architecture

<pre class="mermaid">
flowchart TB
    subgraph Shopify[Shopify boundary]
        Query[GetLocations GraphQL query] --> Service[ShopifyLocationService]
        Client[ShopifyClient] --> Service
        Service --> Mapper[ShopifyLocationMapper]
    end
    Mapper --> Model[ShopifyLocation]
    Model --> Workflow[ListShopifyLocations workflow]
    Workflow --> Result[Structured location result]
    Result --> Table[Table output]
    Result --> Json[JSON output]
</pre>

## Implementation

| Responsibility | Source |
|---|---|
| CLI options, composition, output, and exit status | `bin/list-shopify-locations` |
| GraphQL document | `src/Shopify/Queries/GetLocations.php` |
| Authentication and GraphQL transport | `src/Shopify/ShopifyClient.php` |
| Pagination and response validation | `src/Shopify/ShopifyLocationService.php` |
| API response mapping | `src/Shopify/Mappers/ShopifyLocationMapper.php` |
| Typed location and serialization | `src/Shopify/ShopifyLocation.php` |
| Application operation | `src/Workflows/ListShopifyLocations.php` |
| Terminal formatting | `src/Output/LocationTableRenderer.php` |

The service owns pagination, the mapper translates Shopify fields, and the
workflow returns structured application data. The entry point only assembles
those pieces and selects table or JSON output.

## Tests

```bash
docker compose exec integration composer test
```

The location tests verify mapping, addresses, activity flags, serialization,
and terminal output without contacting Shopify. A sandbox run should also be
used to verify that the app can access the expected locations.
