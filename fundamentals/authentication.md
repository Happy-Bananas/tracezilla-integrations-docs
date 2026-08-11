---
title: Authentication
layout: default
parent: tracezilla Fundamentals
nav_order: 20
---

# Authentication and secrets

The tracezilla examples use an API token as a Bearer token for requests to one
team-specific API path. The team slug identifies the destination team but does
not authenticate the request by itself.

## Create an API token

1. Sign in to the intended test team.
2. Open your account or personal settings.
3. Open **API Tokens**.
4. Create a token for the integration.
5. Select the least privilege that supports the chosen workflow.
6. Copy the token when it is displayed and store it in a password manager or
   encrypted secret store.

The exact labels and available permissions can change. Review the current
[tracezilla API documentation](https://app.tracezilla.com/api/documentation)
and account interface when creating a token.

If broad access is temporarily required to establish a test, record that
decision and reduce the token's access before production adoption.

## Required client configuration

Every implementation needs equivalents of:

| Value | Meaning | Example |
|:--|:--|:--|
| Base URL | Application origin without a team API path | `https://app.tracezilla.com` |
| Team slug | Team identifier inserted into the API path | `integration-demo` |
| API token | Secret Bearer token | Never show an example value |
| Request timeout | Maximum complete-request duration | Implementation-specific |
| Connection timeout | Maximum connection-establishment duration | Implementation-specific |

A resulting client base URL normally has this shape:

```text
https://app.tracezilla.com/api/v1/{team-slug}
```

Do not accept a base URL containing an unexpected team path and then append a
second team path to it.

## Secret-handling rules

- Use an uncommitted local environment file or the platform's encrypted secret
  store.
- Keep tokens out of source code, Git, screenshots, exported scenarios,
  support messages, URLs, logs, and exception details.
- Never print Authorization headers or complete configuration arrays.
- Redact secrets before sharing a request or error.
- Rotate a token immediately if it reaches Git history or another uncontrolled
  location; deleting the visible file is insufficient.
- Use separate credentials and teams for testing and production.

## Example environment names

Focused code repositories use these conventional names where appropriate:

```dotenv
TRACEZILLA_BASE_URL=https://app.tracezilla.com
TRACEZILLA_TEAM_SLUG=integration-demo
TRACEZILLA_API_KEY=secret-value
TRACEZILLA_TIMEOUT=30
TRACEZILLA_CONNECT_TIMEOUT=10
```

Variable names and timeout units may differ by platform. Configuration belongs
to the implementation repository; real values never belong in documentation.

Continue with [Validate the tracezilla connection](./connection-validation.html).

Canonical tracezilla authentication and secret-handling guidance will be
migrated here.
