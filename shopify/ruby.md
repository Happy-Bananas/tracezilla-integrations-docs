---
title: Ruby
layout: default
parent: Shopify
nav_order: 55
has_children: true
---

# Build a Shopify integration with Ruby

{: .label .label-green }
Framework neutral

[`tracezilla-shopify-ruby`](https://github.com/Happy-Bananas/tracezilla-shopify-ruby)
is a runnable Ruby 3.4 template without Rails.

## Clone and start the project

You need Git. Use either Docker with Docker Compose, or install Ruby 3.4 and
Bundler directly on the host.

```bash
git clone https://github.com/Happy-Bananas/tracezilla-shopify-ruby.git
cd tracezilla-shopify-ruby
cp .env.example .env
```

Complete [Shopify Setup](./setup.html) and
[tracezilla authentication](../fundamentals/authentication.html), then add test
credentials to `.env`.

### With Docker

```bash
docker compose build
```

### Without Docker

Install the locked gems directly:

```bash
bundle install
```

Never commit `.env`. Rebuild after source or gem changes.

## Available commands

- [Compare Catalogs](./ruby/compare-catalogs.html) — compare catalogs by SKU
  without changing either service.
- [Create tracezilla SKUs](./ruby/create-tracezilla-skus.html) — preview or create missing tracezilla SKUs.

## Run the tests

```bash
docker compose run --rm --entrypoint bundle app exec rake test
```

Tests use fake clients and readers and never contact live APIs.

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
| Entry points | `bin/` | Parse CLI options and compose one command |
| Configuration | `lib/tracezilla_shopify/configuration.rb` | Validate environment and endpoints |
| HTTP | `http_json.rb` | Provide reusable JSON transport behavior |
| Shopify boundary | `shopify/query.rb`, `client.rb`, `catalog_service.rb`, `variant_mapper.rb` | Query, authenticate, paginate, and normalize variants |
| tracezilla boundary | `tracezilla/client.rb`, `catalog_service.rb`, `sku_mapper.rb` | Retrieve, paginate, and normalize SKUs |
| Shared model | `catalog_item.rb` | Represent the cross-system catalog item |
| Workflow | `compare_catalogs.rb` | Apply comparison rules without API knowledge |
| Output | `table_renderer.rb` | Render results for people |
| Tests | `test/` | Verify isolated behavior with fakes |

This organization keeps Rails, HTTP, and CLI concerns out of business rules.
Explicit objects also make transformations visible and independently testable.

## Create another command

Reuse the existing HTTP and client boundaries, add named queries and mappers,
put orchestration in a workflow object, and keep the `bin/` file small. Add
tests and a child page beneath Ruby. Write commands should be dry-run-first and
require explicit confirmation.

## Use the code elsewhere

The library classes can be loaded by Rails, Sinatra, a job processor, or a
scheduled script. The host application should supply configuration and
composition while retaining tested service, mapper, and workflow objects.
