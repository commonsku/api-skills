---
name: commonsku-api-purchase-orders
description: "List and read commonsku purchase orders, and update a purchase order's shipping info, with /purchase_orders and /purchase_orders/{id}/shipping. Use this when someone syncs POs to a production or shipping system, looks up a PO by number, writes back ship dates, shipping cost or tracking, or gets a 403 or 404 from a purchase order call."
---

# Purchase orders

## What this skill helps you do

A purchase order (PO) is a distributor's order to a supplier. This skill covers the three purchase order operations: `GET /purchase_orders`, `GET /purchase_orders/{id}` and `PUT /purchase_orders/{id}/shipping`. The API reads POs and updates their shipping info. It cannot create or delete them.

New to the commonsku API? Start with the `commonsku-api-setup` skill.

## Steps

### List or find purchase orders

1. Check the current parameters of `GET /purchase_orders` in the spec you downloaded.
2. Call it with `page_limit` and filters. To find a PO by the number people use, filter on `po_number`. Page with `cursor` as the `commonsku-api-setup` skill describes.
3. For a sync, filter on `date_updated__gt` with the time of your last run.

```sh
curl -sS "$COMMONSKU_BASE_URL/purchase_orders?po_number=2045&page_limit=50" \
  -H "x-api-key: <your-api-key>" -H "Accept: application/json"
```

```json
{ "data": [ { "purchase_order_uuid": "7c1d2e3f-0000-4000-8000-000000000001", "purchase_order": 2045,
  "vendor": "Example Apparel Supply", "so_number": 1310, "po_in_hands_date": "2026-12-04",
  "tracking_numbers": [] } ], "meta": { "cursor": "<opaque>" } }
```

### Read one purchase order

Call `GET /purchase_orders/{id}` with the `purchase_order_uuid`. The PO comes back under `data`, with its items, breakdowns and artwork.

### Update shipping info

1. Check the request body of `PUT /purchase_orders/{id}/shipping` in the spec.
2. Send only the fields you are changing. Omitted fields are left as they are.

```json
{ "date_shipped": "2026-12-01", "shipping_cost": 42.5,
  "shipping_tracking_id": "1Z999EXAMPLE0001" }
```

The updated PO comes back under `purchase_order`.

## Gotchas

- The path `id` is the `purchase_order_uuid`. The PO number is a filter, not a path ID.
- `GET` responses wrap the PO in `data`. `PUT .../shipping` wraps it in `purchase_order`.
- You send `shipping_cost` and `shipping_overage` as numbers. They come back as strings.
- `shipping_tracking_id` is one settable string. `tracking_numbers` is a read-only list, one entry per shipment. Whether setting one changes the other is not confirmed.
- A `403` means a missing `MODIFY-PURCHASE-ORDER` capability or a tenant mismatch. See the `commonsku-api-support` skill.
- Breakdown edits do not change `date_updated`. The `commonsku-api-breakdowns` skill covers the workaround.
- To react to new or changed POs instead of polling, use the Purchase Order Created and Modified webhooks in the `commonsku-api-webhooks` skill.

## Known issues

None known.

## What the API cannot do

- **Create a purchase order.** Generate POs in commonsku. Generation fails if a decoration location has no artwork; see the `commonsku-api-artwork` skill.
- **Delete a purchase order.** No operation deletes one.
- **Change anything but shipping info.** Items, costs, supplier and status cannot be set. The shipping body also cannot set the in-hands date, ship method or ship-to address.
- **Bill a PO.** Bills are vouched against PO lines with the `commonsku-api-bills` skill.

## Reference files

| File | Read it when |
| --- | --- |
| `references/fields-and-filters.md` | You need every list filter, sort value, field, the shipping update fields or the error codes |
| `references/sandbox-differences.md` | You work against a sandbox and a field or filter is missing |

Last verified: 2026-11-26
