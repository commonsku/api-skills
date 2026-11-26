---
name: commonsku-api-product-sizes
description: "List the sizes on a commonsku product and add a size to a product. Use this when someone calls /product-sizes, sets up the sizes a product comes in, or needs a size_id for a product. Both operations carry the Beta Feature label."
---

# Product sizes

## What this skill helps you do

A product size is one size a product comes in, such as S, M or L. You can list a product's sizes and add a size to a product. Both operations carry the Beta Feature label in the spec. New to the commonsku API? Start with the `commonsku-api-setup` skill.

## Steps

### List a product's sizes

Send `GET /product-sizes?product_id=<product-uuid>`. `product_id` is required. Check the operation's current schema in the spec you downloaded.

The response is a `sizes` array. Each size has `size_id`, `size_name`, `tenant_id`, `created_by` and `active` (`0` or `1`):

```json
{ "sizes": [ { "size_id": "<size-uuid>", "size_name": "M", "tenant_id": "", "created_by": null, "active": 1 } ] }
```

### Add a size to a product

Send `POST /product-sizes` with `product_id`, plus `size_id` or `size_name`. Check the operation's current schema in the spec you downloaded.

```json
{ "product_id": "<product-uuid>", "size_name": "XL" }
```

The response returns `size_id` and a `productsize` object with `product_id`, `size_id` and, when you sent it, `size_name`. Save the `size_id`.

## Gotchas

- You must give a product. No operation lists sizes on their own.
- The list returns global sizes and your company's sizes only. A global size has an empty `tenant_id`; your company's sizes carry your company ID.
- If you send a `size_name` that does not exist for your company, commonsku creates a new size for your company. Check the product's sizes first and reuse an existing `size_id` so you do not create near-duplicates, such as "XL" and "X-Large".
- On `POST /product-sizes`, a missing `product_id`, or a body with neither `size_id` nor `size_name`, returns `404`, not `400`. On `GET /product-sizes`, a missing `product_id` returns `400`.
- `size_id` can be `"TBD"`. Not fully documented: the spec does not say what TBD means for a product size.
- Not confirmed: whether adding the same size twice creates a duplicate, and whether `size_name` matching ignores case or reuses a global size.
- Order-item quantities by size are breakdowns. See the `commonsku-api-breakdowns` skill.

## Known issues

None known.

## What the API cannot do

- **Remove a size from a product.** No operation deletes a product size.
- **Rename or deactivate a size.** No operation updates a size. `active` is returned but cannot be set.
- **List all sizes.** Every list call needs a `product_id`.

## Reference files

| File | Read it when |
| --- | --- |
| `references/fields.md` | You need every parameter, request field, response field or error code |
| `references/sandbox-differences.md` | You work against a sandbox |

Last verified: 2026-11-26
