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

## Run the command

### With Docker

```bash
./check-connection
```

The helper checks that Docker and `.env` are available, builds the development
container and installs the PHP packages when needed, and then runs the check.
The direct equivalent is:

```bash
docker compose run --rm php php bin/bifrost-connect connection:check
```

### Without Docker

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

Request structured output:

```bash
./check-connection --json
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
| WooCommerce `401` with `Sorry, you cannot list resources.` | The key was not accepted at all, usually because the store does not use HTTPS. Real stores must; the sandbox handles this itself. |
| WooCommerce `Connection refused` | Start the sandbox. With Docker, `WOOCOMMERCE_URL` must be `http://host.docker.internal:8080`. |
| `No tracezilla token is stored. Run bin/bifrost-connect tracezilla:login first.` | Run the [login](../fundamentals/account-and-team.html#oauth-client-and-login-woocommerce). |
| `The refresh token was rejected; run bin/bifrost-connect tracezilla:login again.` | The OAuth client was deleted or the session revoked. Log in again. |
| `Missing required environment variable: TRACEZILLA_CLIENT_ID` | Add the [OAuth client](../fundamentals/account-and-team.html#oauth-client-and-login-woocommerce) to `.env`. |

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
    Checker --> Woo[WooCommerce client<br>GET system_status]
    Checker --> TZ[tracezilla client<br>GET /skus]
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
| WooCommerce REST transport | `src/WooCommerce/Client.php` |
| tracezilla transport and token renewal | `src/Tracezilla/Client.php`, `src/Tracezilla/TokenProvider.php` |

## Tests

```bash
docker compose run --rm php composer test
```

The tests verify the combined result, that a failure in one system does not
hide the other, and that unexpected responses count as failures, without
contacting either service.
