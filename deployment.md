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

Cron starts your integration. Your integration then communicates with the
external service and tracezilla and applies the customer's business rules.

<pre class="mermaid">
flowchart TB
    Cron[Cron]
    Service[External service]
    Integration[Your integration]
    Tracezilla[tracezilla]

    Cron --> Integration
    Service <--> Integration
    Integration <--> Tracezilla
</pre>

## Remove the seed before deploying

The [seed](./fundamentals/seed.html) fills a development company with test
data and has no use on a production server. Its `--execute` already refuses to
write unless `TRACEZILLA_SEED_ALLOWED_TEAM` in `.env` names the team, and a
production `.env` never sets it. To leave nothing to chance, remove the command
before the first deployment, in your integration project folder.

Mac and Linux:

```bash
rm bin/seed-tracezilla
```

Windows (PowerShell):

```powershell
Remove-Item bin/seed-tracezilla
```

If you added your own `seeds/` folder, remove it too. Also remove the
`TRACEZILLA_OWN_COMPANY` and `TRACEZILLA_SEED_ALLOWED_TEAM` lines from
`.env.example`. `bin/bifrost-connect` then stops offering `tracezilla:seed`. A
project created with `--without-seed` has no seed to remove.

## Deployment guides

See the unsupported [Hetzner example](./deployment/hetzner.html) for a manual
installation, [Deploy with Ansible](./deployment/ansible.html) for repeatable
deployments, and [Cron](./deployment/cron.html) for scheduling.
