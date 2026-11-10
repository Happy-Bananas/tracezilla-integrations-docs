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
without `--without-seed`, and copy its `bin/seed-tracezilla` into your project;
see [Seed the Test Dataset](./fundamentals/seed.html).

### `Seed failed:` with a message about tracezilla

The seed uses the same tracezilla credentials as every other command. When it
stops with one of these messages, check the credentials as described in
[The connection to tracezilla doesn't work](#the-connection-to-tracezilla-doesnt-work):

```text
Seed failed: Missing required environment variable: TRACEZILLA_CLIENT_ID
Seed failed: tracezilla token request failed with HTTP 400: … Check TRACEZILLA_CLIENT_ID, …
Seed failed: GET /partners failed with HTTP 404: { "message": "" }
```

### `tracezilla has no partner number 1`

```text
Seed failed: tracezilla has no partner number 1, which is the own company of every tracezilla company.
```

The seed adds its warehouse to your own company's partner. tracezilla creates
that partner, with number 1, together with every company, so the seed looks it
up by that number. Check that the team in `TRACEZILLA_TEAM_SLUG` is the
development company you created; see
[Development Company](./fundamentals/development-company.html).

The seed's output names the partner it uses on its first line, for example
`own_company … Banana McShopface (partner number 1)`. If that is not your
company, stop before `--execute` and check the team.

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
| `Missing required environment variable: TRACEZILLA_CLIENT_ID` (or `_CLIENT_SECRET`, `_USERNAME`, `_PASSWORD`, `_TEAM_SLUG`, `_BASE_URL`) | The setting is missing from `.env` | Add it, see [Credentials](./fundamentals/account-and-team.html) |
| `tracezilla token request failed with HTTP 400: {"error":"invalid_grant" …` | tracezilla refused the login: the API user's username or password is wrong | Copy `TRACEZILLA_USERNAME` and `TRACEZILLA_PASSWORD` again from the Resource Owner Password Credentials. Use the API user (`…@tracezilla.api`), never your own login |
| `tracezilla token request failed with HTTP 401: {"error":"invalid_client" …` | The Client ID or Client Secret is wrong, or the client was deleted | Copy them again from tracezilla, or create new credentials and update `.env` |
| `GET /skus failed with HTTP 404` | The team slug is wrong | Copy the team slug from the browser address when the company is open in tracezilla |

### Check the credentials

1. `.env` contains `TRACEZILLA_TEAM_SLUG`, `TRACEZILLA_CLIENT_ID`,
   `TRACEZILLA_CLIENT_SECRET`, `TRACEZILLA_USERNAME`, and
   `TRACEZILLA_PASSWORD`, copied exactly from tracezilla. See
   [Credentials](./fundamentals/account-and-team.html).
2. The username is the API user that tracezilla created with the credentials
   (it ends in `@tracezilla.api`), not the login you use in the browser.
3. To start over, delete `var/tracezilla-token.json`. The next command logs in
   again with the values in `.env`.

Then run the connection check again.
