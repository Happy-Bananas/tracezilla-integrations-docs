---
title: Deployment
layout: default
nav_order: 80
has_children: true
---

# Deployment

The headless integration runs as PHP console commands started manually or by
cron. It is suitable for a single persistent hosting account with SSH access.

## From development to server

The developer uploads the exact project that was tested locally. The server
does not retrieve or track the public template.

<pre class="mermaid">
flowchart TB
    Template[Public template<br>Framework files]
    Project[Developer's customer project]
    Rules[Customer business rules<br>custom/]
    Upload[Secure upload over SSH]
    Server[Deployment server<br>apps/YOUR_PROJECT_NAME]
    Secrets[Credentials entered on server<br>.env]
    Packages[Composer installs packages<br>vendor/]
    Cron[Cron runs the customer command]

    Template --> Project
    Rules --> Project
    Project --> Upload
    Upload --> Server
    Secrets --> Server
    Packages --> Server
    Server --> Cron
</pre>

The files have different origins:

| Files | Source |
|:--|:--|
| Framework files such as `bin/` and `src/` | Initially copied from the public template |
| Customer rules under `custom/` | Written and tested by the developer |
| Complete tested project | Uploaded from the developer's computer with `rsync` |
| `.env` | Created directly on the server; never committed |
| `vendor/` | Created on the server by Composer |
| `var/` | Created and maintained by the running application |

The deployment environment must provide:

- A supported PHP runtime and the installed Composer dependencies.
- Environment variables or an uncommitted `.env` file containing credentials.
- Cron or an equivalent scheduler.
- One private, persistent directory writable by the same operating-system user
  that executes the cron command.
- A way to retain logs or notify an operator when attention is required.

Before deployment, the developer must have completed
[Getting Started](./getting-started.html), added and tested the customer rules,
and confirmed that the project is ready to run.

Follow the [Hetzner deployment example](./deployment/hetzner.html) for a
complete native PHP installation. Then continue with [Cron](./deployment/cron.html)
to schedule the required business command.
