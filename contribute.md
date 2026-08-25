---
title: Contribute
layout: default
nav_order: 100
---

# Contribute

You do not need write access to become a contributor. Public framework and
documentation improvements are proposed through a GitHub pull request and
reviewed by the maintainers.

Customer credentials and customer-specific business rules are private work.
Keep them in the customer's repository; do not include them in a public pull
request.

## 1. Choose the repository

- [PHP integration](https://github.com/Happy-Bananas/tracezilla-shopify-php)
- [Documentation](https://github.com/Happy-Bananas/tracezilla-integrations-docs)

Use the PHP repository for reusable application behavior, commands, tests, and
bug fixes. Use the documentation repository for guides and corrections.

## 2. Fork and clone it

Open the repository on GitHub, select **Fork**, and create the fork in your own
account. Then clone your fork:

```bash
git clone https://github.com/YOUR-ACCOUNT/REPOSITORY.git
cd REPOSITORY
git remote add upstream https://github.com/Happy-Bananas/REPOSITORY.git
```

In this contributor setup, `origin` is your fork and `upstream` is the public
Happy Bananas repository.

## 3. Create a branch

Start from the current public `main` branch:

```bash
git fetch upstream
git switch main
git merge --ff-only upstream/main
git switch -c describe-the-change
```

Keep the branch focused on one bug, feature, or documentation improvement.

## 4. Make and verify the change

For the PHP application:

```bash
docker compose up -d integration
docker compose exec integration composer test
```

For the documentation:

```bash
docker compose up --build
```

Open <http://localhost:4000/> and review the changed pages. The documentation
repository README contains the complete production-build and link-check
command.

Never add `.env`, API keys, Shopify secrets, production exports, customer
names, or other private data.

## 5. Open a pull request

Commit and push the branch to your fork:

```bash
git add PATHS-YOU-CHANGED
git commit -m "Describe the change"
git push -u origin describe-the-change
```

GitHub will offer to create a pull request from your branch. Explain the
problem, the proposed change, and how you verified it. A maintainer may request
adjustments before merging.

Once a pull request is accepted, you have contributed to the project. Direct
push access to the public `main` branch is not required.
