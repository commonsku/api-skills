---
name: commonsku-api-orders
description: "List and read commonsku presentations, estimates and opportunities through the API, and upgrade an order to a later stage, such as a presentation to a sales order. Use this when someone lists or reads orders before the sales order stage, upgrades an order, or cannot find a sales order or invoice in /orders."
---

# Orders

## What this skill helps you do

This skill covers `/orders` in commonsku: presentations, estimates and opportunities. A Presentation is a distributor's product proposal to a client. You can list and read these orders, and upgrade an order to a later stage. Sales orders and invoices are not in `/orders`. Use `/sales_orders` and `/invoices` for them. For what each stage means, see the `commonsku-api-concepts` skill.

Check each operation's current parameters and fields in the spec you downloaded. This skill does not repeat them.

## Steps

### List orders

1. Look up `GET /orders` in the spec.
2. Pass `order_type`: `presentation`, `estimate` or `opportunity`, in either case. It is required.
3. Filter by status, customer, client or project number. For a date range, add `__lt`, `__lte`, `__gt` or `__gte` to a date filter, as in `date_updated__gte=2026-01-01T00:00:00Z`.
4. Page with `cursor` and `page_limit`, as the `commonsku-api-setup` skill describes.
5. To include each order's client, pass `expand[]=client`.

Example request:

```sh
curl -sS "$COMMONSKU_BASE_URL/orders?order_type=estimate&page_limit=50" \
  -H "x-api-key: <your-api-key>" \
  -H "Accept: application/json"
```

### Get one order

Call `GET /orders/{id}`.

### Upgrade an order

Call `POST /orders/upgrade` with `parent_id`, the order to upgrade, and `order_type`, the stage to create: `PRESENTATION`, `ESTIMATE`, `SALES ORDER` or `INVOICE`. To copy only some items, send `items`. Leave it out to copy every item.

The allowed moves, including skipped stages:

- Opportunity to Presentation, Estimate or Sales Order
- Presentation to Estimate or Sales Order
- Estimate to Sales Order
- Sales Order to Invoice

Example request, with made-up values:

```sh
curl -sS -X POST "$COMMONSKU_BASE_URL/orders/upgrade" \
  -H "x-api-key: <your-api-key>" \
  -H "Content-Type: application/json" \
  -d '{ "parent_id": "4a9d2c70-0000-4000-8000-000000000001", "order_type": "SALES ORDER" }'
```

The response is `201` with the new order. Use its ID from then on.

## Gotchas

- `GET /orders` never returns sales orders or invoices.
- The two operations spell the order type differently. The list takes lowercase or uppercase single words. The upgrade takes uppercase, with a space in `SALES ORDER`.
- `items[].quantity` on the upgrade is a string, not a number.
- The spec documents only `200` for `GET /orders` and `GET /orders/{id}`, but a `403` can still happen. See the `commonsku-api-setup` skill.

## Known issues

None known.

## What the API cannot do

- **Move an order back, or make any other move.** Only the moves listed above are allowed. Any other returns `400`.

Last verified: 2026-11-26
