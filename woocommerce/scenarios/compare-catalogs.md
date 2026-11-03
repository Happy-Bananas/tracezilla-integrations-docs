---
title: Compare Catalogs
layout: default
parent: Scenarios
grand_parent: WooCommerce
nav_order: 10
---

# Compare catalogs

{: .label .label-green }
Read only

This template compares what the shop sells with the SKUs in tracezilla. Use it
before you synchronize inventory or import orders: both need every product the
shop sells to exist in tracezilla with the same SKU.

## Create and run it

```bash
docker compose run --rm php php bin/bifrost-connect scenario:create product-check --template=compare-catalogs
docker compose run --rm php composer test
docker compose run --rm php php bin/bifrost-connect scenario:run product-check
```

Without Docker, leave out `docker compose run --rm php`. `product-check` is
your scenario's name; choose any name.

## What it reads

| From | What |
|:--|:--|
| WooCommerce | Every simple product, and the variations of variable products instead of the variable product itself. Customers buy variations, so a variable product's own SKU is not compared. |
| tracezilla | Every SKU |

It reads all pages, so a run on a large shop takes a while: one request per
100 products, plus one per variable product.

## What it reports

```json
{
    "status": "ok",
    "summary": {
        "matched": 7,
        "only_in_woocommerce": 194,
        "only_in_tracezilla": 1,
        "woocommerce_without_sku": 0,
        "duplicate_skus": 0,
        "inactive_tracezilla_skus_skipped": 2
    },
    "matched": [
        { "sku": "TZ-COFFEE-001-01", "woocommerce_name": "House Blend Coffee Beans - 250 g", "tracezilla_name": "House Blend Coffee Beans 250 g" }
    ],
    "only_in_woocommerce": [
        { "sku": "TZ-BAKERY-001", "name": "Sourdough Rye Bread 750 g" }
    ],
    "only_in_tracezilla": [
        { "sku": "TZ-RAW-001", "name": "Raw Cocoa Beans 25 kg" }
    ],
    "woocommerce_without_sku": [],
    "duplicate_skus": { "woocommerce": [], "tracezilla": [] }
}
```

The lists are shortened here. This is the result against the sandbox and a
development company filled with the [test dataset](../../fundamentals/seed.html).

| Section | Meaning | What to do with the customer |
|:--|:--|:--|
| `matched` | The SKU exists in both systems | Nothing; these are ready for inventory and orders |
| `only_in_woocommerce` | Sold in the shop, unknown in tracezilla | Create the SKUs in tracezilla, or fix the SKU in the shop |
| `only_in_tracezilla` | Known in tracezilla, not sold in the shop | Often fine, for example raw materials. Decide whether the shop should sell it. |
| `woocommerce_without_sku` | A product or variation without a SKU | Add a SKU in WooCommerce; it cannot be matched without one |
| `duplicate_skus` | The same SKU on several records in one system | Fix the data; the comparison never picks one of them |

Inactive tracezilla SKUs are left out and only counted.

## Rules you can change

The rules are in `BusinessRules.php`, marked `RULE`. Agree them with the
customer, change them, and change the matching examples in
`BusinessRulesTest.php`:

| Rule | Template default | Example change |
|:--|:--|:--|
| How SKUs are compared | Surrounding spaces are ignored; upper and lower case differ | Ignore case: return `strtoupper(trim($sku))` in `key()` |
| Inactive tracezilla SKUs | Left out | Compare them too, or report them separately |

To change what is read, for example only published products, filter in
`DataReader.php`:

```php
'woocommerce' => array_values(array_filter(
    $wooCommerce->catalog(),
    static fn (array $item): bool => $item['status'] === 'publish',
)),
```

## Safety

The scenario only reads. It never creates or changes products, variations, or
SKUs, and it never prints credentials. Differences are a normal result: the
command exits with `0` unless a request fails.
