---
title: Create tracezilla SKUs
layout: default
parent: Scenarios
grand_parent: WooCommerce
nav_order: 30
---

# Create tracezilla SKUs

{: .label .label-yellow }
Writes with explicit confirmation

This template creates a tracezilla SKU for every product the shop sells that
tracezilla does not know yet. Inventory synchronization and order import need
those SKUs. It writes only to tracezilla, never to WooCommerce.

Start with [Compare Catalogs](./compare-catalogs.html) to see the differences,
and agree the mapping rules below with the customer before you write anything.

## Create it and preview the plan

```bash
docker compose run --rm php php bin/bifrost-connect scenario:create sku-sync --template=create-tracezilla-skus
docker compose run --rm php composer test
docker compose run --rm php php bin/bifrost-connect scenario:run sku-sync
```

Without Docker, leave out `docker compose run --rm php`. `sku-sync` is your
scenario's name; choose any name.

The last command is a dry run. It writes nothing and shows the plan:

```json
{
    "status": "ok",
    "summary": {
        "to_create": 194,
        "skipped": 0,
        "already_in_tracezilla": 7,
        "weight_assumed": 194
    },
    "skipped": [],
    "weight_assumed": ["TZ-BAKERY-001", "TZ-BAKERY-002"],
    "changes": [
        {
            "action": "create_tracezilla_sku",
            "key": "TZ-BAKERY-001",
            "data": {
                "sku_code": "TZ-BAKERY-001",
                "global_name": "Sourdough Rye Bread 750 g",
                "category": "finished_good",
                "unit_of_measure": "pcs",
                "lot_unit": "pcs",
                "default_uom_conversion": 1,
                "weight_factor_net": 1,
                "weight_factor_gross": 1
            },
            "outcome": "planned"
        }
    ],
    "mode": "dry_run"
}
```

The lists are shortened here. This is the result against the sandbox and a
development company filled with the [test dataset](../../fundamentals/seed.html).
The sandbox products have no weight, so every weight is assumed.

| Section | Meaning |
|:--|:--|
| `changes` | The SKUs that would be created, with exactly the data that would be sent |
| `skipped` | Products that cannot become an SKU, with the reason |
| `weight_assumed` | SKUs that got the assumed weight because the shop has none. Check these with the customer. |

## Write

Check the plan, then create one SKU and look at it in tracezilla:

```bash
docker compose run --rm php php bin/bifrost-connect scenario:run sku-sync --execute --confirm --limit=1
```

When it is right, run it without `--limit` to create the rest. Running it
again is safe: SKUs that exist are not created again.

## Rules you can change

The rules are in `BusinessRules.php`, marked `RULE`. Agree them with the
customer, change them, and change the matching examples in
`BusinessRulesTest.php`:

| Rule | Template default | Example change |
|:--|:--|:--|
| How a product becomes an SKU | A `finished_good`, sold and stored per piece (`pcs`), named like the product | A product sold per kg: `unit_of_measure` `kg` |
| Weight without a weight in the shop | 1 kg per piece, listed under `weight_assumed` | Skip the product until it has a weight |
| Gross weight | Equal to the net weight | Add the packaging weight |
| How SKUs are compared | Surrounding spaces are ignored; upper and lower case differ | Ignore case: return `strtoupper(trim($sku))` in `comparable()` |

The shop's weight is converted to kg from the store's weight unit (kg, g, lbs,
or oz). A SKU that exists in tracezilla, even an inactive one, is never created
again, because tracezilla does not allow the same SKU code twice. A SKU longer
than 17 characters is skipped, because tracezilla rejects it.

## Safety

Nothing is written without `--execute --confirm`. One failed SKU does not stop
the others; it is reported with tracezilla's reason, and the command then exits
with `1`. The scenario takes the integration's run lock, so it never runs at
the same time as another scheduled command.
