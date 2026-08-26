---
title: Deploy with Ansible
layout: default
parent: Deployment
nav_order: 7
---

# Deploy with Ansible

{: .label .label-red }
Unsupported

{: .warning }
This is optional deployment automation. Server access, security, backups,
monitoring, Ansible, and the vault password remain the developer's or
customer's responsibility.

The BifrostConnect project includes a playbook under `deployment/ansible/`.
It uploads the project currently on the developer's computer, generates the
server `.env` from encrypted secrets, installs production PHP packages, and
checks the deployment and both service connections.

The playbook does not use Git on the server and does not configure cron.

## Requirements

On the developer's computer:

- Ansible
- SSH access to the server using a key

On the server:

- Python 3, used by Ansible
- PHP 8.3 or newer
- Composer 2

The hosting account does not need root access.

## 1. Create the inventory

From the BifrostConnect project root, run:

```bash
cp deployment/ansible/inventory.example.yml deployment/ansible/inventory.yml
```

Open `inventory.yml` and replace `YOUR_SERVER`, `YOUR_SSH_PORT`, and
`YOUR_SSH_USER`. The completed file is ignored by Git.

## 2. Configure the deployment

Open:

```text
deployment/ansible/group_vars/production/settings.yml
```

Set the absolute destination path, Shopify store URL, and tracezilla team slug.
The other non-secret defaults normally require no changes.

For example, the destination may be:

```yaml
bifrostconnect_path: /usr/home/YOUR_SSH_USER/apps/YOUR_PROJECT_NAME
```

## 3. Create the encrypted secrets

Run:

```bash
ansible-vault create \
  deployment/ansible/group_vars/production/vault.yml
```

Enter a new vault password and add these values in the editor opened by
Ansible:

```yaml
---
vault_shopify_client_id: YOUR_SHOPIFY_CLIENT_ID
vault_shopify_client_secret: YOUR_SHOPIFY_CLIENT_SECRET
vault_tracezilla_api_key: YOUR_TRACEZILLA_API_KEY
```

Ansible encrypts `vault.yml` before writing it to disk. The encrypted file can
be stored in a private customer repository. Do not commit an unencrypted copy
of these values, the vault password, or `.env`.

The vault protects secrets stored with the project. BifrostConnect still needs
a decrypted `.env` on the server; the playbook creates it with permission
`0600` and suppresses its contents from Ansible output.

## 4. Deploy

Test the project locally first, then run from its root directory:

```bash
ansible-playbook \
  -i deployment/ansible/inventory.yml \
  deployment/ansible/deploy.yml \
  --ask-vault-pass
```

The playbook:

1. Creates the destination and private runtime directory.
2. Uploads the current source and customer business rules through Ansible's
   normal SSH connection. No `rsync`, Git, or interactive server login is
   required.
3. Preserves the server's runtime files and installed packages. The local
   `.env`, `vendor/`, and Git data are never uploaded.
4. Generates the production `.env` from the encrypted vault.
5. Installs the locked production Composer packages.
6. Runs the deployment and read-only connection checks.

The deployment succeeds only when BifrostConnect is ready and can connect to
both Shopify and tracezilla.

## Deploy an update

Test the changes locally and run the same `ansible-playbook` command again.
The server receives the current project files; it does not pull later changes
from the public BifrostConnect repository.

## Vault password file

For personal automation, the vault password may be stored in a protected file
outside the project:

```bash
ansible-playbook \
  -i deployment/ansible/inventory.yml \
  deployment/ansible/deploy.yml \
  --vault-password-file ~/.config/bifrostconnect/vault-password
```

Restrict that file to your user and never commit or share it:

```bash
chmod 600 ~/.config/bifrostconnect/vault-password
```

After the first successful deployment, configure the required commands using
[Cron](./cron.html).
