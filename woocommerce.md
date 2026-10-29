---
title: WooCommerce
layout: default
nav_order: 50
has_children: true
permalink: /woocommerce/
---

# WooCommerce

Connect WooCommerce with tracezilla using the standalone PHP integration
`tracezilla-woocommerce-php`. It accesses WooCommerce through its REST API and
does not need to be installed as a WordPress plugin.

## Start here

1. Prepare [tracezilla](../tracezilla-fundamentals.html): a development company
   and an OAuth client.
2. Complete the [WooCommerce prerequisites](./setup.html): a development store
   and REST API keys.
3. Install the integration [with Docker](./install-with-docker.html) or
   [without Docker](./install-without-docker.html).
4. [Check both connections](./check-connections.html). Nothing else works until
   this passes.
5. [Seed the test dataset](../fundamentals/seed.html) into your development
   company.

## Coverage

| Command | Status |
|:--|:--|
| [Check Connections](./check-connections.html) | Available |
| [Seed the Test Dataset](../fundamentals/seed.html) | Available |
| Catalog, inventory, and order commands | Planned, matching the Shopify commands |
