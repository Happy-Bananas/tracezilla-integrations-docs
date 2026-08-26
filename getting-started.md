---
title: Getting Started
layout: default
nav_order: 10
has_children: true
---

# Getting started

BifrostConnect is one headless PHP console application:

<pre class="mermaid">
flowchart TB
    Shopify[Shopify]
    Integration[BifrostConnect]
    Tracezilla[tracezilla]

    Shopify <--> Integration
    Integration <--> Tracezilla
</pre>

It runs manually during development and from cron in production. Customer
business rules are ordinary PHP files generated from a safe starting point.

## Prerequisites

Prepare both test systems before creating the project:

1. [tracezilla test account and credentials](./fundamentals/account-and-team.html)
2. [Shopify test store and credentials](./shopify/setup/authorize-api.html)

## Choose how to run BifrostConnect

- [Run with Docker](./run-with-docker.html) — the quickest setup; PHP and its
  packages run in a prepared development container.
- [Manual Installation](./manual-installation.html) — run BifrostConnect
  directly with PHP and Composer installed on your computer.

Both paths create the same project and finish by checking the Shopify and
tracezilla connections. Choose one path; you do not need to complete both.
