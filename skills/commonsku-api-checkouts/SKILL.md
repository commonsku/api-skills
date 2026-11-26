---
name: commonsku-api-checkouts
description: "Send orders from an external storefront into a commonsku connected shop, and read a shop's checkouts and their errors. Use this when someone calls /shop/{id}/connected-checkout or /shop/{id}/checkout, a checkout comes back invalid, or a storefront order didn't become a sales order."
---

# Connected shop checkouts

## What this skill helps you do

A connected shop receives orders from a storefront commonsku doesn't host. Each order you submit becomes a checkout, and a valid checkout becomes a sales order. The API returns `403` unless the account's plan includes connected shops and connected shops are turned on. Shopify stores connect through the commonsku Connect app instead. New to the commonsku API? Start with the `commonsku-api-setup` skill.

## Steps

### Map products first

Every variant the storefront sells needs an external SKU on the matching product in the commonsku shop. Matching is not case-sensitive.

### Submit a checkout

Send `POST /shop/{id}/connected-checkout`. Each line in `breakdowns` needs `external_sku`, `quantity`, `unit_price` and `subtotal`. Optional: `customer` (`contact_name`, `contact_email`), `shipping_address`, `customer_po`, `comment`, `shipping_range_id`, `checked_out_amount`, `applied_discount`, `order_metadata` and `item_metadata`.

```json
{
  "breakdowns": [{"external_sku": "TEE-BLK-L", "quantity": 2, "unit_price": 18.50, "subtotal": 37.00}],
  "customer": {"contact_name": "Example Customer", "contact_email": "sam@example.com"},
  "customer_po": "10045",
  "checked_out_amount": 37.00
}
```

### Check the result

Don't rely on the status code alone. Check that the response includes an order, then confirm the checkout with `GET /checkout/{id}` or `GET /shop/{id}/checkout` and read `is_valid`, `order_id` and `error_message`.

### Find checkouts that stopped

Send `GET /shop/{id}/checkout?has_error=true`. Each checkout, and each line, carries an `error_message`. To read one checkout, use `GET /checkout/{id}`.

### Convert automatically

To turn valid checkouts into sales orders automatically once payment succeeds, set `auto_upgrade_orders_to_sales_order` when you create the shop with `POST /shops/connected`, or later with `PUT /shop/{id}`. On `PUT` it applies only to connected shops and is silently dropped if connected shops aren't turned on.

## Gotchas

- `unit_price` is the shop's unit price for that product, not the retail price. commonsku checks it against the shop's setup.
- A `201` can come back with `is_valid` false and no `order_id`. The line's error gives the reason, such as the price commonsku expected.
- commonsku doesn't reject a repeated checkout. Put the storefront's order number in `customer_po` or `order_metadata`, and before resending after a timeout, read `GET /shop/{id}/checkout` once and look for it. It has no paging, so don't repeat the read in a loop.
- Shipping method is a choice from the shop's shipping options (`shipping_range_id`). There are no fields for shipping cost, tax already charged or a payment reference.
- A `400` can mean the shop is closed or isn't a connected shop.

## Known issues

- **Price check expects a different price (confirmed 2026-11-26).** Price checks can expect a different price from the one set on the shop's product and mark the checkout invalid. If the expected price in the error doesn't match your shop setup, send support the checkout ID.

## What the API cannot do

- Edit or delete a checkout.
- Send shipping cost, tax charged or a payment reference with a checkout.
- Notify you when a checkout fails. Poll `GET /shop/{id}/checkout?has_error=true` on a schedule, no more often than once an hour, within the rate limits in the `commonsku-api-setup` skill. The `commonsku-api-webhooks` skill lists the webhooks that exist.

Last verified: 2026-11-26
