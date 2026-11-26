---
name: commonsku-api-items
description: "Read and update order items in commonsku through the API, change margins on several items at once, and add a product to an order. Use this when someone reads or edits a line on a commonsku order, applies a margin in bulk, or adds a product from a supplier catalogue to an order."
---

# Items

## What this skill helps you do

This skill covers items in commonsku. An item is one line on an order. You can read an item, update it, change margins on several items at once, and add a product to an order. It does not cover item costs, item locations or item decorations. For sizes and colours, use the `commonsku-api-breakdowns` skill.

Check each operation's current fields in the spec you downloaded. This skill does not repeat them.

## Steps

### Get an item

Call `GET /item/{id}`. The response takes one of six shapes: product, service, separator, option, artwork PO or order item. For the reorder (resku) view, pass `resku=true`. For the normal view, leave the parameter out entirely.

### Update an item

Call `PUT /item/{id}` with `item_name` and the fields you want to change.

Example request, with made-up values:

```sh
curl -sS -X PUT "$COMMONSKU_BASE_URL/item/7b3c9e21-0000-4000-8000-000000000001" \
  -H "x-api-key: <your-api-key>" \
  -H "Content-Type: application/json" \
  -d '{ "item_name": "Example Tote Bag", "total_units": 250 }'
```

### Update margins in bulk

Call `PUT /item/{id}` with `actionName` set to `bulk-update-margin`, plus `orderId`, `itemIds` (an array), `margin` and, optionally, `includeRunCharges` (default `false`).

### Add a product to an order

Call `POST /items/product` with `order_id`, `parent_id` (the product's ID in its source) and `source`. Use `index` to set the display position.

Example request, with made-up values:

```sh
curl -sS -X POST "$COMMONSKU_BASE_URL/items/product" \
  -H "x-api-key: <your-api-key>" \
  -H "Content-Type: application/json" \
  -d '{ "order_id": "2e8f4a63-0000-4000-8000-000000000002", "parent_id": "example-product-1001", "source": "ps-products" }'
```

The response is `201` with the new item.

## Gotchas

- `GET /item/{id}` and `PUT /item/{id}` carry the spec's Beta Feature label. `POST /items/product` does not.
- `item_name` is required on every single-item update, even to change only the price. Send the current name.
- The bulk margin body uses camelCase (`actionName`, `orderId`, `itemIds`). The rest of the API uses snake_case.
- If you leave out `source`, `parent_id` is treated as a personal or commonsku product ID. Send `source` for any other catalogue, such as `ps-products` for PromoStandards.
- `POST /items/product` has no size or colour fields.

## Known issues

None known.

## What the API cannot do

- **Delete an item.** There is no `DELETE` operation for items.
- **Explain everything.** The spec does not list the other `field_lock` values or say what it controls, which item ID goes in the path for a bulk margin update, whether `price`, `subtotal` and `total` are recalculated, or what errors `POST /items/product` returns.

Last verified: 2026-11-26
