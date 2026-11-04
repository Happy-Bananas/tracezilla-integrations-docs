---
title: Set Up a Cron Task
layout: default
parent: Cron
grand_parent: Deployment
nav_order: 10
---

# Set up a cron task

{: .label .label-red }
Unsupported

{: .warning }
This deployment guidance is an optional example. Hosting, security, backups,
monitoring, and scheduled operation remain your or your customer's
responsibility.

Cron calls the same console command that you tested manually. It does not call
a web page and it does not contain customer business rules.

## 1. Check the deployment

From the integration project, run:

```bash
php bin/bifrost-connect connection:check
```

The Shopify integration also has a deployment check:

```bash
php bin/bifrost-connect deployment:check
```

Continue only when both connections pass and, for Shopify, the deployment
check reports `"ready": true`. That verifies that the application can use its
private runtime directory safely. The WooCommerce integration has no
deployment check yet.

## 2. Test the workflow manually

Run the exact workflow in dry-run mode first. For individual Shopify orders:

```bash
php bin/bifrost-connect orders:import-individual \
  --customer='Webshop customer' \
  --warehouse=2 \
  --days=3 \
  --limit=100 --dry-run
```

For WooCommerce, run your scenario, for example one created from the
`create-tracezilla-skus` template:

```bash
php bin/bifrost-connect scenario:run create-tracezilla-skus --limit=10 --dry-run
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

Use a command like this, replacing every example value and path. For Shopify:

```bash
cd /absolute/path/customer-integration && mkdir -p var/log && /absolute/path/php bin/bifrost-connect orders:import-individual --customer='Webshop customer' --warehouse=2 --days=3 --limit=100 --execute >> var/log/cron.log 2>&1
```

For WooCommerce:

```bash
cd /absolute/path/customer-integration && mkdir -p var/log && /absolute/path/php bin/bifrost-connect scenario:run create-tracezilla-skus --limit=100 --execute >> var/log/cron.log 2>&1
```

`mkdir -p var/log` creates the log folder if it is missing. Without it, the
shell cannot open the log file and the command never starts.

The application prevents overlapping runs automatically. If a previous
workflow is still running, the later invocation exits safely.

### Exit codes

Every command ends with an exit code that a monitoring tool can check:

| Exit code | Meaning | What to do |
|:--|:--|:--|
| `0` | The run succeeded. | Nothing. |
| `75` | Skipped: another integration command still held the run lock. Nothing was changed. | Nothing, unless it happens on every run; then a command is stuck or the interval is too short. |
| `1` | The run failed, or at least one change failed. | Read `var/log/cron.log`. |
| `2` | The command was called with wrong options. | Fix the cron command. |

Because the command line above sends all output to `var/log/cron.log`, cron
has nothing to email you. Nothing tells you about a failed run unless you set
up monitoring that alerts on exit codes other than `0` and `75`.

## 5. Verify the scheduled run

After the first scheduled time:

1. Open `var/log/cron.log` and confirm the command started and completed.
2. Confirm the expected dry-run or execution result.
3. Shopify only: run `php bin/bifrost-connect failures:list`.
4. Check again after the next scheduled time.

For the Shopify order import, the `--days` lookback must be comfortably longer
than the cron interval and the complete automatic retry period. That overlap
allows a later run to find work missed during an outage. Stable external
references prevent duplicates.

If a run does not behave as expected, continue with
[Troubleshoot scheduled runs](./troubleshooting.html).
