---
name: commonsku-api-product-costs
description: "Read and set a commonsku product's net cost tiers through the API: up to six quantity breaks, optionally per size and colour. Use this when someone reads or syncs what a distributor pays a supplier for a product, sets quantity breaks, or asks whether the API can set a product's selling price."
---

# Product costs

## What this skill helps you do

This skill covers a product's net costs in commonsku: what the distributor pays the supplier. A cost row holds up to six quantity breaks, for the whole product or for one size and colour. Product costs hold no selling price. For the product itself, use the `commonsku-api-products` skill.

Check each operation's current fields in the spec you downloaded. This skill does not repeat them.

## Steps

### Read a product's costs

1. Look up `GET /product-costs` in the spec.
2. Call it with `product_id` and `currency_id`, such as `USD`. Both are required.
3. To narrow the result, add `size_id` or `color_id`. To get every size and colour, pass `include_all=true`.
4. To fall back to another currency when there are no costs in the first one, add `alternate_currency_id`.

For products managed in commonsku, you get your company's costs, falling back to catalogue costs.

### Set a product's costs

Call `POST /product-costs` with `product_id`, `currency_id`, the size and colour (a UUID, `"TBD"` or `null`), and `quantity1` to `quantity6` and `cost1` to `cost6`. If no row exists for that product, size and colour, one is created. If one exists, it is updated.

Example request, with made-up values:

```sh
curl -sS -X POST "$COMMONSKU_BASE_URL/product-costs" \
  -H "x-api-key: <your-api-key>" \
  -H "Content-Type: application/json" \
  -d '{ "product_id": "6c1f8e42-0000-4000-8000-000000000001", "currency_id": "USD", "size_id": null, "color_id": null,
        "quantity1": "100", "cost1": "4.25", "quantity2": "250", "cost2": "3.90" }'
```

The response is `200` with the saved cost row. Send blank quantity and cost fields only when you mean to delete the row.

### Delete a cost row

Call `POST /product-costs` for that product, size and colour, with every quantity and cost field empty. The response has no `product_cost`.

## Gotchas

- New rows are stored in the supplier's default currency, whatever `currency_id` you send. On `POST`, `currency_id` only finds an existing row.
- Sending every quantity and cost field empty deletes an existing row. A sync must not send blanks to skip a product.
- Quantities and costs are strings, as in `"12.50"`, not numbers.
- "Product not found" is `400` on `GET` and `404` on `POST`. `POST` also returns `400` when the product's division is not found.
- For ESP and other external products, `GET` returns the provider's cost data in a different shape.
- Both operations carry the spec's Beta Feature label.

## Known issues

None known.

## What the API cannot do

- **Set a selling price.** Product costs hold net costs only.
- **Explain everything.** The spec does not say whether you can set costs on external products, why the stored currency ignores the one you send, what `"TBD"` does for a size or colour, or whether "empty" means `""`, `null` or leaving the fields out.

Last verified: 2026-11-26
