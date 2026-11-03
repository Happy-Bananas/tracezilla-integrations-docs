---
title: Development Company
layout: default
parent: tracezilla Prerequisites
nav_order: 5
---

# Development company

Integration development needs its own tracezilla company that contains no
customer or production data. The seed and the write commands create partners,
SKUs, and orders, and they must never run against a live company.

Create a new company as described in the tracezilla tutorial
[Create company](https://www.tracezilla.com/en/tutorials/create-company). Give
it a name that is clearly a test company, for example `DEV-WooCommerce`.

## Next step

Continue with [Credentials](./account-and-team.html) to add the company's team
slug and authentication to `.env`. Then fill the company with the
[standard test dataset](./seed.html).
