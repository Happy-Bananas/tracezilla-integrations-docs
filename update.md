---
title: Update Your Integration
layout: default
nav_order: 90
---

# Update your integration

Your integration gets fixes and improvements from two places:

| Part | Where it lives in your project | How it is updated |
|:--|:--|:--|
| The shared core: the tracezilla client and login, the run lock, `--dry-run` / `--execute`, and the seed | `vendor/happy-bananas/bifrost-core/`, installed by Composer | With one `composer update` command |
| The template: the commands in `bin/`, the code for the external service in `src/`, and the scenario templates | Your project's own files | By hand, from the template's changelog |

Your business rules in `custom/` and your `.env` are yours. An update never
changes them.

## Update the shared core

Read the [bifrost-core changelog](https://github.com/Happy-Bananas/bifrost-core/blob/main/CHANGELOG.md).
Each release says what changed and what, if anything, your integration has to
change.

Then, in your integration project folder:

With Docker:

```bash
docker compose run --rm php composer update happy-bananas/bifrost-core
docker compose run --rm php composer test
```

Without Docker:

```bash
composer update happy-bananas/bifrost-core
composer test
```

`composer update` installs the newest release that your `composer.json`
allows, for example every `0.6.x` release for `"happy-bananas/bifrost-core": "^0.6"`.
To move to the next version, for example `0.7`, change that line to `^0.7`
first and make the changes the changelog lists for it.

Then run your commands with `--dry-run`, check the connections, and deploy as
usual. Commit `composer.json` and `composer.lock`, so the server installs the
same version.

## Update from a newer template

The file `TEMPLATE_VERSION` in your project says which template release it was
created from, for example:

```text
tracezilla-woocommerce-php v0.1.0
```

The template's changelog lists what changed in each later release:
[WooCommerce](https://github.com/Happy-Bananas/tracezilla-woocommerce-php/blob/main/CHANGELOG.md) and
[Shopify](https://github.com/Happy-Bananas/tracezilla-shopify-php/blob/main/CHANGELOG.md).
Copy the changes you want into your project:

1. Create a fresh project of the same platform next to yours, for example
   `create-project woocommerce YOUR_INTEGRATION_NAME-new`. It comes with the
   newest release.
2. Compare the two folders, for example with `diff -r --exclude=vendor --exclude=var
   YOUR_INTEGRATION_NAME YOUR_INTEGRATION_NAME-new`, and copy over the files the
   changelog mentions. Never copy `custom/` or `.env`.
3. Set `TEMPLATE_VERSION` to the new release, run the tests, and delete the
   fresh project.

A project created before `TEMPLATE_VERSION` existed started from the
template's `main` before release `0.1.0`.
