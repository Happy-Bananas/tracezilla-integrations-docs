---
title: Deploy on Hetzner
layout: default
parent: Deployment
nav_order: 5
---

# Deploy on Hetzner

This example installs the headless PHP application in a Hetzner hosting
account with SSH access. Replace every uppercase value with the value supplied
for the hosting account.

It assumes that the complete customer project—including the rules under
`custom/`—has already been tested and pushed to a private Git repository. See
the [deployment overview](../deployment.html#from-template-to-server) for the
file flow.

## Requirements

- SSH access
- PHP 8.3 or newer
- Composer 2
- Git
- Cron
- A private customer repository that the server can read
- [Shopify and tracezilla credentials](../getting-started.html#prerequisites)

## 1. Connect with SSH

```bash
ssh -p YOUR_SSH_PORT YOUR_SSH_USER@YOUR_SERVER
```

## 2. Get the application files

```bash
mkdir -p ~/apps
cd ~/apps
git clone YOUR_PRIVATE_REPOSITORY_URL YOUR_PROJECT_NAME
cd YOUR_PROJECT_NAME
git remote set-url --push origin DISABLED
```

The deployment server needs read access to that private repository. The final
command disables pushes from the server while still allowing approved updates
to be fetched from the private `origin` repository.

## 3. Install production dependencies

```bash
composer install --no-dev --no-interaction --prefer-dist --optimize-autoloader
```

## 4. Configure the credentials

```bash
cp .env.example .env
chmod 600 .env
```

Open `.env` with the editor available on the server and enter the Shopify and
tracezilla values. Never display the completed file in terminal output or
commit it to Git.

## 5. Prepare persistent runtime files

```bash
mkdir -p var
chmod 700 var
```

The same hosting user must own the application, runtime directory, and cron
task.

## 6. Verify the deployment

```bash
php bin/tracezilla-integration deployment:check
php bin/tracezilla-integration connection:check
```

Continue only when the deployment reports `"ready": true` and both Shopify
and tracezilla report a successful connection. The connection check is
read-only.

## 7. Schedule the integration

The application is installed but does not run continuously. Follow
[Set up a cron task](./cron/setup.html) after selecting and manually testing
the customer workflow.
