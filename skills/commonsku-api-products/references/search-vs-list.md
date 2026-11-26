# Product search and the product list

These notes follow the live production spec. Both operations carry the Beta Feature label. Check the spec you downloaded before you rely on them.

## Which to use

| | `GET /product/search` | `GET /products` |
| --- | --- | --- |
| Spec tag | `products` | `product` |
| Path | Singular `product` | Plural `products` |
| Looks in | Product sources, such as ESP, Sage, Distributor Central and PromoStandards | Your company's products, or a mode's results |
| Match on | `product_sku`, required | `keyword_sku`: name, supplier code or tag; and other filters |
| Paging | None in the spec | `max-results` and `start-index` |
| No match | `204`, no body | Not documented |
| Response | An object keyed by source; each value is an array of products | `products` array and `total_products` |

Use the search to find a product by SKU in outside sources. Use the list to find products already in your company.

## Search parameters

| Parameter | Notes |
| --- | --- |
| `product_sku` | Required. The SKU to search for |
| `source` | One source, or a comma-separated list |
| `division_name` | Name of the division or supplier |
| `asi_id`, `asi_number` | Sets `source` to `esp` |
| `dc_id`, `dc_number` | Sets `source` to `dc` |
| `sage_id` | Sets `source` to `sage-connect` |
| `promostandards_id` | Sets `source` to `promostandards` |
| `currency_id` | Currency to use when possible |

Not documented: the full list of `source` values, and which wins when you send `source` and a source-specific ID together.

## Search response

Each product in a source's array has `product_id`, `supplier_id`, `supplier_name`, `division_id`, `division_name`, `ps_product_id`, `product_supplier_code`, `catalog` and `skus`. Each SKU has `ext_sku_id`, `sku`, `description` and `options`, each option with `name` and `axis`.

A validation error returns `422` with `error`.

Not confirmed: whether a search result's `product_id` can be passed to `GET /products/{id}` or to `POST /products` with `COPY`.
