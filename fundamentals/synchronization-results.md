---
title: Synchronization Results
layout: default
parent: tracezilla Fundamentals
nav_order: 90
---

# Synchronization results

A synchronization run can contain several outcomes at once. Return one
structured result per selected record plus a run summary; do not reduce a
mixed run to a single success boolean.

## Common outcomes

| Outcome | Meaning |
|:--|:--|
| `would_create` / `would_update` | Preview found a write candidate but made no write |
| `created` / `updated` | tracezilla or the external destination accepted the write |
| `unchanged` | Source and destination already agree |
| `skipped` | A documented policy intentionally excluded the record |
| `invalid` | Source data or the mapped payload is unsafe |
| `failed` | An API or unexpected operational action failed |

Not every workflow needs every outcome. Use stable reason codes for automation
and separate human-readable messages for context.

## Run summary

Include:

- Source and destination counts where meaningful.
- Candidate, selected, and processed counts.
- Counts by outcome.
- Whether the run was a preview.
- The applied limit and selection window.
- A non-zero command exit when an operational request failed.

Skipped or invalid records must remain visible even when policy permits the
overall run to finish.

## Partial and uncertain failures

Process item results independently so one rejected record does not hide the
others. Never report a write as completed until the destination accepts it.

For a timeout after a create request, read the destination by stable identity
before retrying. For an update, perform a fresh read and preview. Error results
may contain a safe HTTP status, reason code, or exception class, but never
tokens, Authorization headers, complete configuration, or uncontrolled error
payloads.
