---
title: Home
layout: home
nav_order: 1
---

# BifrostConnect

{: .warning }
Do not start a customer project with `git clone` of a BifrostConnect
repository. A clone stays connected to the public repository, so the customer's
business rules could end up there. Use the
[create command](#start-a-new-project) instead.

BifrostConnect is for **developers who build integrations between tracezilla
and external services for tracezilla customers**. An external service can be
any system with an API: a webshop, a marketplace, an accounting or ERP system,
a shipping provider, and so on.

It is an open-source starting point: you **build the connection** for your
customer in PHP, and you or the customer **host it yourselves**. You need to be
comfortable with PHP and the command line; this manual covers the rest, from
setting up your computer to running the integration on a server.

{: .important }
BifrostConnect is not a hosted service. Nobody runs your integration for you:
you or your customer install it on a server you control, and **cron starts it** on a
schedule. Nothing is exchanged between the systems unless your server runs a
command.

<pre class="mermaid">
flowchart LR
    subgraph Server["Your server"]
        Cron["Cron<br>every few minutes"]
        Bifrost["Your integration<br>built with BifrostConnect,<br>your business rules in PHP"]
        Cron --> Bifrost
    end
    Service["External service<br>Shopify, WooCommerce, ..."]
    Tracezilla[tracezilla]

    Service <--> Bifrost
    Bifrost <--> Tracezilla
</pre>

## How it works

1. **Build the connection.** [Create a project](#start-a-new-project) for the
   external service, write the customer's business rules in PHP, and verify them with
   tests on your own computer. A scaffold command, `scenario:create`, generates
   the files for a new feature with a working example and its test
   ([Shopify](./shopify/custom-business-logic.html),
   [WooCommerce](./woocommerce/scenarios.html)).
2. **Host it yourself.** Copy the finished project to a server you or the
   customer control. It is a plain console application: PHP and a `.env` file
   with the credentials; no web server, database, or open port is required.
3. **Trigger it with cron.** Your integration does not run in the background and
   does not receive anything by itself. A cron task on the server starts a
   command at a regular interval, for example every five minutes. Each run reads
   what has changed, applies the rules, writes the results, and stops.

The same command you run by hand during development is the one cron runs in
production.

## Start a new project

Create every customer project with this command. Replace `woocommerce` with
`shopify` for a Shopify project, and `YOUR_INTEGRATION_NAME` with the name of
your integration:

```bash
curl -fsSL https://happy-bananas.github.io/tracezilla-integrations-docs/create-project | sh -s -- woocommerce YOUR_INTEGRATION_NAME
```

On Windows, use the command in [Getting Started](./getting-started.html#4-create-the-project).

The command gives you a plain folder that belongs to you and your customer,
with no Git history and no connection to the BifrostConnect repositories. If
you want version control, put it in your own private Git repository.

## Your responsibilities

Because you host it, you also own its operation:

- The server, its security, and updates of PHP and the project.
- The credentials in `.env` and the stored tracezilla token.
- The cron schedule, and monitoring that the runs succeed.
- Backups of the project and its runtime files.

The [Deployment](./deployment.html) guides show one way to do this; they are
examples, not a supported hosting service.

## Available integrations

Each external service has a standalone PHP console application (no web server needed):

| Service | Project | Status |
|:--|:--|:--|
| [Shopify](./shopify/) | `tracezilla-shopify-php` | Connection check, test-data seed, and catalog, inventory, and order commands |
| [WooCommerce](./woocommerce/) | `tracezilla-woocommerce-php` | Connection check, test-data seed, and scenarios |

Start with [Getting Started](./getting-started.html).
