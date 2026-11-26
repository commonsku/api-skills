# Listing products: filters, paging and modes

These tables follow the live production spec. Check the spec you downloaded before you rely on them, because it wins if they differ. `GET /products` carries the Beta Feature label.

## Contents

- Default list
- Filters
- Sorting and paging
- Modes
- Response shape by mode

## Default list

Without a mode parameter, `GET /products` returns a filtered, paged list of your company's products. The response has `products` and `total_products`.

## Filters

| Parameter | Type | Notes |
| --- | --- | --- |
| `keyword_sku` | string | Matches product name, supplier code or tag |
| `supplier_id` | string (UUID) | |
| `division_id` | string (UUID) | |
| `include_inactive` | boolean | Include inactive products |
| `tag_id` | string (UUID) | Products with this tag. Tag IDs come from the `commonsku-api-tags` skill |
| `owner_id` | string (UUID) | Owner, a tenant or a supplier |
| `liked_by` | string (UUID) | Products a user liked. User IDs come from the `commonsku-api-users` skill |
| `net_min` | number | Minimum net price (`cost1`) |
| `net_max` | number | Maximum net price (`cost1`) |
| `by-root` | boolean | Group results by root product |

## Sorting and paging

| Parameter | Type | Notes |
| --- | --- | --- |
| `orderby-field` | `product_name` or `date_created` | |
| `orderby-dir` | `asc` or `desc` | |
| `max-results` | integer | Page size |
| `start-index` | integer | Offset |

Paging on this operation is offset style. The `commonsku-api-setup` skill explains it. Not documented: the default and maximum page size.

## Modes

Each mode is a boolean parameter that changes what the list returns.

| Mode | Returns | Related parameters |
| --- | --- | --- |
| `popular` | Products popular by invoice history | `since` (date-time): lower bound |
| `recent` | Products recently used by a sales rep | `sales_rep_id` (UUID): defaults to the current user |
| `recently_presented` | Recently presented products | `days` (integer): lookback |
| `recently_quoted` | Recently quoted products | `days` (integer): lookback |
| `supplier_popular` | Popular products for a supplier | Not fully documented |
| `products_by_order` | Products from one order | `order_id` (UUID): required |
| `products_by_tenant` | Products from a tenant's orders | `tenant_id` (UUID): required; `presale` (boolean); `interval` (integer): lookback in days |

`purchase_order_id` (UUID) returns the products linked to a purchase order.

Not documented: what happens when you send more than one mode, and which filters each mode honours.

## Response shape by mode

The `products` array holds one of two shapes:

| Shape | Returned by |
| --- | --- |
| Full view | `popular`, `recent`, `purchase_order_id` |
| List item | The default list, `supplier_popular`, `products_by_order`, `products_by_tenant` |

Not documented: which shape `recently_presented` and `recently_quoted` return. Read `fields.md` for both shapes.
