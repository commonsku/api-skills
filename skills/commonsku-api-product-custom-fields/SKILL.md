---
name: commonsku-api-product-custom-fields
description: "Create and rename your company's custom fields for commonsku products, and set or read a product's value for each field. Use this when someone calls /product-custom-fields or /product-custom-field-values, stores extra product data such as an internal code, or can't find a custom field's ID."
---

# Product custom fields

## What this skill helps you do

A product custom field is a labelled field your company adds to products. You define the field once, then store a value for it on each product. New to the commonsku API? Start with the `commonsku-api-setup` skill.

## Steps

### Create a field

Send `POST /product-custom-fields` with `label` (up to 50 characters) and `category`: `OVERVIEW`, `DETAILS_IMPRINT`, `DETAILS_PACKAGING` or `DETAILS_MISC`. The category decides where the field appears on the product. commonsku sets the field's owner from your API key's account, so don't send `parent_type` or `parent_id`.

```json
{ "label": "Internal code", "category": "DETAILS_MISC" }
```

Save `custom_field_id` from the `201` response.

### Rename a field

Send `PUT /product-custom-fields/{id}` with `label`. The label is the only thing you can change.

### Set a product's value

Send `POST /product-custom-field-values` with `custom_field_id`, `product_id` and `custom_field_value` (up to 255 characters). Save `custom_field_value_id` from the `201` response.

### Read values

- One value by ID: `GET /product-custom-field-values/{id}`.
- One field on one product: `GET /product-custom-field-values?product_id=<id>&custom_field_id=<id>`. Both are required.

## Gotchas

- No operation lists your custom fields. Store each `custom_field_id` when you create the field: you need it to set or read values.
- You can't read all of a product's custom values in one call. Each read needs the product and the field.
- Names change between write and read. You send `custom_field_value` and `category`; the list returns `value` and `category_id`. `GET /product-custom-field-values/{id}` returns `custom_field_value`.
- Values are strings.
- You can read only values whose field belongs to your company.
- A `404` when setting a value means the custom field wasn't found. Check the `custom_field_id`.

## Known issues

None known.

## What the API cannot do

- List the custom fields defined for your account.
- Change a field's category, or delete a field.
- Change or delete a value once it's set.
- Add custom fields to order items. Only products have them.

Last verified: 2026-11-26
