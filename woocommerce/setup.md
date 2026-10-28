---
title: Setup
layout: default
parent: WooCommerce
nav_order: 10
---

# WooCommerce setup

Prepare a safe WooCommerce environment and create REST API credentials for the
integration. No prior WooCommerce or WordPress development experience is
required.

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

## Populate the development store

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

## Create a REST API key

In WordPress administration, select:

**WooCommerce → Settings → Advanced → REST API**

For the local sandbox, you can also open the
[REST API settings directly](http://localhost:8080/wp-admin/admin.php?page=wc-settings&tab=advanced&section=keys).

Select **Create an API key** or **Add key**, then enter:

| Field | Suggested value |
|:--|:--|
| Description | `Tracezilla local development` |
| User | `admin` |
| Permissions | `Read/Write` |

Select **Generate API key** and copy both the Consumer Key and Consumer Secret
immediately. WooCommerce displays the complete Consumer Secret only once.

Read-only credentials are sufficient for catalog comparison. The example uses
Read/Write credentials because later commands will create or update test data.
Use only the permissions required by the integration in a production store.

## Configure the PHP integration

Clone the deployable console application separately from the sandbox:

```bash
git clone https://github.com/Happy-Bananas/tracezilla-woocommerce-php.git
cd tracezilla-woocommerce-php
```

Create the local environment file:

```bash
cp .env.example .env
```

For the Docker sandbox, configure:

```dotenv
WOOCOMMERCE_URL=http://host.docker.internal:8080
WOOCOMMERCE_CONSUMER_KEY=ck_...
WOOCOMMERCE_CONSUMER_SECRET=cs_...
```

For an existing store, replace `WOOCOMMERCE_URL` with its public HTTPS URL.
Never commit `.env` or API credentials to Git.

Install the PHP dependencies and validate the connection:

```bash
docker compose run --rm php composer install
docker compose run --rm php php bin/test-connection
```

A successful check prints the connected store and WooCommerce version.

## Seed tracezilla

Populate your development company in tracezilla with the
[standard test dataset](../fundamentals/test-data.html): partners, a
warehouse, SKUs that match the sandbox, and received stock.

First complete [Create a development company](../fundamentals/development-company.html).

### Create an OAuth client

The WooCommerce integration authenticates with an OAuth client and the
Resource Owner Password Credentials grant. Unlike a personal access token, it
receives a refresh token, so the integration renews its own access token and
never stops because a token expired.

In tracezilla, open **Settings → My account → API Tokens** and create an OAuth
client. Copy the Client ID and Client Secret into `.env`:

```dotenv
TRACEZILLA_TEAM_SLUG=your-development-company
TRACEZILLA_CLIENT_ID=...
TRACEZILLA_CLIENT_SECRET=...
TRACEZILLA_OWN_COMPANY=Your development company
```

The team slug is the first part of the browser URL when the company is open,
for example `dev-woocommerce-abc1` in
`https://app.tracezilla.com/dev-woocommerce-abc1/dashboard/home`.
`TRACEZILLA_OWN_COMPANY` is the exact partner name of your own company in
tracezilla. The seed adds its warehouse location to that partner.

### Log in once

Exchange your tracezilla username and password for tokens:

```bash
docker compose run --rm php php bin/tracezilla-login
```

The password is used once and never stored. The tokens are saved in
`var/tracezilla-token.json`, which must stay out of Git. Every integration
command renews the access token automatically 30 days before it expires.
Run the login again only if the command reports that the refresh token was
rejected, for example after the client was deleted in tracezilla.

In a customer installation, log in as a dedicated integration user instead of
a personal account, so the integration keeps working when employees leave.

### Run the seed

Preview what would be created. Nothing is written:

```bash
docker compose run --rm php php bin/seed-tracezilla
```

Check that the output names your development company, then write the data:

```bash
docker compose run --rm php php bin/seed-tracezilla --execute --confirm
```

Running the command again reports every record as `exists`. To use another
dataset, pass the path of its YAML file as the first argument.

## Expected result

At the end of setup, you should have:

- A local sandbox or an authorized WooCommerce test store.
- A WooCommerce REST API Consumer Key and Consumer Secret.
- Credentials stored in the untracked `.env` file.
- A successful connection test from the standalone PHP container.
- A development company in tracezilla seeded with the standard test dataset.

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

## Official WooCommerce reference

- [WooCommerce REST API key setup](https://woocommerce.com/document/woocommerce-rest-api/)
