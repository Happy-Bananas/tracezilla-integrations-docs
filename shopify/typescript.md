---
title: TypeScript
layout: default
parent: Shopify
nav_order: 40
has_children: true
---

# Build a Shopify integration with TypeScript

{: .label .label-green }
Framework neutral

[`tracezilla-shopify-typescript`](https://github.com/Happy-Bananas/tracezilla-shopify-typescript)
is a runnable Node.js 22 and TypeScript template without a web framework.

## Clone and start the project

You need Git and Docker with the Docker Compose plugin. Node.js does not need
to be installed on the host.

```bash
git clone https://github.com/Happy-Bananas/tracezilla-shopify-typescript.git
cd tracezilla-shopify-typescript
cp .env.example .env
```

Complete [Shopify Setup](./setup.html) and
[tracezilla authentication](../fundamentals/authentication.html), then add the
test credentials to `.env`. Never commit that file.

```bash
docker compose build
```

The source is copied into the image, so rebuild after changing source code or
dependencies.

## Available commands

- [Compare Catalogs](./typescript/compare-catalogs.html) — compare Shopify
  variants and tracezilla SKUs without changing either system.

## Tests and type checking

```bash
docker compose run --rm app npm test
docker compose run --rm app npm run typecheck
```

Tests use in-memory readers and do not contact either API.

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
    ShopifyMapper --> Model[Shared CatalogItem]
    TracezillaMapper --> Model
    Model --> Workflow[CompareCatalogs workflow]
    Workflow --> Output[Table or JSON output]
</pre>

| Layer | Location | Responsibility |
|---|---|---|
| Entry points | `src/cli/` | Parse options, assemble dependencies, invoke one workflow, and choose output |
| Configuration | `src/configuration.ts` | Validate environment variables and endpoint settings |
| Queries | `src/shopify/queries/` | Keep Shopify GraphQL documents separate |
| Clients | `src/shopify/shopify-client.ts`, `src/tracezilla/tracezilla-client.ts` | Own authentication and HTTP transport |
| Services | `*-catalog-service.ts` | Retrieve and paginate API data |
| Mappers | `*-mapper.ts` | Convert vendor responses to shared models |
| Shared models | `src/shared/` | Define concepts used by both APIs |
| Workflows | `src/workflows/` | Hold integration behavior without HTTP concerns |
| Output | `src/output/` | Render structured results |
| Tests | `tests/` | Verify behavior without live credentials |

The code uses TypeScript modules and the Node.js Fetch API. Clients should not
contain mapping rules; services should not format terminal output; workflows
should not know about HTTP, Docker, or environment files.

## Create another command

Add or extend the query, client, service, mapper, shared model, and workflow
needed by the use case. Then create a focused file under `src/cli/` that wires
those parts together. Add tests and document the command as a child beneath
TypeScript. Write commands should default to dry run and require explicit
confirmation.

## Use the code elsewhere

The modules can be reused in an Express application, worker, serverless
function, or scheduled process. The host should replace composition,
configuration, and output while retaining tested clients, services, mappers,
and workflow rules.
