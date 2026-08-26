---
title: Deployment
layout: default
nav_order: 80
has_children: true
---

# Deployment

The headless integration runs as PHP console commands started manually or by
cron. It is suitable for a single persistent hosting account with SSH access.

The deployment environment must provide:

- A supported PHP runtime and the installed Composer dependencies.
- Environment variables or an uncommitted `.env` file containing credentials.
- Cron or an equivalent scheduler.
- One private, persistent directory writable by the same operating-system user
  that executes the cron command.
- A way to retain logs or notify an operator when attention is required.

Follow the [Hetzner deployment example](./deployment/hetzner.html) for a
complete native PHP installation. Then continue with [Cron](./deployment/cron.html)
to schedule the required business command.

{: .important }
Atomic locking, file retry storage, deployment checks, history, and failure
management are implemented. Persistent automatic retry is currently connected
to individual-order imports. Verify retry coverage on each other workflow
before relying on it for unattended writes.
