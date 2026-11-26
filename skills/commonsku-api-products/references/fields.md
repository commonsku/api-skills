# Product fields

These tables follow the live production spec. Check the spec you downloaded before you rely on them, because it wins if they differ. All five operations carry the Beta Feature label.

## Contents

- `GET /products/{id}` parameters and response
- Full view and list item
- `POST /products` request bodies
- `POST /products` responses
- `PUT /products/{id}` request body
- Responses and errors

`GET /products` parameters are in `list-filters-and-modes.md`. `GET /product/search` is in `search-vs-list.md`.

## `GET /products/{id}` parameters and response

| Parameter | In | Type |
| --- | --- | --- |
| `id` | path | string (UUID), required |
| `currency_id` | query | string |
| `source` | query | string |
| `division_id` | query | string (UUID) |

Not documented: what `source` and `division_id` change on this operation.

The response has:

| Field | Notes |
| --- | --- |
| `product` | The full view, below |
| `account_details` | Object or null. Its shape depends on the product's source. Empty for your own products |
| `productError` | Object with `message`, or null. Present when fetching data from an outside source failed |

## Full view and list item

The full view is returned by `GET /products/{id}`, `PUT /products/{id}` and some list modes. It has:

`product_id`, `product_name`, `product_supplier_code`, `product_description`, `root_product_id`, `tenant_id` (UUID or null), `active`, `source`, `source_name`, `division_id`, `division_name`, `supplier_id`, `currency_id`, `editable`, `default_img_id`, `product_img`, `product_image`, `colors`, `sizes`, `imprints`, `images`, `skus`, `costs`, `prices`, `bookmarked` (`0` or `1`), `product_warnings`, `stored`.

The spec types `colors`, `sizes`, `imprints`, `images`, `skus`, `prices` and `product_warnings` as arrays of objects, and `costs` as an object, without their fields. Not fully documented.

The list item is returned by the default list and some modes. It has:

`product_id`, `product_name`, `product_supplier_code`, `root_product_id`, `active`, `source`, `default_img_id`, `product_img`, `product_image`, `division_id`, `division_name`, `supplier_id`, `currency_id`, and `cost1` to `cost6`.

## `POST /products` request bodies

The body takes one of three shapes.

### Create a product (omit `target_type`)

| Field | Type | Notes |
| --- | --- | --- |
| `supplier_id` | string (UUID) | Required |
| `division_id` | UUID, or array of UUIDs | Required for supplier callers. An array creates the product in each division |
| `product_supplier_code` | string, up to 50 | Required |
| `product_name` | string, up to 100 | Required |
| `product_description` | string | Required |
| `product_url` | string | |
| `product_additional_information` | string | Required |
| `product_packaging` | string | Required |
| `product_men` | boolean, default false | Required |
| `product_women` | boolean, default false | Required |
| `product_unisex` | boolean, default false | Required |
| `product_price_description` | string | Required |
| `product_price_imprint_id` | string, up to 100 | Required |
| `product_price_display` | string, up to 100 | Required |
| `copied_from` | string | Required. The `product_id` of the source product |
| `lead_time_id` | string (UUID) | Required |
| `imprint_description` | string | |
| `imprint_area` | string | |
| `pricing` | object | Map of `division_id` to pricing data. Not fully documented |
| `colors` | array of strings | Colour names |
| `sizes` | array of strings | Size names |
| `imprints` | array of UUIDs | Imprint IDs |

### Copy a global product

| Field | Type | Notes |
| --- | --- | --- |
| `target_type` | `"COPY"` | Required |
| `target_id` | string (UUID) | Required. The global product to copy |

### Refresh compliance warnings

| Field | Type | Notes |
| --- | --- | --- |
| `target_type` | `"WARNING"` | Required |
| `target_id` | string (UUID) | Required. The product to refresh |

## `POST /products` responses

| Status | When | Body |
| --- | --- | --- |
| `200` | Create | `products`, an array of created products, and `product`, the first one |
| `200` | `WARNING` | `product_warnings` and `date_warnings_updated` |
| `201` | `COPY` | `product` |
| `400` | Missing `target_id`, or a supplier caller missing `division_id` | |

A created product in `products` returns the product's stored columns, including `product_id`, `root_product_id`, `tenant_id`, `copied_from`, `created_by`, `date_created`, `product_tags` and the fields from the `PUT` body below.

## `PUT /products/{id}` request body

Every field is optional.

| Group | Fields |
| --- | --- |
| Identity | `source`, `product_supplier_code`, `product_name`, `product_description`, `product_url`, `copied_from`, `active`, `default_img_id` (UUID), `lead_time_id` (UUID) |
| Audience and origin | `product_men`, `product_women`, `product_unisex`, `product_origin`, `product_env_friendly`, `product_recyclable` |
| Text | `product_additional_information`, `product_packaging`, `product_tags`, `product_labels`, `product_options`, `product_keywords`, `product_smart_code` |
| Price display | `product_price_description`, `product_price_imprint_id`, `product_price_display` |
| Imprint | `imprint_description`, `imprint_area`, `product_imprint_methods`, `product_imprint_colors` |
| Carton | `product_units_per_carton`, `product_weight_per_carton` (numbers) |
| Charges, each a number with a matching `_code` string | `product_setup_charge`, `product_repeat_setup_charge`, `product_art_pms_charge`, `product_art_charge_per_hour`, `product_art_charge_per_job`, `product_art_proof_charge`, `product_art_copy_change_charge`, `product_art_spec_sample_charge` |
| Policy text | `product_art_info`, `product_copy_change_info`, `product_order_change_info`, `product_coop_info`, `product_proof_info`, `product_less_min_info`, `product_overrun_info`, `product_shipping_info`, `product_warranty_info`, `product_terms_info`, `product_returns_info`, `product_other_info` |

The response is `product`, the full view.

## Responses and errors

| Operation | Status | Meaning |
| --- | --- | --- |
| `GET /products` | `403` | Access denied |
| `GET /products/{id}` | `404` | Product not found |
| `PUT /products/{id}` | `400` | Invalid JSON body |
| `PUT /products/{id}` | `403` | Sage-sourced product, or another company's or supplier's product |
| `PUT /products/{id}` | `404` | Not found |
| `GET /product/search` | `204` | No products found |
| `GET /product/search` | `422` | Validation error, with `error` |
