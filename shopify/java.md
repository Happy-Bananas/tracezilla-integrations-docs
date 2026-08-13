---
title: Java
layout: default
parent: Shopify
nav_order: 65
has_children: true
---

# Build a Shopify integration with Java

{: .label .label-green }
Framework neutral

[`tracezilla-shopify-java`](https://github.com/Happy-Bananas/tracezilla-shopify-java)
is a runnable Java 21 command-line template without Spring.

## Clone and start the project

You need Git and Docker with Docker Compose. Java and Maven do not need to be
installed on the host.

```bash
git clone https://github.com/Happy-Bananas/tracezilla-shopify-java.git
cd tracezilla-shopify-java
cp .env.example .env
```

Complete [Shopify Setup](./setup.html) and
[tracezilla authentication](../fundamentals/authentication.html), then add test
credentials to `.env` and build:

```bash
docker compose build
```

Never commit `.env`. Rebuild after code or Maven dependency changes.

## Available commands

- [Compare Catalogs](./java/compare-catalogs.html) — compare Shopify variants
  and tracezilla SKUs without changing either system.
- [Create tracezilla SKUs](./java/create-tracezilla-skus.html) — preview or create missing tracezilla SKUs.

## Run the tests

```bash
docker compose run --rm --entrypoint mvn app test
```

JUnit tests exercise comparison behavior without contacting live APIs. Maven
also compiles with lint warnings treated as errors.

## Architecture

<pre class="mermaid">
flowchart TB
    subgraph Shopify[Shopify boundary]
        Query[GetProductVariants query] --> ShopifyCatalog[Catalog reader]
        ShopifyClient[API client] --> ShopifyCatalog
    end
    subgraph Tracezilla[tracezilla boundary]
        TracezillaCatalog[Catalog reader]
    end
    ShopifyCatalog --> Model[Shared CatalogItem]
    TracezillaCatalog --> Model
    Model --> Workflow[CompareCatalogs]
    Workflow --> Main[CLI table or JSON output]
</pre>

| Layer | Location | Responsibility |
|---|---|---|
| Entry point | `Main.java` | Parse options, assemble objects, invoke the workflow, and render output |
| Configuration | `Configuration.java` | Validate environment and endpoints |
| Shopify query | `shopify/GetProductVariants.java` | Store the GraphQL document separately |
| Shopify client | `shopify/ShopifyClient.java` | Authenticate and execute GraphQL over HTTP |
| Catalog readers | `shopify/ShopifyCatalog.java`, `tracezilla/TracezillaCatalog.java` | Retrieve, paginate, and normalize API data |
| Shared contracts | `shared/CatalogItem.java`, `CatalogReader.java` | Define common catalog data and reader behavior |
| Workflow | `workflow/CompareCatalogs.java` | Compare normalized records without transport knowledge |
| Tests | `src/test/` | Verify comparison behavior without live credentials |

The current compact implementation performs mapping inside its
catalog readers. As commands grow, extract dedicated mapper and service classes
instead of adding more responsibilities to those readers. The query, client,
shared contract, and workflow boundaries already provide the seams for that
refactoring.

## Create another command

Add named query classes, focused client methods, services for pagination,
dedicated mappers for vendor records, shared models, and a workflow for the
business rule. Keep `Main` limited to CLI composition. Add JUnit tests and a
child page beneath Java. Write commands should default to dry run and require
explicit confirmation.

## Use the code elsewhere

The ordinary Java classes can be reused in Spring, Quarkus, a scheduled job, or
a serverless handler. A host framework should replace composition and
configuration while retaining tested API and workflow boundaries.
