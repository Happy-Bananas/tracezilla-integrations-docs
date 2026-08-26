---
title: Deployment
layout: default
nav_order: 80
has_children: true
---

# Deployment

{: .label .label-red }
Unsupported

{: .warning }
Deployment guidance is an optional example. Hosting, server security, backups,
monitoring, and scheduled operation are the developer's or customer's
responsibility.

Cron starts BifrostConnect. BifrostConnect then communicates with
Shopify and tracezilla and applies the customer's business rules.

<pre class="mermaid">
flowchart TB
    Cron[Cron]
    Shopify[Shopify]
    Integration[BifrostConnect]
    Tracezilla[tracezilla]

    Cron --> Integration
    Shopify <--> Integration
    Integration <--> Tracezilla
</pre>

See the unsupported [Hetzner example](./deployment/hetzner.html) for one
possible installation and [Cron](./deployment/cron.html) for scheduling.
