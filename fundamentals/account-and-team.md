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

## OAuth client and login

Every integration authenticates with an OAuth client and a one-time login. The
integration then renews its own access token, so it never stops because a
token expired.

In a customer installation, use a dedicated integration user rather than a
personal account, so the integration keeps working when employees leave.

Create an OAuth client as described in the
[tracezilla API documentation](https://tech.tracezilla.com/docs/rest-api-v1/making-requests/authentication.html#bearer-token-and-resource-owner-password-credentials),
and add its Client ID and Client Secret to `.env`:

```dotenv
TRACEZILLA_CLIENT_ID=...
TRACEZILLA_CLIENT_SECRET=...
```

After installing the integration, log in once from its project folder:

| | Command |
|:--|:--|
| WooCommerce with Docker | `docker compose run --rm php php bin/bifrost-connect tracezilla:login` |
| Shopify with Docker | `docker compose run --rm integration php bin/bifrost-connect tracezilla:login` |
| Without Docker | `php bin/bifrost-connect tracezilla:login` |

The command asks for your tracezilla username and password. The password is
used once and never stored. The tokens are saved in
`var/tracezilla-token.json`, and every integration command renews the access
token 30 days before it expires. Run the login again only if a command reports
that the refresh token was rejected, for example after the client was deleted
in tracezilla.

## Keep credentials secret

The Client Secret and the token file are secrets. Never commit or share
`.env` or `var/`. If a secret is exposed, revoke it in tracezilla immediately
and create a new one.

## Official reference

- [tracezilla API authentication](https://tech.tracezilla.com/docs/rest-api-v1/making-requests/authentication.html#bearer-token-and-resource-owner-password-credentials)
