---
title: Deployment
layout: default
nav_order: 80
has_children: true
---

# Deployment

The headless integration is designed to run anywhere PHP and cron are
available, including modest shared-hosting environments. A deployment does not
require Laravel, a web server, a queue service, Redis, or a SQL database for
its basic scheduling and retry model.

The deployment environment must provide:

- A supported PHP runtime and the installed Composer dependencies.
- Environment variables or an uncommitted `.env` file containing credentials.
- Cron or an equivalent scheduler.
- One private, persistent directory writable by the same operating-system user
  that executes the cron command.
- A way to retain logs or notify an operator when attention is required.

Start with [Cron](./deployment/cron.html) for the scheduling model. Its child
page [Cron errors and retries](./deployment/cron/errors.html) describes
failure recovery without requiring a database.

{: .important }
Atomic locking, file retry storage, deployment checks, history, and failure
management are implemented. Persistent automatic retry is currently connected
to individual-order imports. Verify retry coverage on each other workflow
before relying on it for unattended writes.
