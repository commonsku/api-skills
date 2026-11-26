---
name: commonsku-api-breakdowns
description: "Create, copy, update and delete the size and colour breakdowns of an order item in commonsku. Use this when someone changes quantities, sizes, colours or prices on an order line through the API, or matches order lines to another system by size and colour."
---

# Breakdowns

## What this skill helps you do

A breakdown is one size and colour combination on an order item, with its quantity, cost and price. New to the commonsku API? Start with the `commonsku-api-setup` skill.

## Steps

### Find breakdowns

`GET /item/{id}` returns the item's breakdowns with `breakdown_id`, `size_id`, `color_id`, `quantity`, `unit_cost` and `unit_price`. Take the item ID from the order's `item_uuid`.

### Create a breakdown

Look up `POST /breakdown` and send `item_id`, with `quantity`, `size_id`, `color_id`, `product_sku_id`, `unit_cost` or `unit_price` as needed. To copy an existing breakdown, send `target_type: "COPY"` and `target_id` instead.

```json
{ "item_id": "<item-uuid>", "quantity": 24, "size_id": "L", "color_id": "Navy" }
```

### Update or delete a breakdown

- `PUT /breakdowns/{id}`: send only the fields you are changing.
- `DELETE /breakdown/{id}`: removes the breakdown. The item stays on its order. The response returns the item and the order.

## Gotchas

- The paths differ: `POST /breakdown` and `DELETE /breakdown/{id}` are singular, `PUT /breakdowns/{id}` is plural.
- Breakdowns have no stable ID to sync on. When you create or update one, send the size and colour names in `size_id` and `color_id`, and commonsku matches them.
- Sales order and purchase order responses show size and colour names only. The IDs come from `GET /item/{id}`.
- A `403` means a missing permission. See the `commonsku-api-support` skill.

## Known issues

- **Breakdown edits don't change `date_updated` (confirmed 2026-11-23).** Editing a breakdown doesn't change `date_updated` on the sales order or purchase order, so a sync filtered on `date_updated` misses those edits. Re-read open orders on a schedule, within the rate limits in the `commonsku-api-setup` skill.

## What the API cannot do

- **Delete the item.** `DELETE /breakdown/{id}` removes only the breakdown.

Last verified: 2026-11-26
