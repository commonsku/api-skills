# Shop types and connected shops

Read this when you choose which kind of shop to create, or connect a shop to an external store. The live spec wins: check it before you send a request.

## Types

`GET /shop` reports each shop's `type` as `Connected`, `Pop-up`, `Marketing` or `Company`. Use those values for the `type` filter.

## Which call creates which type

| Type | Create with | `shop_type` value |
| --- | --- | --- |
| Pop-up | `POST /shop` | `popup` |
| Company | `POST /shop` | `company` |
| Marketing | `POST /shop` | `marketing` |
| Connected | `POST /shops/connected` | None. The endpoint makes a connected shop |

`POST /shop` returns a `400` when `shop_type` is connected. All four types are updated with `PUT /shop/{id}` and read with `GET /shop/{id}`.

## Connected shops

A connected shop takes orders from a storefront commonsku does not host. Orders arrive as checkouts through `POST /shop/{id}/connected-checkout`. The `commonsku-api-checkouts` skill covers submitting and reading checkouts, mapping products and the access a connected shop needs.

- `POST /shops/connected` requires `shop_name` and `client_id`.
- `auto_upgrade_orders_to_sales_order` turns a checkout into a sales order once payment succeeds. Set it on `POST /shops/connected`, or later on `PUT /shop/{id}`.

## External store fields

| Field | Where | Notes |
| --- | --- | --- |
| `external_store_type` | `POST /shop`, `PUT /shop/{id}` | On `POST /shop` the only accepted value is `SHOPIFY`. On `PUT` it is a string naming the external system. Setting it on `PUT` needs `FEATURE-ADVANCED` or `FEATURE-ENTERPRISE` |
| `is_headless` | `POST /shop`, `PUT /shop/{id}` | Setting it on `PUT` needs `FEATURE-ADVANCED` or `FEATURE-ENTERPRISE` |
| `external_store_id`, `external_store_name`, `external_store_url` | `PUT /shop/{id}` | |
| `auto_upgrade_orders_to_sales_order` | `POST /shops/connected`, `PUT /shop/{id}` | On `PUT` it is meant for shops whose `external_store_type` is `CONNECTED`, the generic API integration type, as opposed to a Shopify-connected shop. It needs the `ENABLE-CONNECTED-SHOPS` flag there: a connected shop returns a `403` without it, and other shops drop the field silently |

## Not confirmed

The spec does not say:

- What `is_headless` changes about a shop.
- Whether a shop's type can be changed after it is created.
- How a Shopify-connected shop relates to the `Connected` type in `GET /shop`.
