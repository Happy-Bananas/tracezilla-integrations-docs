---
title: Cron Error and Retry Internals
layout: default
parent: Cron
grand_parent: Deployment
nav_order: 90
---

# Cron error and retry internals

{: .warning }
This page is for framework maintainers. Consultants should begin with
[Troubleshoot scheduled runs](./troubleshooting.html).

Cron does not understand integration tasks and cannot retry an individual
failed order, SKU, or inventory update. Cron only starts the application again.
The application must remember incomplete work and retry it during later runs.

## File-based retry spool

The integration stores retry state in a private file-based spool under the
configured runtime directory:

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

Each retry file contains metadata, not a copy of a sensitive commerce payload:

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

On retry, the workflow retrieves authoritative data from the source system
again. This avoids retaining customer addresses, order lines, tokens, or other
sensitive payloads in the spool.

## Make file updates atomic

Writing a JSON file directly can leave a partial file if the process stops at
the wrong moment. Use this sequence while holding the global application lock:

1. Create a temporary file in the same directory as the destination.
2. Restrict its permissions.
3. Write the complete JSON document.
4. Flush and close the file.
5. Atomically rename it to the final task filename.

Renaming within the same filesystem prevents readers from observing a
half-written task. Use a deterministic task ID derived from the workflow and
external identity so recording the same failure twice updates one task instead
of creating duplicates.

PHP releases an operating-system `flock()` automatically if the process exits
or crashes. Never decide lock ownership from the text stored inside a lock
file; diagnostic metadata may be stale even when the operating-system lock is
free.

## Directory permissions

Retry state requires one persistent writable directory. The requirement is
small and can be tested during setup.

Recommended rules:

- Keep runtime files outside the public web root where possible.
- Create directories with owner-only access, normally mode `0700`.
- Create task files with owner-only access, normally mode `0600`.
- Run setup, manual commands, and cron as the same hosting-account user.
- Make the runtime path configurable when the project directory is read-only.
- Never store API tokens or full source payloads in task files or logs.
- Do not commit `var/` contents to Git.

The application provides a setup check:

```bash
php bin/bifrost-connect deployment:check
```

It should create, atomically rename, read, lock, and remove a harmless test file
inside the configured runtime directory. Deployment must stop with a clear
message if this check fails.

## Classify failures

| Classification | Examples | Action |
|---|---|---|
| Temporary | Timeout, rate limit, connection failure, HTTP `5xx` | Retry automatically with backoff |
| Business decision | Unknown SKU, unsupported currency, missing customer mapping | Move to `attention`; retry after correction |
| Already processed | Existing stable external reference | Mark successful without another write |
| Unexpected | Unclassified exception or invalid response | Retry a limited number of times, then move to `attention` |

Do not retry every error forever. Permanent business problems can otherwise
create noise, API load, and misleading logs.

## Retry schedule

A conservative example backoff is:

```text
Attempt 1: next cron run
Attempt 2: 5 minutes later
Attempt 3: 15 minutes later
Attempt 4: 1 hour later
Attempt 5: 6 hours later
Then: attention required
```

Store an absolute `next_attempt_at` time. A later scheduled run first processes
pending tasks whose retry time has arrived, then performs normal
reconciliation.

## Do not lose work outside the lookback window

Reconciliation and the retry spool complement each other:

- The overlap window recovers recent missed or interrupted work.
- The retry spool remembers a specific failure after its source record ages
  out of that window.
- Stable external identities prevent either mechanism from creating a
  duplicate.

A source selection limit must not repeatedly select already-processed records
while leaving older eligible records behind. Verify pagination, ordering,
checkpoint behavior, and limits with a data volume larger than one run can
process.

## Operator commands

The console interface makes persistent failures visible without
opening JSON files manually:

```bash
php bin/bifrost-connect failures:list
php bin/bifrost-connect failures:retry --task=<task-id>
php bin/bifrost-connect failures:dismiss --task=<task-id> --reason='Approved exclusion'
```

Every manual retry still uses the global atomic lock and the workflow's normal
idempotency checks.

## Deployment boundary

The file spool supports one integration instance on one persistent filesystem.
Do not run the same project concurrently from several hosts. The deployment
check must pass on the same account and filesystem used by cron.
