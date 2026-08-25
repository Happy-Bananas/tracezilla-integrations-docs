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

The [Getting Started guide](../getting-started.html#2-add-credentials) explains
where to find both values while configuring a project.

## Create an API token

1. Sign in to the intended test team.
2. Open your account or personal settings.
3. Open **API Tokens**.
4. Create a token for the integration.

The exact labels and available permissions can change. Review the current
[tracezilla API documentation](https://app.tracezilla.com/api/documentation)
and account interface when creating a token.

If broad access is temporarily required to establish a test, record that
decision and reduce the token's access before production adoption.

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
