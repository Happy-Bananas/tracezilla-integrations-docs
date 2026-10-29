---
title: REST API Keys
layout: default
parent: WooCommerce Prerequisites
grand_parent: WooCommerce
nav_order: 20
---

# REST API keys

The integration reads and writes WooCommerce through its REST API. Create a
key pair for it in the store from [Development Store](./store.html).

In WordPress administration, select:

**WooCommerce → Settings → Advanced → REST API**

For the local sandbox, you can also open the
[REST API settings directly](http://localhost:8080/wp-admin/admin.php?page=wc-settings&tab=advanced&section=keys).

Select **Create an API key** or **Add key**, then enter:

| Field | Suggested value |
|:--|:--|
| Description | `Tracezilla local development` |
| User | `admin` |
| Permissions | `Read/Write` |

Select **Generate API key** and copy both the Consumer Key and Consumer Secret
immediately. WooCommerce displays the complete Consumer Secret only once.

Read-only credentials are sufficient for catalog comparison. The example uses
Read/Write credentials because later commands will create or update test data.
Use only the permissions required by the integration in a production store.

Keep both values for the `.env` file of the integration. You add them when you
install it.

## Next step

Prepare [tracezilla](../../tracezilla-fundamentals.html) if you have not done
so, then install the integration [with Docker](../install-with-docker.html) or
[without Docker](../install-without-docker.html).

## Official WooCommerce reference

- [WooCommerce REST API key setup](https://woocommerce.com/document/woocommerce-rest-api/)
