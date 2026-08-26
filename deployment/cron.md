---
title: Cron
layout: default
parent: Deployment
nav_order: 10
has_children: true
---

# Schedule BifrostConnect with cron

{: .label .label-red }
Unsupported

{: .warning }
This deployment guidance is an optional example. Hosting, security, backups,
monitoring, and scheduled operation remain the developer's or customer's
responsibility.

Cron is the primary production trigger. It starts a console command at a
regular interval; BifrostConnect then communicates with the
commerce service and tracezilla and applies the customer's PHP business rules.

<pre class="mermaid">
flowchart TB
    Cron[Cron starts scheduled run]
    Retry[Retry due failed tasks]
    Reconcile[Find new and recently changed records]
    Rules[Apply customer PHP rules]
    APIs[Read from or write to commerce service and tracezilla]
    Result[Store results and failures]

    Cron --> Retry --> Reconcile --> Rules --> APIs --> Result
</pre>

Cron contains no integration business logic. It only selects the command,
schedule, working directory, and log destination.

## What to do next

Consultants normally need only these two pages:

- [Set up a cron task](./cron/setup.html) — copy, test, and schedule the
  headless command on a PHP host.
- [Troubleshoot scheduled runs](./cron/troubleshooting.html) — read the history
  first, inspect quarantined files, and identify tasks that need attention.

The application automatically prevents overlapping workflow commands. Failed
individual-order writes are stored and considered again by later cron
reconciliations according to their retry time. Business problems stop in the
attention list instead of retrying forever.

Framework maintainers can read [Cron error and retry internals](./cron/errors.html)
for the retry implementation details.
