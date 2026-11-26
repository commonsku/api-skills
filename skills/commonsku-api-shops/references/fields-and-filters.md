# Shop fields, filters and errors

Read this when you need every `GET /shop` filter, the fields of a shop, what each call can set, or what an error from a shop call means. The live spec wins: check it before you send a request.

## `GET /shop` parameters

Every parameter is optional.

| Parameter | Type | What it does |
| --- | --- | --- |
| `client_id` | UUID | Shops for this client |
| `search` | string | Text search. The spec does not say which fields it matches |
| `client_rep_id` | UUID | Shops for this client rep. The `commonsku-api-users` skill covers user IDs |
| `status_id` | UUID | Shops with this status |
| `type` | `Connected`, `Pop-up`, `Marketing` or `Company` | Shops of this type |
| `is_headless` | boolean | Headless or not |
| `sort` | `shop_name`, `date_created` or `shop_number`, each with an optional `-` prefix for descending | Sort order |
| `page_limit` | integer | Page size. Default 50, maximum 250 |
| `cursor` | string | The `meta.cursor` from the previous page |

The `commonsku-api-setup` skill covers cursor paging and when to stop.

## `GET /shop` response

`{ "data": [ ... ], "meta": { "cursor", "page_count", "page_limit", "next_page" } }`. Each entry is a short view:

| Field | Type |
| --- | --- |
| `shop_id` | UUID |
| `shop_name` | string |
| `shop_number` | integer |
| `date_created` | date-time |
| `date_expiry` | date-time or null |
| `is_headless`, `aggregate` | boolean |
| `status_name` | string |
| `type` | `Connected`, `Pop-up`, `Marketing`, `Company` or null |
| `client_name` | string or null |
| `contact_first_name`, `contact_last_name` | string |
| `last_order` | date-time or null |
| `order_count`, `checkout_count` | integer |
| `order_total` | number |

The short view has `status_name` but not `status_id`, and `client_name` but not `client_id`. Call `GET /shop/{id}` for the IDs.

## The full shop object

`GET /shop/{id}`, `POST /shop`, `PUT /shop/{id}` and `POST /shops/connected` return the same object, under a different wrapper each time (see the `SKILL.md` gotchas). Besides every field `PUT` can set (below), it returns:

- `shop_id`, `tenant_id`, `shop_number`, `date_created`, `shop_type`, `order_id`
- `default_tag_id`. The `commonsku-api-tags` skill covers it
- `currency`: `currency_id`, `currency_country`, `currency_name`, `currency_symbol`
- `allow_new_contacts`, `allow_new_addresses`, `zip2tax`, `in_aggregating`
- `default_avalara_category_id`, `avalara_shipping_category_id`
- `template_data`: template data grouped by template ID and property name
- `contact_first_name`, `contact_last_name`
- `metadata`: `size`, `last_update`, `hash`

`shop_tags` comes back as an object, and `free_shop_data` as an array of objects with `domain`, although `PUT` takes both as arrays of strings.

## Fields you can send

### `POST /shop`

| Field | Type | Notes |
| --- | --- | --- |
| `shop_name` | string | Required |
| `shop_type` | `popup`, `company` or `marketing` | Connected is rejected. Use `POST /shops/connected` |
| `client_id` | UUID | Must belong to your company |
| `aggregate` | boolean | |
| `is_headless` | boolean | |
| `external_store_type` | `SHOPIFY` | Any other value is rejected |
| `public_view_template_id`, `public_view_layout_id` | UUID | |

### `POST /shops/connected`

| Field | Type | Notes |
| --- | --- | --- |
| `shop_name` | string | Required |
| `client_id` | UUID | Required |
| `auto_upgrade_orders_to_sales_order` | boolean | Turns checkouts into a sales order once payment succeeds |

### `PUT /shop/{id}`

Every field is optional. Send only what you change.

