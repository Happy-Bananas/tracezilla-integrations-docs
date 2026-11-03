---
title: Retries
layout: default
parent: Cron
grand_parent: Deployment
nav_order: 90
---

# How failed work is retried

{: .label .label-red }
Unsupported

{: .warning }
This page explains what the integration does with failures. To fix a failed
run, start with [Troubleshoot scheduled runs](./troubleshooting.html).

Cron only starts the integration again; it cannot retry one failed order, SKU,
or inventory update. The integration therefore remembers failed items itself
and retries them on later runs. You do not need to do anything for this to
happen, but you need to know where it keeps its files and what you must not
change.

## Where the files are

The integration keeps its state in the runtime directory, `var/` by default:

```text
var/
├── locks/
│   └── integration.lock
├── retry/
│   ├── pending/
│   │   └── <task-id>.json
│   ├── attention/
│   │   └── <task-id>.json
│   └── corrupt/
│       └── <task-id>.json
├── history/
│   └── YYYY-MM.ndjson
└── log/
    └── cron.log
```

A retry file holds metadata about the failure, not a copy of the order:

```json
{
  "schema_version": 1,
  "task_id": "<sha256>",
  "workflow": "orders:import-individual",
  "source": "shopify",
  "external_id": "1842",
  "attempts": 2,
  "first_failed_at": "2026-08-24T12:30:00+00:00",
  "last_failed_at": "2026-08-24T12:35:00+00:00",
  "next_attempt_at": "2026-08-24T12:50:00Z",
  "status": "retry_pending",
  "last_error": {
    "code": "tracezilla_order_creation_failed",
    "category": "unexpected",
    "message": "tracezilla rejected the sales-order creation request."
  },
  "attention_reason": null
}
```

On a retry, the integration reads the record from the source system again. No
customer addresses, order lines, or tokens are stored in these files.

The files are written safely, so a crash never leaves a half-written file. Do
not edit or move them by hand; use the commands below.

## What you need to do

- Keep `var/` between deployments, and out of Git and the public web root.
- Run setup, manual commands, and cron as the same user, so all of them can
  read and write `var/`.
- If the project folder is read-only on the server, set
  `TRACEZILLA_RUNTIME_DIR` to a writable folder.
- Run the project on one server only. Two servers running the same project
  would each keep their own retry files and could process the same item twice.

Before you schedule cron, check that the runtime directory works:

```bash
php bin/bifrost-connect deployment:check
```

The check writes, renames, reads, locks, and removes a test file in the
runtime directory. If it fails, fix the permissions before you schedule cron.

## Which failures are retried

| Kind | Examples | What happens |
|---|---|---|
| Temporary | Timeout, rate limit, connection failure, HTTP `5xx` | Retried automatically on the schedule below |
| Business decision | Unknown SKU, unsupported currency, missing customer mapping | Moved to `attention`; retried after you fix the cause |
| Already processed | The tracezilla record with the same external reference exists | Counted as done; nothing is written again |
| Unexpected | Unclassified error or invalid response | Retried a few times, then moved to `attention` |

Business problems are not retried forever, because retrying cannot fix them.

## Retry schedule

```text
Attempt 1: next cron run
Attempt 2: 5 minutes later
Attempt 3: 15 minutes later
Attempt 4: 1 hour later
Attempt 5: 6 hours later
Then: moved to attention
```

Each run first retries the tasks that are due, then looks for new work.

## Test with more data than one run handles

A run looks back over a period (for example, the last day) and processes at
most a limited number of records. The retry files remember a failure after its
record has left that period, and stable external references stop either from
creating duplicates.

Before you schedule a workflow, test it with more records than one run can
process, and check that older records are still picked up on later runs
instead of being left behind.

## Commands for handling failures

```bash
php bin/bifrost-connect failures:list
php bin/bifrost-connect failures:retry --task=<task-id>
php bin/bifrost-connect failures:dismiss --task=<task-id> --reason='Approved exclusion'
```

A manual retry takes the same run lock and uses the same duplicate checks as a
scheduled run.
