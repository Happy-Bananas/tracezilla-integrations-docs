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

Write the values down for now. The project's `.env` file only exists after you
create the project in Getting Started step 4; you enter them there.

## Team slug

In tracezilla, a company is a team. The team slug is the first part of the
browser URL when the company is open, for example `dev-woocommerce-abc1` in:

```text
https://app.tracezilla.com/dev-woocommerce-abc1/dashboard/home
```

The slug is not necessarily the same as the displayed company name. In
`.env`, it goes here:

```dotenv
TRACEZILLA_TEAM_SLUG=dev-woocommerce-abc1
```

The team slug is not secret.

## OAuth client and API user

Every integration logs in to tracezilla with the **Resource Owner Password
Credentials** you create in tracezilla. They have four values: an OAuth client
(Client ID and Client Secret) and an API user that tracezilla generates with
it (a username ending in `@tracezilla.api`, and a password). The API user is
not a person's login, so the integration keeps working when employees leave,
and nobody ever types a password into it.

Create them as described in the
[tracezilla API documentation](https://tech.tracezilla.com/docs/rest-api-v1/making-requests/authentication.html#bearer-token-and-resource-owner-password-credentials),
and copy all four values into `.env`:

```dotenv
TRACEZILLA_CLIENT_ID=...
TRACEZILLA_CLIENT_SECRET=...
TRACEZILLA_USERNAME=...@tracezilla.api
TRACEZILLA_PASSWORD=...
```

There is no separate login step. The first command that needs tracezilla logs
in with these values and saves the tokens in `var/tracezilla-token.json`.
Later commands use the saved token and renew it 30 days before it expires. If
tracezilla refuses the renewal, or the token file is gone, the next command
simply logs in again.

## Keep credentials secret

The Client Secret, the API user's password, and the token file are secrets. Never commit or share
`.env` or `var/`. If a secret is exposed, revoke it in tracezilla immediately
and create a new one.

## Official reference

- [tracezilla API authentication](https://tech.tracezilla.com/docs/rest-api-v1/making-requests/authentication.html#bearer-token-and-resource-owner-password-credentials)
