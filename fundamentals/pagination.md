---
title: Pagination
layout: default
parent: tracezilla Fundamentals
nav_order: 70
---

# Pagination

A successful first response is not evidence that an integration read the
complete catalog, inventory set, partner list, or order set.

Existing tracezilla examples request a bounded `perPage` value, collect the
response `data`, and continue using `links.next_page` until it is empty. Verify
these field names and supported limits against the current endpoint schema in
the [tracezilla API documentation](https://app.tracezilla.com/api/documentation).

## Safe traversal

1. Choose an explicit stable sort when the endpoint supports one.
2. Request a bounded page size.
3. Process or collect the current `data` array.
4. Stop when `links.next_page` is empty.
5. Otherwise parse only the next-page query parameters required by the
   configured tracezilla endpoint.
6. Reject invalid or repeated next pages instead of looping forever.

Never forward the Bearer token to an unexpected origin taken from a pagination
link. Either require the configured origin or reuse only validated query
parameters with the existing authenticated client.

## Workflow limits are different

An execution limit is not an API page size and is not a substitute for
pagination. Unless a workflow explicitly defines sampling:

1. Read the complete source and destination identity sets.
2. Determine candidates.
3. Apply the workflow's preview or execution limit.
4. Report source, candidate, selected, and processed counts.

This prevents a command from reporting false differences merely because it
read only the first page.
