---
title: Troubleshooting
layout: default
nav_order: 70
---

# Troubleshooting

Find the message you see, then follow the fix. Run every command in your
integration project folder. With Docker, Docker must be running first.

## The seed doesn't work

### `Unknown command: tracezilla:seed`

The seed is not installed: the project was created with `--without-seed`. To
check, list the commands and look for `tracezilla:seed`.

With Docker:

```bash
docker compose run --rm php php bin/bifrost-connect help
```

Without Docker:

```bash
php bin/bifrost-connect help
```

To add the seed, create a second, temporary project of the same platform
without `--without-seed`, and copy its `bin/seed-tracezilla` into your project.
Then add `TRACEZILLA_OWN_COMPANY` to `.env`;
see [Seed the Test Dataset](./fundamentals/seed.html).

### `Seed failed:` with a message about tracezilla

The seed uses the same tracezilla credentials as every other command. When it
stops with one of these messages, check the credentials as described in
[The connection to tracezilla doesn't work](#the-connection-to-tracezilla-doesnt-work):

```text
Seed failed: Missing required environment variable: TRACEZILLA_CLIENT_ID
Seed failed: No tracezilla token is stored. Run bin/bifrost-connect tracezilla:login first.
Seed failed: GET /partners failed with HTTP 404: { "message": "" }
```

### `No tracezilla partner is named […]`

```text
Seed failed: No tracezilla partner is named [Your company]. Check TRACEZILLA_OWN_COMPANY.
```

`TRACEZILLA_OWN_COMPANY` in `.env` must be the exact name of your own company's
partner in tracezilla. Copy the name from tracezilla.

## The connection to tracezilla doesn't work

Run the connection check; its `tracezilla:` line names the problem.

With Docker:

```bash
docker compose run --rm php php bin/bifrost-connect connection:check
```

Without Docker:

```bash
php bin/bifrost-connect connection:check
```

| Message | Cause | Fix |
|:--|:--|:--|
| `Missing required environment variable: TRACEZILLA_CLIENT_ID` (or `_CLIENT_SECRET`, `_TEAM_SLUG`, `_BASE_URL`) | The setting is missing from `.env` | Add it, see [Credentials](./fundamentals/account-and-team.html) |
| `No tracezilla token is stored. Run bin/bifrost-connect tracezilla:login first.` | You have not logged in from this project, or `var/` was deleted | Log in, see below |
| `GET /skus failed with HTTP 404` | The team slug is wrong | Copy the team slug from the browser address when the company is open in tracezilla |
| `The refresh token was rejected; run bin/bifrost-connect tracezilla:login again.` | The OAuth client was deleted, or the login was revoked | Log in again. If the client was deleted, create a new one and update `.env` first. |
| `tracezilla token request failed with HTTP 401 … Check TRACEZILLA_CLIENT_ID, TRACEZILLA_CLIENT_SECRET, and the username and password.` | The login was refused | Check the Client ID and Client Secret in `.env`, and your tracezilla username and password |

### Check the credentials

1. `.env` contains `TRACEZILLA_TEAM_SLUG`, `TRACEZILLA_CLIENT_ID`, and
   `TRACEZILLA_CLIENT_SECRET`, copied exactly from tracezilla. See
   [Credentials](./fundamentals/account-and-team.html).
2. You logged in from this project folder. The login stores its token in
   `var/` of the project it ran in, so another project needs its own login.
3. Log in again if you are unsure. The password is used once and not stored.

With Docker:

```bash
docker compose run --rm php php bin/bifrost-connect tracezilla:login
```

Without Docker:

```bash
php bin/bifrost-connect tracezilla:login
```

Then run the connection check again.
