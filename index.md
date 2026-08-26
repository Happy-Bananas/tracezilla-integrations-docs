---
title: Home
layout: home
nav_order: 1
---

# BifrostConnect

BifrostConnect bridges commerce platforms and tracezilla, carrying your custom
business rules safely between them.

<pre class="mermaid">
flowchart TB
    Commerce["Commerce service<br>Shopify, WooCommerce, ..."]
    Bifrost["BifrostConnect<br>Custom business logic in PHP"]
    Tracezilla[tracezilla]

    Commerce <--> Bifrost
    Bifrost <--> Tracezilla
</pre>

BifrostConnect is a standalone headless PHP application. Create a platform
scenario, write the customer's rules in PHP, verify them with tests, and run
the same command manually or from cron. Start with [Getting Started](./getting-started/).
