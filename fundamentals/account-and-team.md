---
title: Test Account and Credentials
layout: default
parent: tracezilla Fundamentals
nav_order: 10
---

# Create a test account and acquire credentials

## 1. Create or select a test account

Use the official [tracezilla tutorials](https://www.tracezilla.com/en/tutorials)
to create the account. Use a demo or otherwise isolated team that contains no
customer production data.

If an integration project needs a demo extension or account assistance,
[contact tracezilla](https://www.tracezilla.com/en/contact-us).

## 2. Acquire the credentials

You need the team slug and an API key.

### Team slug

Open a page inside the intended team and identify the team-specific slug in
the browser URL. The slug identifies the team and is not necessarily the same
as the displayed company name. Add it to `.env` as `TRACEZILLA_TEAM_SLUG`.

### API key

1. Open your account or personal settings in the intended test team.
2. Open **API Tokens**.
3. Create a token for the integration.
4. Add it to `.env` as `TRACEZILLA_API_KEY`.

The team slug is not secret, but the API key is. Never commit or share the
API key. If it is exposed, rotate it immediately.
