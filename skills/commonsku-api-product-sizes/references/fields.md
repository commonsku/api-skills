# Product size fields

These tables follow the live production spec. Check the spec you downloaded before you rely on them, because it wins if they differ. Both operations carry the Beta Feature label.

## Contents

- `GET /product-sizes` parameters
- `GET /product-sizes` response
- `POST /product-sizes` request body
- `POST /product-sizes` response
- Responses and errors

## `GET /product-sizes` parameters

| Parameter | In | Type | Notes |
| --- | --- | --- | --- |
| `product_id` | query | string (UUID) | Required. The product to list sizes for |

The operation has no other filters, no paging parameters and no sort.

## `GET /product-sizes` response

The body is an object with a `sizes` array. Each size has:

| Field | Type | Notes |
| --- | --- | --- |
| `size_id` | string | A UUID, or `"TBD"` |
| `size_name` | string | The size's name |
| `tenant_id` | string | Empty for a global size; your company ID for your company's sizes |
| `created_by` | string (UUID) or null | Not fully documented |
| `active` | integer | `0` or `1`. Read only |

## `POST /product-sizes` request body

| Field | Type | Notes |
| --- | --- | --- |
| `product_id` | string (UUID) | Required |
| `size_id` | string (UUID), `"TBD"` or null | Send this or `size_name` |
| `size_name` | string or null | Used when `size_id` is not sent. Creates the size for your company if it does not exist |

## `POST /product-sizes` response

| Field | Type | Notes |
| --- | --- | --- |
| `META.primary_key` | string | `size_id` |
| `size_id` | string | A UUID, or `"TBD"` |
| `productsize.product_id` | string (UUID) | The product |
| `productsize.size_id` | string | A UUID, or `"TBD"` |
| `productsize.size_name` | string or null | Present when you sent `size_name` |

## Responses and errors

| Operation | Code | Meaning |
| --- | --- | --- |
| `GET /product-sizes` | `200` | List of the product's sizes |
| `GET /product-sizes` | `400` | `product_id` is missing |
| `POST /product-sizes` | `200` | Size added to the product |
| `POST /product-sizes` | `404` | `product_id` is missing, or neither `size_id` nor `size_name` was sent |
