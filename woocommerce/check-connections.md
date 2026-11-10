---
title: Check Connections
layout: default
parent: WooCommerce
nav_order: 10
---

# Check the WooCommerce and tracezilla connections

{: .label .label-green }
Read only

## Behavior

The command checks the WooCommerce and tracezilla credentials with one small
read-only request each:

- WooCommerce: the store's system status, which returns its URL and
  WooCommerce version.
- tracezilla: one page of SKUs in the configured team, using the stored OAuth
  token. The token is renewed automatically when it is close to expiry.

Both systems are always checked, so a problem with one does not hide the state
of the other. Run this check first: no other command works until both
connections pass.

## Before you run it

With Docker, two things must be running:

1. **Docker.** Start Docker Desktop on Mac and Windows, or the Docker service on
   Linux.
2. **The WooCommerce store.** For the local sandbox, run `docker compose up -d`
   in the sandbox folder. It does not start by itself after the computer
   restarts.

## Run the command

With Docker:

Mac and Linux:

```bash
./check-connection
```

The helper checks that Docker and `.env` are available, builds the development
container and installs the PHP packages when needed, and then runs the check.

Windows (PowerShell), or anywhere without the helper:

```powershell
docker compose run --rm php php bin/bifrost-connect connection:check
```

Without Docker:

```bash
php bin/bifrost-connect connection:check
```

A successful result looks like:

```text
Connection check passed.
WooCommerce: connected to http://localhost:8080 (WooCommerce 11.2.1)
tracezilla:  connected to team your-development-company
```

## Options

Request structured output.

With Docker, Mac and Linux:

```bash
./check-connection --json
```

With Docker, Windows (PowerShell):

```powershell
docker compose run --rm php php bin/bifrost-connect connection:check --json
```

Without Docker:

```bash
php bin/bifrost-connect connection:check --json
```

```json
{
    "status": "ok",
    "woocommerce": {
        "connected": true,
        "store_url": "http://localhost:8080",
        "version": "11.2.1"
    },
    "tracezilla": {
        "team": "your-development-company",
        "connected": true
    }
}
```

## When a connection fails

The failing line explains the problem. Common causes:

| Message | Fix |
|:--|:--|
| `Missing required environment variable: WOOCOMMERCE_CONSUMER_KEY` | Add the [REST API keys](./setup/rest-api-keys.html) to `.env`. |
| WooCommerce `401` with `Consumer key is invalid` or `Consumer secret is invalid` | The key in `.env` does not match the store. For the sandbox, use `ck_sandbox` / `cs_sandbox`. |
| `WOOCOMMERCE_URL must start with https://` | `.env` names a real store with `http://`. Use its `https://` address. Plain `http://` is allowed only for a store on this computer (`localhost` or `host.docker.internal`), such as the sandbox, because it would send the API keys unencrypted. |
| WooCommerce `401` with `Sorry, you cannot list resources.` | The key was not accepted at all, usually because the store does not use HTTPS. Real stores must; the sandbox handles this itself. |
| `Cannot connect to the Docker daemon` or `error during connect` | Docker is not running. Start Docker Desktop, or the Docker service on Linux. |
| WooCommerce `Connection refused` or `Could not connect to server` | The store is not running. Start the sandbox with `docker compose up -d` in its folder. With Docker, `WOOCOMMERCE_URL` must be `http://host.docker.internal:8080`. |
| `Missing required environment variable: TRACEZILLA_CLIENT_ID` (or `_USERNAME`, `_PASSWORD`) | Add the [Resource Owner Password Credentials](../fundamentals/account-and-team.html#oauth-client-and-api-user) to `.env`. |
| `tracezilla token request failed with HTTP 400 … Check TRACEZILLA_CLIENT_ID, …` | tracezilla refused the login. Copy all four values again from tracezilla; the username ends in `@tracezilla.api`. |

## Safety and exit status

The command never changes products, orders, inventory, or settings, and never
prints credentials or tokens. It returns exit code `0` only when both
connections work, and `1` otherwise, so it can be used in scripts and before
scheduled runs. It does not take the run lock, so it can run while a scheduled
command is working.

## Architecture

<pre class="mermaid">
flowchart TB
    Entry[bin/bifrost-connect connection:check] --> Command[bin/check-connection]
    Command --> Checker[ConnectionChecker]
    Checker --> Woo[LiveWooCommerceConnection<br>GET system_status]
    Checker --> TZ[LiveTracezillaConnection<br>GET /skus]
    TZ --> Tokens[TokenProvider<br>renews the OAuth token]
    Checker --> Result[Structured result]
    Result --> Text[Text output]
    Result --> Json[JSON output]
</pre>

## Implementation

| Responsibility | Source |
|---|---|
| Docker helper | `check-connection` |
| Entry point | `bin/bifrost-connect` |
| CLI options, composition, output, and exit status | `bin/check-connection` |
| Both checks and the structured result | `src/Connection/ConnectionChecker.php` |
| What the check needs from each service | `src/Connection/WooCommerceConnection.php`, `src/Connection/TracezillaConnection.php` |
| The real requests | `src/Connection/LiveWooCommerceConnection.php`, `src/Connection/LiveTracezillaConnection.php` |
| WooCommerce REST transport | `src/WooCommerce/Client.php` |
| tracezilla transport and token renewal | `src/Tracezilla/Client.php`, `src/Tracezilla/TokenProvider.php` |

## Tests

With Docker:

```bash
docker compose run --rm php composer test
```

Without Docker:

```bash
composer test
```

The tests verify the combined result, that a failure in one system does not
hide the other, and that unexpected responses count as failures. They never
contact either service: instead of the live connections they use fake ones,
`tests/Unit/Connection/FakeWooCommerceConnection.php` and
`FakeTracezillaConnection.php`, which return a prepared answer or fail with a
prepared message.
