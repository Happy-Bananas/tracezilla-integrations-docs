---
title: Your Computer
layout: default
nav_order: 15
---

# Your computer

BifrostConnect can run in two ways on Mac, Linux, and Windows. Both give the
same project, and you can switch later.

| | With Docker (recommended) | Without Docker |
|:--|:--|:--|
| You install | Docker | PHP, Composer, and for WooCommerce a local WordPress |
| PHP and package versions | Pinned in the project | Whatever you have installed |
| WooCommerce test store | Included, runs in Docker | You provide one |
| Terminal | Mac and Linux: Terminal. Windows: PowerShell. | The same |

How to install Docker, PHP, Composer, or WordPress is outside the scope of this
manual. Follow the official guides linked below.

## With Docker

Install [Docker](https://docs.docker.com/get-started/get-docker/):

- Mac and Windows: Docker Desktop.
- Linux: Docker Engine with the Compose plugin.

Nothing else is needed. PHP, Composer, WordPress, and the database all run in
containers.

On Windows, Docker Desktop sets up WSL 2 itself to run the containers. You do
not need to use WSL; run every command in PowerShell. Where Mac and Linux use a
shell script, this manual shows the PowerShell command next to it.

## Without Docker

Install:

- [PHP](https://www.php.net/manual/en/install.php) 8.3 or newer.
- [Composer](https://getcomposer.org/download/) 2.
- For WooCommerce, a WordPress site with WooCommerce for testing, for example
  a local WordPress installation or an existing test store. See
  [Development Store](./woocommerce/setup/store.html).

Setting up and maintaining these is your responsibility. The manual assumes
they work. WSL is not needed on Windows.

## Components per platform

What each platform needs, and where it runs. Online accounts are the same with
and without Docker.

### WooCommerce

| Component | With Docker | Without Docker |
|:--|:--|:--|
| Integration (PHP 8.3, Composer packages) | Container, created with the [create command](./getting-started.html#4-create-the-project) | Your PHP and Composer |
| Test store (WordPress, WooCommerce, database) | The [sandbox](./woocommerce/setup/store.html), in containers | Your own WordPress with WooCommerce |
| tracezilla development company and OAuth client | Online, see [tracezilla Prerequisites](./tracezilla-fundamentals.html) | The same |

### Shopify

| Component | With Docker | Without Docker |
|:--|:--|:--|
| Integration (PHP 8.3, Composer packages) | Container, created with the [create command](./getting-started.html#4-create-the-project) | Your PHP and Composer |
| Development store and app | Online, see [Shopify Prerequisites](./shopify/setup.html) | The same |
| tracezilla development company and personal access token | Online, see [tracezilla Prerequisites](./tracezilla-fundamentals.html) | The same |
