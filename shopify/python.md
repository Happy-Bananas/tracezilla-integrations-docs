---
title: Python
layout: default
parent: Shopify
nav_order: 50
has_children: true
---

# Build a Shopify integration with Python

{: .label .label-green }
Framework neutral

[`tracezilla-shopify-python`](https://github.com/Happy-Bananas/tracezilla-shopify-python)
is a runnable Python 3.12 template without Django or Flask.

## Clone and start the project

You need Git and Docker with Docker Compose; Python is not required on the host.

```bash
git clone https://github.com/Happy-Bananas/tracezilla-shopify-python.git
cd tracezilla-shopify-python
cp .env.example .env
```

Complete [Shopify Setup](./setup.html) and
[tracezilla authentication](../fundamentals/authentication.html), add test
credentials to `.env`, then build:

```bash
docker compose build
```

Never commit `.env`. Rebuild after source or dependency changes because the
application is copied into the image.

## Available commands

- [Compare Catalogs](./python/compare-catalogs.html) — compare catalogs by SKU
  without changing either system.
- [Create tracezilla SKUs](./python/create-tracezilla-skus.html) — preview or create missing tracezilla SKUs.

## Tests and type checking

```bash
docker compose run --rm --entrypoint pytest app
docker compose run --rm --entrypoint mypy app src tests
```

Tests use in-memory clients and never contact either API. Dependencies are
locked in `requirements.lock` and mypy runs in strict mode.

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
    Model --> Workflow[Comparison workflow]
    Workflow --> Output[Table or JSON output]
</pre>

| Layer | Location | Responsibility |
|---|---|---|
| Entry point | `src/tracezilla_shopify/cli.py` | Parse options, assemble dependencies, run the workflow, and choose output |
| Configuration | `configuration.py` | Validate environment and endpoint settings |
| Shopify boundary | `shopify/query.py`, `client.py`, `service.py`, `mapper.py` | Query, authenticate, paginate, and normalize Shopify data |
| tracezilla boundary | `tracezilla/client.py`, `service.py`, `mapper.py` | Retrieve, paginate, and normalize tracezilla data |
| Shared model | `shared.py` | Define the common catalog record and reader contract |
| Workflow | `workflow.py` | Compare normalized records independently of APIs |
| Output | `output.py` | Render human and machine-readable results |
| Tests | `tests/` | Test each boundary with in-memory inputs |

Keep vendor payloads behind mappers and services. Business behavior belongs in
the workflow, while `cli.py` should remain a small composition layer.

## Create another command

Extend only the layers required by the use case, add a focused CLI entry point,
and cover mapping and workflow rules with tests. Document every command beneath
Python. Any write command should default to dry run, preview changes, impose a
limit, and require explicit confirmation.

## Use the code elsewhere

The package can be reused in Django, Flask, a queue worker, scheduled task, or
serverless function. Framework adapters should replace configuration and
composition rather than moving API or mapping rules into controllers.
