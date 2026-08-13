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

- The disposable local sandbox included in
  [`tracezilla-woocommerce-php`](https://github.com/Happy-Bananas/tracezilla-woocommerce-php).
- An existing WooCommerce store where you are authorized to create REST API
  credentials.

Use the local sandbox while learning or testing commands that can change data.

## Start the local sandbox

Clone the PHP example and enter its directory:

```bash
git clone https://github.com/Happy-Bananas/tracezilla-woocommerce-php.git
cd tracezilla-woocommerce-php
```

Start WordPress, WooCommerce, and MariaDB:

```bash
docker compose --profile sandbox up -d
```

The first start downloads and activates WooCommerce. Follow its progress with:

```bash
docker compose --profile sandbox logs -f wp-cli
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
docker compose run --rm app composer install
docker compose run --rm app php bin/test-connection
```

A successful check prints the connected store and WooCommerce version.

## Expected result

At the end of setup, you should have:

- A local sandbox or an authorized WooCommerce test store.
- A WooCommerce REST API Consumer Key and Consumer Secret.
- Credentials stored in the untracked `.env` file.
- A successful connection test from the standalone PHP container.

## Stop the local sandbox

Stop it while retaining the store data:

```bash
docker compose --profile sandbox down
```

To permanently delete the disposable store and start over:

```bash
docker compose --profile sandbox down --volumes
```

The second command deletes the local WordPress database and files. It does not
affect an external WooCommerce store or tracezilla.

## Official WooCommerce reference

- [WooCommerce REST API key setup](https://woocommerce.com/document/woocommerce-rest-api/)
