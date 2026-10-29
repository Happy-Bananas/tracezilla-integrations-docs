---
title: Home
layout: home
nav_order: 1
---

# BifrostConnect

BifrostConnect is an open-source starting point for **building your own
connection** between tracezilla and an external service, and **hosting it
yourself**. An external service can be any system with an API: a webshop, a
marketplace, an accounting or ERP system, a shipping provider, and so on.

{: .important }
BifrostConnect is not a hosted service. Nobody runs it for you: you or your
customer install it on a server you control, and **cron starts it** on a
schedule. Nothing is exchanged between the systems unless your server runs a
command.

<pre class="mermaid">
flowchart LR
    subgraph Server["Your server"]
        Cron["Cron<br>every few minutes"]
        Bifrost["BifrostConnect<br>your business rules in PHP"]
        Cron --> Bifrost
    end
    Service["External service<br>Shopify, WooCommerce, ..."]
    Tracezilla[tracezilla]

    Service <--> Bifrost
    Bifrost <--> Tracezilla
</pre>

## How it works

1. **Build the connection.** Start from the template project for the external
   service, write the customer's business rules in PHP, and verify them with
   tests on your own computer. A scaffold command, `scenario:create`, generates
   the files for a new feature with a working example and its test
   ([Shopify](./shopify/custom-business-logic.html); planned for WooCommerce).
2. **Host it yourself.** Copy the finished project to a server you or the
   customer control. It is a plain console application: PHP and a `.env` file
   with the credentials; no web server, database, or open port is required.
3. **Trigger it with cron.** BifrostConnect does not run in the background and
   does not receive anything by itself. A cron task on the server starts a
   command at a regular interval, for example every five minutes. Each run reads
   what has changed, applies the rules, writes the results, and stops.

The same command you run by hand during development is the one cron runs in
production.

## Your responsibilities

Because you host it, you also own its operation:

- The server, its security, and updates of PHP and the project.
- The credentials in `.env` and the stored tracezilla token.
- The cron schedule, and monitoring that the runs succeed.
- Backups of the project and its runtime files.

The [Deployment](./deployment.html) guides show one way to do this; they are
examples, not a supported hosting service.

## Available integrations

Each external service has a standalone headless PHP application:

| Service | Project | Status |
|:--|:--|:--|
| [Shopify](./shopify/) | `tracezilla-shopify-php` | Catalog, inventory, and order commands |
| [WooCommerce](./woocommerce/) | `tracezilla-woocommerce-php` | Connection check and test-data seed; more commands planned |

Start with [Getting Started](./getting-started.html).
