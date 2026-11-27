---
name: commonsku-api-product-colors
description: "List the colours on a commonsku product, add a colour to a product, and set the image for a product colour through the API. Use this when someone reads or syncs a product's colours, adds a colour by ID or name, sets a colour image, or asks whether the API can remove a colour."
---

# Product colours

## What this skill helps you do

This skill covers a product's colours in commonsku. You can list a product's colours, add a colour, and set the image for a product and colour. You cannot remove a colour. For the product itself, use the `commonsku-api-products` skill.

Check each operation's current fields in the spec you downloaded. This skill does not repeat them.

## Steps

### List a product's colours

Call `GET /product-colors` with `product_id`. You get the shared colours plus your company's own colours.

### Add a colour

1. List the product's colours first, so you do not add one twice.
2. If you have the colour's `color_id`, call `POST /product-colors` with `product_id` and `color_id`.
3. Otherwise, send `color_name` instead of `color_id`. If no colour matches the name, commonsku creates one for your company only.

Example request, with made-up values:

```sh
curl -sS -X POST "$COMMONSKU_BASE_URL/product-colors" \
  -H "x-api-key: <your-api-key>" \
  -H "Content-Type: application/json" \
  -d '{ "product_id": "6c1f8e42-0000-4000-8000-000000000001", "color_id": "9b2d7a15-0000-4000-8000-000000000002" }'
```

The response is `200` with the `color_id` used.

### Set a colour's image

Call `PUT /product-colors` with `product_id`, `color_id` and `image_id`. All three are required. If the product and colour are not linked yet, the call links them.

## Gotchas

- The spec does not say how `color_name` is matched, such as by case or spacing. Adding by name can create a duplicate colour. Send `color_id` whenever you can.
- A colour created by name is visible only to your company. Shared colours have an empty `tenant_id`. Your company's colours have your company's ID.
- `color_id` can be `"TBD"`, meaning the colour has not been chosen yet.
- A `404` can mean a missing field, not a missing record: `product_id`, `color_id`, or both `color_id` and `color_name` on `POST`. Only a missing `image_id` on `PUT` returns `400`.
- All three operations carry the spec's Beta Feature label.

## Known issues

None known.

## What the API cannot do

- **Remove a colour from a product.** There is no `DELETE` operation for product colours.
- **Upload an image.** `PUT` takes an existing `image_id`. The spec does not say where that ID comes from.

Last verified: 2026-11-26
