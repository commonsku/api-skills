---
name: commonsku-api-product-imprints
description: "List the imprint methods on a commonsku product and add an imprint method to a product. Use this when someone calls /product-imprints, sets up how a product can be decorated, or needs the imprints attached to a product. Both operations carry the Beta Feature label."
---

# Product imprints

## What this skill helps you do

A product imprint links an imprint method, such as screen print or embroidery, to a product. You can list a product's imprints and add an imprint to a product. Both operations carry the Beta Feature label in the spec. New to the commonsku API? Start with the `commonsku-api-setup` skill.

To list, create or rename the imprint methods themselves, use the `commonsku-api-imprints` skill. To find a `product_id`, use the `commonsku-api-products` skill.

## Steps

### List a product's imprints

Send `GET /product-imprints?product_id=<product-uuid>`. `product_id` is required. Check the operation's current schema in the spec you downloaded.

The response is an `imprints` array of imprint objects, with the same fields as an imprint from `GET /imprints`:

```json
{ "imprints": [ { "imprint_id": "<imprint-uuid>", "imprint_name": "Example Screen Print", "active": 1 } ] }
```

### Add an imprint to a product

1. Call `GET /imprints`, as the `commonsku-api-imprints` skill describes, and find the imprint's `imprint_id`.
2. Send `POST /product-imprints` with `product_id`, plus `imprint_id` or `imprint_name`. Check the operation's current schema in the spec you downloaded.

```json
{ "product_id": "<product-uuid>", "imprint_id": "<imprint-uuid>" }
```

The response returns `imprint_id` and a `productimprint` object with `product_id`, `imprint_id` and, when you sent it, `imprint_name`. When the product already had that imprint, the response also carries `exists: true`.

## Gotchas

- You must give a product. No operation lists product imprints across products.
- The list returns only imprints that belong to your company.
- If you send an `imprint_name` that matches no imprint, commonsku creates a new imprint. Send an existing `imprint_id` where you can, so you do not create near-duplicates, such as "Screen Print" and "Screenprint".
- Not fully documented: the spec does not say which `default_supplier_id` an imprint created this way gets, or whether `imprint_name` matching ignores case.
- A missing `product_id` returns `404`, not `400`, on both operations. On `POST`, a body with neither `imprint_id` nor `imprint_name` also returns `404`.
- Not confirmed: which capability, if any, the API key's user needs.

## Known issues

None known.

## What the API cannot do

- **Remove an imprint from a product.** No operation deletes a product imprint.
- **Change a product's imprint.** No operation updates a product imprint. To rename or edit the imprint method itself, use `PUT /imprints/{id}` in the `commonsku-api-imprints` skill.
- **List all product imprints.** Every list call needs a `product_id`.

## Reference files

| File | Read it when |
| --- | --- |
| `references/fields.md` | You need every parameter, request field, response field or error code |
| `references/sandbox-differences.md` | You work against a sandbox |

Last verified: 2026-11-26