| Group | Fields |
| --- | --- |
| Basics | `shop_name`, `client_id`, `status_id`, `date_expiry` (date), `aggregate`, `shop_version`, `shop_options` |
| People | `client_rep_id`, `sales_rep_id`, `contact_name`, `contact_email`. The `commonsku-api-users` skill covers user IDs |
| Defaults | `default_event_type_id` (the `commonsku-api-event-types` skill lists them), `default_project_template_id`, `currency_id`, `default_tax_id` |
| Look and content | `public_view_layout_id`, `public_view_template_id`, `template_color`, `introduction`, `checkout_message`, `portal_image`, `product_list_style`, `show_product_warnings`, `show_tenant_on_receipt` |
| Access | `shop_password` |
| Billing | `billing_address_type`, `billing_address_id`, `add_billing_address_to_client`, `billing_contact_type`, `billing_contact_id`, `add_billing_contact_to_client` |
| Shipping | `shipping_address_type`, `shipping_address_id`, `add_shipping_address_to_client`, `shipping_contact_type`, `shipping_contact_id`, `add_shipping_contact_to_client`, `collect_ship_address` |
| Checkout | `allow_credit_card_checkout`, `per_checkout_limit`, `force_minimum_qty`, `notify_checkout_recipients`, `collect_customer_po` |
| Tax | `ignore_tax_on_shipping`, `calculate_checkout_taxes`, `tax_custom_ship_address` |
| Free shops | `is_shop_free`, `single_use_for_free_shop`, `free_shop_data` (email addresses or domains allowed to claim the free shop), `free_items_count`, `allow_multiple_free_item` |
| Tags | `shop_tags`: an array of tag labels that replaces the shop's tags |
| Chat | `drift_id`, `chat_bot_id` |
| External store | `is_headless`, `external_store_id`, `external_store_name`, `external_store_url`, `external_store_type`, `auto_upgrade_orders_to_sales_order`. `references/shop-types.md` covers their requirements |

## Errors

| Operation | Status | Cause |
| --- | --- | --- |
| `POST /shop` | `400` | `shop_name` is missing, `external_store_type` is not `SHOPIFY`, or `shop_type` is connected |
| `POST /shop` | `403` | Missing the `CREATE-SHOP` capability, or `client_id` belongs to another company |
| `POST /shop` | `404` | `client_id` does not exist |
| `GET /shop/{id}` | `403` | The shop belongs to another company |
| `GET /shop/{id}` | `404` | Shop not found |
| `PUT /shop/{id}` | `400` | The shipping address or checkout tax settings are invalid for Avalara |
| `PUT /shop/{id}` | `403` | Missing the `MODIFY-SHOP` capability; missing `FEATURE-ADVANCED` or `FEATURE-ENTERPRISE` for `is_headless` or `external_store_type`; missing the `ENABLE-CONNECTED-SHOPS` flag for `auto_upgrade_orders_to_sales_order` on a connected shop; or the shop or `client_id` belongs to another company |
| `PUT /shop/{id}` | `404` | Shop not found, or `client_id` does not exist |
| `POST /shops/connected` | `400` | Invalid input. The body is a string, or an object with a `shop_name` message |
| `POST /shops/connected` | `403` | Access denied. The body is a string, or an object with a `user_flag` message |
| `POST /shops/connected` | `404` | Not found |
| `POST /shops/connected` | `500` | The shop could not be saved. The body has a `shop_creation` message |

The `commonsku-api-setup` skill covers capabilities, and what to do when reads work but writes fail.

## Not confirmed

The spec does not say:

- What the `send_email` query parameter on `GET /shop/{id}` does. Do not set it until you know.
- Which fields `search` on `GET /shop` matches.
- What `client_rep_id` and `sales_rep_id` take on `PUT /shop/{id}`.
- Which values `billing_address_type`, `shipping_address_type`, `billing_contact_type`, `shipping_contact_type`, `product_list_style`, `notify_checkout_recipients`, `collect_customer_po` and `shop_options` accept.
- Where to find IDs for `status_id`, `currency_id`, `default_project_template_id`, `public_view_template_id` and `public_view_layout_id`. No operation lists them. Read them from an existing shop.
- Whether `POST /shop` needs a capability to set `is_headless` or `external_store_type`, as `PUT` does.
- Which capability or flag `POST /shops/connected` checks.
