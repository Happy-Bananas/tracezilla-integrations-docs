---
title: C# / .NET
layout: default
parent: Shopify
nav_order: 60
has_children: true
---

# Build a Shopify integration with C# and .NET

{: .label .label-green }
Framework neutral

[`tracezilla-shopify-dotnet`](https://github.com/Happy-Bananas/tracezilla-shopify-dotnet)
is a runnable .NET 8 console template without ASP.NET.

## Clone and start the project

You need Git and Docker with Docker Compose; the .NET SDK is not required on
the host.

```bash
git clone https://github.com/Happy-Bananas/tracezilla-shopify-dotnet.git
cd tracezilla-shopify-dotnet
cp .env.example .env
```

Complete [Shopify Setup](./setup.html) and
[tracezilla authentication](../fundamentals/authentication.html), add test
credentials to `.env`, then build:

```bash
docker compose build
```

Never commit `.env`. Rebuild after code or dependency changes.

## Available commands

- [Compare Catalogs](./dotnet/compare-catalogs.html) — compare catalogs by SKU
  without modifying either API.

## Run the tests

```bash
docker compose run --rm --entrypoint dotnet app test tests/TracezillaShopify.Tests --no-restore
```

Tests use fake clients and readers. Nullable reference types and warnings as
errors are enabled in the project.

## Architecture

<pre class="mermaid">
flowchart TB
    subgraph Shopify[Shopify boundary]
        Query[GraphQL query] --> ShopifyService[Catalog service]
        ShopifyClient[API client] --> ShopifyService
        ShopifyService --> ShopifyMapper[Variant mapper]
    end
    subgraph Tracezilla[tracezilla boundary]
        TracezillaClient[API client] --> TracezillaService[Catalog service]
        TracezillaService --> TracezillaMapper[SKU mapper]
    end
    ShopifyMapper --> Model[CatalogItem]
    TracezillaMapper --> Model
    Model --> Workflow[CompareCatalogs]
    Workflow --> Output[Table or JSON]
</pre>

| Layer | Location | Responsibility |
|---|---|---|
| Entry point | `src/TracezillaShopify/Program.cs` | Parse options, register dependencies, run the workflow, and select output |
| Configuration | `Configuration.cs` | Validate environment and endpoints |
| Shopify boundary | `Shopify/GetProductVariants.cs`, `ShopifyClient.cs`, `ShopifyCatalog.cs` | Query, authenticate, paginate, map variants |
| tracezilla boundary | `Tracezilla/TracezillaClient.cs`, `TracezillaCatalog.cs` | Retrieve, paginate, and map SKUs |
| Shared contracts | `Shared/` | Define `CatalogItem` and catalog reader behavior |
| Workflow | `Workflows/CompareCatalogs.cs` | Compare normalized items independently of APIs |
| Output | `Output/TableRenderer.cs` | Render results |
| Tests | `tests/TracezillaShopify.Tests/` | Verify workflow and mappings using fakes |

Interfaces and constructor dependencies keep transport replaceable and make
workflow tests independent of the network.

## Create another command

Add focused client operations, services, mappers, models, and a workflow. Keep
`Program.cs` as composition rather than business logic. Add tests and document
the command beneath C# / .NET. Write operations should default to dry run and
require an explicit confirmation.

## Use the code elsewhere

The classes can be reused in ASP.NET, a Worker Service, Azure Functions, or a
scheduled console application. The host should change composition and
configuration rather than duplicating API and mapping logic.
