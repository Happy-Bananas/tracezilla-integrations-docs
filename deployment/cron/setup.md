---
title: Set Up a Cron Task
layout: default
parent: Cron
grand_parent: Deployment
nav_order: 10
---

# Set up a cron task

Cron calls the same headless command that you tested manually. It does not call
a web page and it does not contain customer business rules.

## 1. Check the deployment

From the integration project, run:

```bash
php bin/bifrost-connect deployment:check
```

Continue only when it reports `"ready": true`. This verifies that the
application can use its private runtime directory safely.

## 2. Test the workflow manually

Run the exact workflow in dry-run mode first. For individual Shopify orders:

```bash
php bin/bifrost-connect orders:import-individual \
  --customer='Webshop customer' \
  --warehouse=2 \
  --days=3 \
  --limit=100
```

Review the result, configuration, customer mapping, and selected records.
Perform a bounded test-account write before scheduling unattended execution.

## 3. Find the absolute paths

Cron usually has a smaller environment than an interactive terminal. Record:

- The absolute project directory.
- The absolute PHP executable path supplied by the host.
- The command options approved for this customer.

Do not assume that the host uses `/usr/bin/php`. Hosting control panels often
display the correct PHP command for the selected runtime version.

## 4. Create the cron command

A shared-hosting control panel commonly accepts a schedule and one command.
For a run every five minutes:

```text
*/5 * * * *
```

Use a command like this, replacing every example value and path:

```bash
cd /absolute/path/customer-integration && /absolute/path/php bin/bifrost-connect orders:import-individual --customer='Webshop customer' --warehouse=2 --days=3 --limit=100 --execute --confirm >> var/log/cron.log 2>&1
```

The application prevents overlapping runs automatically. If a previous
workflow is still running, the later invocation exits safely.

## 5. Verify the scheduled run

After the first scheduled time:

1. Open `var/log/cron.log` and confirm the command started and completed.
2. Confirm the expected dry-run or execution result.
3. Run `php bin/bifrost-connect failures:list`.
4. Check again after the next scheduled time.

The `--days` lookback must be comfortably longer than the cron interval and
the complete automatic retry period. That overlap allows a later run to find
work missed during an outage. Stable external references prevent duplicates.

If a run does not behave as expected, continue with
[Troubleshoot scheduled runs](./troubleshooting.html).
