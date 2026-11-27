---
name: commonsku-api-products
description: "List, get, create, copy and update commonsku products, and search product sources by SKU, through the API. Use this when someone calls /products, /products/{id} or /product/search, looks up a product_id, copies a global product into their company, or edits a product's details. All five operations carry the Beta Feature label."
---

# Products

## What this skill helps you do

This skill covers listing, getting, creating, copying and updating commonsku products, and searching product sources by SKU. All five operations carry the Beta Feature label in the spec. New to the commonsku API? Start with the `commonsku-api-setup` skill.

## Steps

Check each operation's current schema in the spec you downloaded before you call it.

### Find a product

- By name, supplier code or tag in your products: `GET /products?keyword_sku=<text>&max-results=25&start-index=0`. To find one product, stop paging once it appears; do not walk the whole list. Paging and other filters are in `references/list-filters-and-modes.md`.
- By SKU across product sources: `GET /product/search?product_sku=<sku>`. See `references/search-vs-list.md`.

### Get one product

Send `GET /products/{id}`. The response has `product`, plus `account_details` and `productError` for products from outside sources.

### Copy a global product into your company

```json
{ "target_type": "COPY", "target_id": "<global-product-uuid>" }
```

Send it to `POST /products`. A copy returns `201` with `product`. Save its `product_id`.

### Create a product

Send `POST /products` without `target_type`. The create body has many required fields, listed in `references/fields.md`. The response has a `products` array and `product`, the first product created.

### Update a product

Send `PUT /products/{id}` with only the fields to change. Every field is optional:

```json
{ "product_name": "Example Tumbler 20 oz", "active": true }
```

## Gotchas

- `POST /products` has three body shapes. Omit `target_type` to create, send `COPY` to copy a global product, or send `WARNING` with `target_id` to refresh a product's compliance warnings.
- `GET /products` changes behaviour and response shape with its mode parameters, such as `popular` or `recent`. Not documented: what happens when you send more than one mode.
- `GET /product/search` returns `204` with no body when nothing matches.
- `PUT /products/{id}` returns `403` for a Sage-sourced product, or a product your company or supplier does not own. The spec's description of `PUT /products/{id}` gives the ownership rules.
- For the format of `product_tags`, see the `commonsku-api-tags` skill.
- Not fully documented: what `copied_from` should hold when you create a new product rather than copy one.
- A product's sizes, imprints and custom fields have their own skills: `commonsku-api-product-sizes`, `commonsku-api-product-imprints` and `commonsku-api-product-custom-fields`. Look up colour and cost operations in the spec. User IDs in `sales_rep_id` and `liked_by` come from the `commonsku-api-users` skill.

## Known issues

None known.

## What the API cannot do

- **Delete a product.** No operation deletes a product. Send `"active": false` on `PUT /products/{id}` to deactivate it.
- **Change a Sage-sourced product.** The update returns `403`. Not confirmed: whether a copy of it can be changed.

## Reference files

| File | Read it when |
| --- | --- |
| `references/list-filters-and-modes.md` | You filter, sort, page or use a mode on `GET /products` |
| `references/search-vs-list.md` | You choose between `GET /product/search` and `GET /products` |
| `references/fields.md` | You need every parameter, request field, response field or error code |
| `references/sandbox-differences.md` | You work against a sandbox |

Last verified: 2026-11-26
