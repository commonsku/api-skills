---
name: commonsku-api-tax-amounts
description: "Apply a tax to a commonsku order item or change which tax an item uses, with POST /tax-amount and PUT /tax-amount/{id}. Use this when someone taxes an order item through the API, needs the required fields or the tax_amount_id, or gets a 400, 403 or 404 from a tax-amount call."
---

# Tax amounts

## What this skill helps you do

A tax amount links one tax to one order item. These calls apply a tax to an item and switch an existing tax amount to a different tax. Both operations carry the spec label `Beta Feature`. The label does not limit access: a Beta endpoint works with your key like any other. It marks recently released endpoints that may still change, so retest after updates.

New to the commonsku API? Start with the `commonsku-api-setup` skill.

## Steps

### Apply a tax to an order item

1. Check the current schema of `POST /tax-amount` in the spec you downloaded.
2. Get the `item_id` and a `tax_id` that belongs to your company. `references/fields-and-responses.md` says where the API returns tax IDs.
3. Send `parent_type: "ITEM"`, `parent_id` set to the `item_id`, and `tax_id`. All three are required.
4. A `201` returns `tax_amount`, the updated `item` and its `order`. Keep `tax_amount.tax_amount_id` for later updates.

Example request, with made-up values:

```json
{
  "parent_type": "ITEM",
  "parent_id": "3f2a1b4c-0000-4000-8000-000000000010",
  "tax_id": "7c8d9e0f-0000-4000-8000-000000000020"
}
```

Example response, shortened:

```json
{
  "tax_amount": {
    "tax_amount_id": "9a8b7c6d-0000-4000-8000-000000000030",
    "parent_type": "ITEM",
    "parent_id": "3f2a1b4c-0000-4000-8000-000000000010",
    "tax_id": "7c8d9e0f-0000-4000-8000-000000000020",
    "percent": "13.0000",
    "amount": "26.00",
    "display_order": 0
  },
  "item": { "...": "..." },
  "order": { "...": "..." }
}
```

### Change the tax on an item

1. Find the `tax_amount_id` in the item's `tax_amounts` array, from `GET /item/{id}` or an earlier tax-amount response.
2. Send `PUT /tax-amount/{id}` with the new `tax_id`.
3. A `200` returns `tax_amount`, `item`, `order` and a `taxes` array.

## Gotchas

- `parent_type` accepts only `ITEM` in a request. Responses can show other values, but you cannot send them here.
- Send `percent` and `amount` only with the reserved automation tax ID. With any other `tax_id`, the spec says the call returns a `400`.
- The override needs feature access. The spec names it only for `PUT`: the `FEATURE-ADVANCED` or `FEATURE-ENTERPRISE` capability plus the `ENABLE-CONNECTED-SHOPS` user flag. Without it you get a `403`.
- `percent` comes back as a string. `amount` can be a string or a number.
- `POST` always adds another tax amount. To change one, use `PUT`.
- A `tax_id` from another company returns `403`. An unknown `tax_id` or item returns `404`.
- Use the returned `item` and `order` to refresh your totals instead of calling `GET /item/{id}` again.

## Known issues

None known.

## What the API cannot do

- **List or get taxes.** The spec has no taxes endpoint. Read tax IDs from the places listed in `references/fields-and-responses.md`.
- **Read or delete a tax amount on its own.** Read it through the item's `tax_amounts`.
- **Change `percent` or `amount` with `PUT`.** It takes only `tax_id`.
- **Tax a bill line.** `POST /bills` takes `items[].tax_amounts[].tax_id` directly.

## Reference files

| File | Read it when |
| --- | --- |
| `references/fields-and-responses.md` | You need every field, the error codes, where tax IDs come from, related order and bill calls, or what the spec leaves unconfirmed |

Last verified: 2026-11-26
