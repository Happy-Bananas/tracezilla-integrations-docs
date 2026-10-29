---
title: REST API Keys
layout: default
parent: WooCommerce Prerequisites
grand_parent: WooCommerce
nav_order: 20
---

# REST API keys

The integration reads and writes WooCommerce through its REST API with a
Consumer Key and Consumer Secret.

## Local sandbox

The sandbox has a key preinstalled, and `.env.example` already contains it. No
action is needed:

| Setting | Value |
|:--|:--|
| `WOOCOMMERCE_CONSUMER_KEY` | `ck_sandbox` |
| `WOOCOMMERCE_CONSUMER_SECRET` | `cs_sandbox` |

The key has Read/Write permissions and is recreated every time the sandbox
starts, so it also works after a reset. Never use these values on a real store.

## Existing store

Create a key pair in the store's WordPress administration, as described in the
official [WooCommerce REST API documentation](https://woocommerce.com/document/woocommerce-rest-api/).
Use Read/Write permissions in a test store only, and only the permissions the
integration needs in a production store.

The store must use HTTPS. WooCommerce accepts a Consumer Key and Secret only
over HTTPS.

## Next step

Prepare [tracezilla](../../tracezilla-fundamentals.html) if you have not done
so, then install the integration [with Docker](../install-with-docker.html) or
[without Docker](../install-without-docker.html).
