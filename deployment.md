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
monitoring, and scheduled operation are your or your customer's
responsibility.

Cron starts BifrostConnect. BifrostConnect then communicates with the
external service and tracezilla and applies the customer's business rules.

<pre class="mermaid">
flowchart TB
    Cron[Cron]
    Service[External service]
    Integration[BifrostConnect]
    Tracezilla[tracezilla]

    Cron --> Integration
    Service <--> Integration
    Integration <--> Tracezilla
</pre>

See the unsupported [Hetzner example](./deployment/hetzner.html) for a manual
installation, [Deploy with Ansible](./deployment/ansible.html) for repeatable
deployments, and [Cron](./deployment/cron.html) for scheduling.
