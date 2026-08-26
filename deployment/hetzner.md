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
`custom/`—has already been tested on the developer's computer. See the
[deployment overview](../deployment.html) for context.

## Requirements

- SSH access
- PHP 8.3 or newer
- Composer 2
- `rsync` on the developer's computer and server
- Cron
- [Shopify and tracezilla credentials](../getting-started.html#prerequisites)

## 1. Connect with SSH

```bash
ssh -p YOUR_SSH_PORT YOUR_SSH_USER@YOUR_SERVER
```

## 2. Prepare the server folder

```bash
mkdir -p ~/apps
mkdir -p ~/apps/YOUR_PROJECT_NAME
```

## 3. Upload the tested project

Run this from the project directory on the developer's computer, not from the
SSH session:

```bash
rsync -az --delete \
  --exclude='.git/' \
  --exclude='.env' \
  --exclude='vendor/' \
  --exclude='var/' \
  -e 'ssh -p YOUR_SSH_PORT' \
  ./ YOUR_SSH_USER@YOUR_SERVER:~/apps/YOUR_PROJECT_NAME/
```

This makes the deployed source match the tested local source. The exclusions
protect server credentials, runtime files, installed packages, and local
version-control metadata.

## 4. Install production dependencies

Return to the SSH session:

```bash
cd ~/apps/YOUR_PROJECT_NAME
composer install --no-dev --no-interaction --prefer-dist --optimize-autoloader
```

## 5. Configure the credentials

```bash
test -f .env || cp .env.example .env
chmod 600 .env
```

Open `.env` with the editor available on the server and enter the Shopify and
tracezilla values. Never display the completed file in terminal output or
upload or share it.

## 6. Prepare persistent runtime files

```bash
mkdir -p var
chmod 700 var
```

The same hosting user must own the application, runtime directory, and cron
task.

## 7. Verify the deployment

```bash
php bin/tracezilla-integration deployment:check
php bin/tracezilla-integration connection:check
```

Continue only when the deployment reports `"ready": true` and both Shopify
and tracezilla report a successful connection. The connection check is
read-only.

## 8. Deploy an update

Test the change locally, run the same `rsync` command again, and then run on
the server:

```bash
cd ~/apps/YOUR_PROJECT_NAME
composer install --no-dev --no-interaction --prefer-dist --optimize-autoloader
php bin/tracezilla-integration deployment:check
php bin/tracezilla-integration connection:check
```

The server receives only the version currently present in the developer's
project directory. Nothing changes on an existing customer installation until
the developer uploads another tested version.

## 9. Schedule the integration

The application is installed but does not run continuously. Follow
[Set up a cron task](./cron/setup.html) after selecting and manually testing
the customer workflow.
