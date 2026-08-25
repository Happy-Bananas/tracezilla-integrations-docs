---
title: Manual Installation
layout: default
nav_order: 11
---

# Manual installation

This is the step-by-step alternative to the one-command setup in
[Getting Started](./getting-started.html).

You need Docker with Docker Compose and Git. A GitHub account is not required
to download or run the public project.

## 1. Choose a project name

Run the following from the directory where you keep development projects.
Replace `my-shopify-integration` with the folder name you want:

```bash
git clone https://github.com/Happy-Bananas/tracezilla-shopify-php.git my-shopify-integration
cd my-shopify-integration
git remote rename origin template
```

## 2. Prepare the local configuration

Create your `.env` file and tell Docker which local user owns the project:

```bash
cp .env.example .env
printf '\nTRACEZILLA_DOCKER_UID=%s\nTRACEZILLA_DOCKER_GID=%s\n' "$(id -u)" "$(id -g)" >> .env
```

Open `.env` in your editor and add the Shopify and tracezilla credentials.
Git ignores `.env`, so credentials are not included when you commit.
See the [Getting Started prerequisites](./getting-started.html#prerequisites)
if the two test systems are not ready yet.

## 3. Check the connections

```bash
./check-connection
```

The first run can take a little while because Docker builds the development
environment and Composer installs the PHP packages. A successful check names
the Shopify store and confirms the tracezilla connection.

## 4. Decide where the code should live

The integration now works locally. You can continue without creating an online
repository.

For backup, collaboration, or deployment, create an empty **private**
repository in GitHub, GitLab, or another Git service. Then connect and push the
project using the instructions shown by that service. For GitHub over SSH, the
commands normally look like:

```bash
git remote add origin git@github.com:YOUR-ACCOUNT/YOUR-PRIVATE-REPOSITORY.git
git push -u origin main
```

After this, `origin` is your private customer project. `template` is the public
Happy Bananas project from which you can retrieve future framework updates.
Having a public template remote does not give you permission to push customer
code to the Happy Bananas repository.

## Next

- [Generate custom business logic](./shopify/php/custom-business-logic.html)
- [Deploy and schedule with cron](./deployment/)
