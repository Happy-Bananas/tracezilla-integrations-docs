---
title: Credentials
layout: default
parent: tracezilla Prerequisites
nav_order: 10
---

# tracezilla credentials

Every integration needs the **team slug** of your development company and a
way to authenticate. Complete [Development Company](./development-company.html)
first.

## Team slug

In tracezilla, a company is a team. The team slug is the first part of the
browser URL when the company is open, for example `dev-woocommerce-abc1` in:

```text
https://app.tracezilla.com/dev-woocommerce-abc1/dashboard/home
```

The slug is not necessarily the same as the displayed company name. Add it to
`.env`:

```dotenv
TRACEZILLA_TEAM_SLUG=dev-woocommerce-abc1
```

The team slug is not secret.

## Authentication

The integrations authenticate in different ways today:

| Integration | Authentication | Token lifetime |
|:--|:--|:--|
| Shopify | Personal access token | About 13 months, then it must be replaced by hand |
| WooCommerce | OAuth client and a one-time login | Renews itself automatically |

The Shopify integration will move to the OAuth client later.

Only users with the Owner or Administrator role can create tokens and clients.
In a customer installation, use a dedicated integration user rather than a
personal account, so the integration keeps working when employees leave.

### Personal access token (Shopify)

1. In tracezilla, open **Settings → My account → API Tokens**.
2. Create a **personal access token**.
3. Copy the complete token. It is long and starts with `eyJ`.
4. Add it to `.env`:

```dotenv
TRACEZILLA_API_KEY=eyJ...
```

### OAuth client and login (WooCommerce)

The OAuth client uses the Resource Owner Password Credentials grant. Unlike a
personal access token, it receives a refresh token, so the integration renews
its own access token and never stops because a token expired.

1. In tracezilla, open **Settings → My account → API Tokens**.
2. Create an **OAuth client**.
3. Add its Client ID and Client Secret to `.env`:

```dotenv
TRACEZILLA_CLIENT_ID=...
TRACEZILLA_CLIENT_SECRET=...
```

After installing the integration, log in once from its project folder:

```bash
docker compose run --rm php php bin/bifrost-connect tracezilla:login
```

The command asks for your tracezilla username and password. The password is
used once and never stored. The tokens are saved in
`var/tracezilla-token.json`, and every integration command renews the access
token 30 days before it expires. Run the login again only if a command reports
that the refresh token was rejected, for example after the client was deleted
in tracezilla.

## Keep credentials secret

The token, Client Secret, and token file are secrets. Never commit or share
`.env` or `var/`. If a secret is exposed, revoke it in tracezilla immediately
and create a new one.

## Official reference

- [tracezilla API authentication](https://tech.tracezilla.com/docs/rest-api-v1/making-requests/authentication)
