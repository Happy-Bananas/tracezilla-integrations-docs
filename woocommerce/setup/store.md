---
title: Development Store
layout: default
parent: WooCommerce Prerequisites
grand_parent: WooCommerce
nav_order: 10
---

# Development store

Prepare a safe WooCommerce store for development. No prior WooCommerce or
WordPress development experience is required.

## Choose a store

You can use either:

- The disposable `tracezilla-woocommerce-sandbox` included in the integrations
  workspace.
- An existing WooCommerce store where you are authorized to create REST API
  credentials.

Use the local sandbox while learning or testing commands that can change data.

## Start the local sandbox

Clone the integrations workspace and enter the sandbox directory:

```bash
git clone --recurse-submodules \
  https://github.com/Happy-Bananas/tracezilla-integrations.git
cd tracezilla-integrations/tracezilla-woocommerce-sandbox
```

Start WordPress, WooCommerce, and MariaDB:

```bash
docker compose up -d
```

The first start downloads and activates WooCommerce. Follow its progress with:

```bash
docker compose logs -f wp-cli
```

Wait until the log reports that the WooCommerce sandbox is ready. Then open
[the local WordPress administration page](http://localhost:8080/wp-admin/) and
sign in:

| Field | Local sandbox value |
|:--|:--|
| Username | `admin` |
| Password | `tracezilla` |

These credentials are only for the disposable local sandbox. Never use this
password on a public WordPress installation.

## Inspect the test data

The sandbox setup automatically creates deterministic WooCommerce data:

- 200 top-level products and two product variations.
- Three customers.
- Three recent orders with `processing`, `completed`, and `on-hold` statuses.

In WordPress administration, use **Products → All Products**,
**WooCommerce → Customers**, and **WooCommerce → Orders** to inspect the
fixtures. The catalog contains food products in ten categories and includes
simple, sale-priced, out-of-stock, and variable products.

The products have stable `TZ-<CATEGORY>-<NNN>` SKUs, such as `TZ-DAIRY-001`,
so that integration examples and tests can refer to predictable catalog data.
Running the product seeder again updates the existing fixtures instead of
creating duplicates, and removes `TZ-` products that are no longer part of the
catalog:

```bash
docker compose run --rm \
  --entrypoint wp wp-cli \
  eval-file /seed-products.php --path=/var/www/html
```

Run this command from the `tracezilla-woocommerce-sandbox` directory whenever
you want to restore the standard products, customers, and orders. The operation
is idempotent and does not reset the rest of the WordPress database. Fixture
creation does not send customer or order emails.

## Stop the local sandbox

Stop it while retaining the store data:

```bash
docker compose down
```

To permanently delete the disposable store and start over:

```bash
docker compose down --volumes
```

The second command deletes the local WordPress database and files. It does not
affect an external WooCommerce store or tracezilla.

## Next step

Continue with [REST API Keys](./rest-api-keys.html).
