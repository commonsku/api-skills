# Product imprint fields

These tables follow the live production spec. Check the spec you downloaded before you rely on them, because it wins if they differ. Both operations carry the Beta Feature label.

## Contents

- `GET /product-imprints` parameters
- `GET /product-imprints` response
- `POST /product-imprints` request body
- `POST /product-imprints` response
- Responses and errors

## `GET /product-imprints` parameters

| Parameter | In | Type | Notes |
| --- | --- | --- | --- |
| `product_id` | query | string (UUID) | Required. The product to list imprints for |

The spec lists no other filters, no paging parameters and no sort.

## `GET /product-imprints` response

The body is an object with an `imprints` array. Each imprint has:

| Field | Type | Notes |
| --- | --- | --- |
| `imprint_id` | string (UUID) | |
| `tenant_id` | string (UUID) | |
| `imprint_name` | string | |
| `imprint_description` | string | |
| `imprint_price_description` | string | |
| `imprint_price_description_default` | string | |
| `imprint_front_end_enabled` | integer | |
| `default_supplier_id` | string (UUID) | |
| `created_by` | string (UUID) | |
| `date_created` | string (date-time) | |
| `active` | integer | |
| `ext_decoration_id` | string or null | |
| `ext_artwork_id` | string or null | |
| `ext_location_id` | string or null | |

## `POST /product-imprints` request body

| Field | Type | Notes |
| --- | --- | --- |
| `product_id` | string (UUID) | Required |
| `imprint_id` | string (UUID) or null | Send this or `imprint_name` |
| `imprint_name` | string or null | Used when `imprint_id` is not sent. Creates the imprint if it does not exist |

## `POST /product-imprints` response

| Field | Type | Notes |
| --- | --- | --- |
| `META.primary_key` | string | `imprint_id` |
| `imprint_id` | string (UUID) | |
| `exists` | boolean or null | Present and `true` when the product already had this imprint |
| `productimprint.product_id` | string (UUID) | The product |
| `productimprint.imprint_id` | string (UUID) | The imprint |
| `productimprint.imprint_name` | string or null | Present when you sent `imprint_name` |

## Responses and errors

| Operation | Code | Meaning |
| --- | --- | --- |
| `GET /product-imprints` | `200` | List of the product's imprints |
| `GET /product-imprints` | `404` | `product_id` is missing |
| `POST /product-imprints` | `200` | Imprint added to the product |
| `POST /product-imprints` | `404` | `product_id` is missing, or neither `imprint_id` nor `imprint_name` was sent |
