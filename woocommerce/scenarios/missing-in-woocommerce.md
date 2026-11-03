---
title: Missing in WooCommerce
layout: default
parent: Scenarios
grand_parent: WooCommerce
nav_order: 20
---

# Missing in WooCommerce

{: .label .label-green }
Read only

This template lists the tracezilla SKUs that the shop does not sell. Go through
the list with the customer: for each SKU, decide whether it belongs in the
shop. It is the tracezilla side of [Compare Catalogs](./compare-catalogs.html).

## Create and run it

```bash
docker compose run --rm php php bin/bifrost-connect scenario:create shop-decisions --template=missing-in-woocommerce
docker compose run --rm php composer test
docker compose run --rm php php bin/bifrost-connect scenario:run shop-decisions
```

Without Docker, leave out `docker compose run --rm php`. `shop-decisions` is
your scenario's name; choose any name.

## What it reads

The same as Compare Catalogs: every product and variation the shop sells, and
every tracezilla SKU.

## What it reports

```json
{
    "status": "ok",
    "summary": {
        "decide_with_customer": 1,
        "not_for_the_shop": 1,
        "inactive_tracezilla_skus_skipped": 2
    },
    "decide_with_customer": [
        {
            "sku": "TZ-GIFT-001",
            "tracezilla_id": "dc0eea55-77b2-4225-84f6-d3f6f1e654b9",
            "name": "Gourmet Gift Box",
            "category": "finished_good"
        }
    ],
    "not_for_the_shop": [
        {
            "sku": "TZ-RAW-001",
            "tracezilla_id": "6d80bd84-6194-42d4-a19e-920f3bb4c4c6",
            "name": "Raw Cocoa Beans 25 kg",
            "category": "raw_material",
            "reason": "category is raw_material"
        }
    ]
}
```

This is the result against the sandbox and a development company filled with
the [test dataset](../../fundamentals/seed.html): the gift box could be sold,
and the cocoa beans are a raw material.

| Section | Meaning | What to do with the customer |
|:--|:--|:--|
| `decide_with_customer` | Finished goods that may be sold, but are not in the shop | Decide for each: add it to the shop, or leave it out on purpose |
| `not_for_the_shop` | SKUs that can never be sold in the shop, with the reason | Usually nothing; check that the reasons are right |

Inactive tracezilla SKUs are left out and only counted.

## Decide with the customer

For each SKU the customer wants in the shop, the shop needs more than tracezilla
knows. Agree on:

- whether it becomes a new product, or a variation of an existing product;
- the product name, description, images, and categories;
- price and tax;
- whether WooCommerce manages its stock;
- whether it is published or kept as a draft.

The template only reports. It never creates products in WooCommerce.

## Rules you can change

The rules are in `BusinessRules.php`, marked `RULE`. Agree them with the
customer, change them, and change the matching examples in
`BusinessRulesTest.php`:

| Rule | Template default | Example change |
|:--|:--|:--|
| What can never be sold in the shop | SKUs with sales not allowed in tracezilla, and everything that is not a `finished_good` | Also leave out SKUs whose code starts with `TZ-B2B-` |
| Inactive tracezilla SKUs | Ignored | Report them separately |
| How SKUs are compared | Surrounding spaces are ignored; upper and lower case differ | Ignore case: return `strtoupper(trim($sku))` in `comparable()` |

## Safety

The scenario only reads. It never creates or changes products, variations, or
SKUs, and it never prints credentials. The command exits with `0` unless a
request fails.
