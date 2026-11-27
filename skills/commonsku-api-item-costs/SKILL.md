---
name: commonsku-api-item-costs
description: "Add, read, update and delete the extra charges on a commonsku order line through the API, such as setup, freight, rush and run charges, and keep a cost internal so the client never sees it. Use this when someone adds or changes a charge on an order item, hides a cost from the client invoice, or changes costs on an invoiced or locked order."
---

# Item costs

## What this skill helps you do

This skill covers item costs in commonsku. An item cost is an extra charge on an order line, such as setup, freight, rush or a run charge. You can create, read, update and delete item costs. There is no list operation: costs come back with their item, as the `commonsku-api-items` skill describes.

Check each operation's current fields in the spec you downloaded. This skill does not repeat them.

## Steps

### Add a cost

1. Call `POST /item_costs` with `item_id`. Set `item_cost_type` and `item_cost_title` if you need them. The type defaults to `MISCELLANEOUS`.
2. Keep the new cost's ID from the response.
3. Call `PUT /item_costs/{id}` to set `unit_cost`, `unit_price` and `hidden`. Create cannot set them.

Example request, with made-up values:

```sh
curl -sS -X POST "$COMMONSKU_BASE_URL/item_costs" \
  -H "x-api-key: <your-api-key>" \
  -H "Content-Type: application/json" \
  -d '{ "item_id": "7b3c9e21-0000-4000-8000-000000000001", "item_cost_type": "SETUP", "item_cost_title": "Example setup charge", "quantity": 1 }'
```

The response is `201` with the cost, the item and the order.

### Keep a cost internal

To track a cost without the client seeing it, set `hidden` to `1` and `unit_price` to `0` with `PUT /item_costs/{id}`. `hidden: 1` keeps the cost off the client invoice. `unit_price: 0` keeps it out of the sales order total. The cost still counts in the order's internal cost and margin.

### Update a cost

Call `PUT /item_costs/{id}`. When you change the margin or price, `field_lock` decides which field stays fixed: `unit_cost` (the default) or `unit_price`. To keep the selling price unchanged, send `field_lock: "unit_price"`.

### Delete a cost

Call `DELETE /item_costs/{id}`. The response returns the item and the order.

## Gotchas

- Leave out `quantity` for run charges and for fixed charges on option items. It defaults to `1`. On an option item, `option: true` makes a run charge.
- `hidden: 1` alone does not keep a cost out of the sales order total. Set `unit_price` to `0` as well.
- You can add and update item costs through the API after a sales order is invoiced or locked. The lock restricts pricing changes in commonsku, not through the API. A cost with a non-zero `unit_price` still changes the sales order total. Changing costs on an invoiced or locked order this way gets around the lock in commonsku. Confirm with the user before you do it.
- `DELETE` returns `400` when the cost cannot be deleted, for example when a credit memo is linked to it.
- A cost added after a purchase order exists reaches that purchase order only when it is regenerated. See the `commonsku-api-bills` skill.

## Known issues

None known.

## What the API cannot do

- **List costs on their own.** Read them from their item.
- **Add a cost to a purchase order only.** See the `commonsku-api-bills` skill.
- **Name the capability.** `POST`, `PUT` and `DELETE` return `403` for a missing capability, but the spec does not say which one. See the `commonsku-api-setup` skill.

Last verified: 2026-11-26
