---
title: Home
layout: home
nav_order: 1
---

# BifrostConnect

BifrostConnect bridges external services and tracezilla, carrying your custom
business rules safely between them. An external service can be any system with
an API: a webshop, a marketplace, an accounting or ERP system, a shipping
provider, and so on.

<pre class="mermaid">
flowchart TB
    Service["External service<br>Shopify, WooCommerce, ..."]
    Bifrost["BifrostConnect<br>Custom business logic in PHP"]
    Tracezilla[tracezilla]

    Service <--> Bifrost
    Bifrost <--> Tracezilla
</pre>

Each external service has a standalone headless PHP application. Available
today:

| Service | Project | Status |
|:--|:--|:--|
| [Shopify](./shopify/) | `tracezilla-shopify-php` | Catalog, inventory, and order commands |
| [WooCommerce](./woocommerce/) | `tracezilla-woocommerce-php` | Connection check and test-data seed; more commands planned |

Write the customer's rules in PHP, verify them with tests, and run the same
command manually or from cron. Start with [Getting Started](./getting-started.html).
