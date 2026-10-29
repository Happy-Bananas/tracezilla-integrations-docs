---
title: WooCommerce Prerequisites
layout: default
parent: WooCommerce
nav_order: 2
has_children: true
---

# WooCommerce prerequisites

Create a safe WooCommerce development store, fill it with predictable test
data, and create REST API credentials for the integration.

## Setup path

Complete these guides in order:

1. [Development Store](./setup/store.html): use the disposable local sandbox or
   an existing test store.
2. [REST API Keys](./setup/rest-api-keys.html): create the Consumer Key and
   Consumer Secret.

## Expected result

At the end of setup, you should have:

- The local sandbox, or an existing WooCommerce store you are authorized to
  test with.
- A WooCommerce REST API Consumer Key and Consumer Secret.

Use development data throughout this section. Do not test write commands in a
customer's production store.

Then install the integration [with Docker](./install-with-docker.html) or
[without Docker](./install-without-docker.html).
