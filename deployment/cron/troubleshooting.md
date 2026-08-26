---
title: Troubleshoot Scheduled Runs
layout: default
parent: Cron
grand_parent: Deployment
nav_order: 20
---

# Troubleshoot scheduled runs

Start with the files written by the integration. Do not begin by changing cron
or deleting retry files.

## 1. Read the history log

History is stored as one JSON event per line:

```text
var/history/YYYY-MM.ndjson
```

Display the latest events:

```bash
tail -n 50 var/history/$(date -u +%Y-%m).ndjson
```

Look for events such as:

- `resolved` — the work completed or was already present safely.
- `attention_required` — automatic retries stopped and a person must correct
  a business or persistent technical problem.
- `retried_manually` — an operator requested another attempt.
- `dismissed` — an operator deliberately removed the task.
- `corrupt_task_quarantined` — a task file could not be read safely.

## 2. Look in the corrupt folder

Unreadable task files are moved out of normal processing:

```bash
find var/retry/corrupt -maxdepth 1 -type f -name '*.json' -print
```

Do not move a file back into `pending` or edit it in place. Preserve it for a
developer to inspect alongside the corresponding history event. Other valid
tasks continue running.

## 3. List current failures

Use the supported command rather than opening pending task files:

```bash
php bin/tracezilla-integration failures:list
```

Tasks under `pending` have a future or due automatic retry. Tasks under
`attention` require a correction before another attempt.

## 4. Read the cron output

The setup example writes command output to:

```text
var/log/cron.log
```

Check its latest lines:

```bash
tail -n 100 var/log/cron.log
```

Common causes include a wrong absolute PHP path, wrong project directory,
missing `.env`, an unwritable runtime directory, expired credentials, or an API
outage.

## 5. Retry after correcting the cause

Copy the 64-character `task_id` returned by `failures:list`:

```bash
php bin/tracezilla-integration failures:retry --task=<task-id>
```

The task becomes pending immediately. The next reconciliation still performs
normal duplicate and business-rule checks.

If the customer deliberately does not want the record processed:

```bash
php bin/tracezilla-integration failures:dismiss \
  --task=<task-id> \
  --reason='Customer approved exclusion'
```

Dismissal is written to history before the retry file is removed.

## When to involve a developer

Escalate when a corrupt task appears, an unexpected error repeatedly moves to
attention, valid records remain outside the selected window, or the deployment
check fails. The implementation details are documented in
[Cron error and retry internals](./errors.html).
