# Tags on other objects

Read this when you apply tags to a record, or read the tags another call returns. `GET /tags`, `POST /tags` and `PUT /tags/{id}` only manage the tag list. Each record's own call applies tags to it. The live spec wins: check each field before you send a request, because the form differs from call to call. Each object's own skill covers the rest of that call.

## Fields you can set

| Operation | Field | Form in the spec |
| --- | --- | --- |
| `POST /clients`, `PUT /clients/{id}` | `tags` | Array of tag objects. Each needs `label` and can have `description`. `tag_id` is read-only. The `commonsku-api-clients` skill covers the rest |
| `PUT /products/{id}` | `product_tags` | String. The spec does not describe its format |
| `PUT /account/{id}` | `account_tags` | String or null. The spec does not describe its format. The `commonsku-api-accounts` skill covers the rest |
| `PUT /shop/{id}` | `shop_tags` | Array of tag labels. Replaces the shop's current tags |

## Filter by tag

- `GET /products` takes `tag_id` to return only products with that tag.

## Fields that are only returned

| Operations | Field |
| --- | --- |
| `GET /clients`, `GET /clients/{id}`, `DELETE /clients/{id}` and the client create and update responses | `tags` |
| `GET /item/{id}`, `PUT /item/{id}`, `PUT /item-decorations/{id}`, `POST /products` | `product_tags` |
| `GET /account/{id}`, `PUT /account/{id}`, `POST /account` | `account_tags`, `client_tags`, `supplier_tags`, `tenant_tags`, `tags` |
| `GET /company/get` | `tags`, `account_tags` |
| `POST /shop`, `GET /shop/{id}`, `PUT /shop/{id}`, `POST /shops/connected` | `shop_tags`, `default_tag_id` |
| `GET /orders`, `GET /orders/{id}`, `POST /orders/upgrade` | `tags` |
| `GET /sales_orders`, `GET /sales_orders/{id}`, `PUT /sales_orders/{id}` | `tags` |
| `GET /invoices`, `GET /invoices/{id}`, `PUT /invoices/{id}` | `tags` |
| `GET /projects/{id}/overview` | `tags` on each feed entry |

`ORDER` is a `resource_type` on `GET /tags`, but no order, sales order or invoice call takes tags.

## Tagging users on a note

`POST /projects/{id}/notes` takes `tagged_users`. Those are commonsku users, not tags from `GET /tags`. The `commonsku-api-users` skill says which value to send.

## Not fully documented

The live spec does not cover these. Where noted, a fact comes from engineering confirmation.

- Whether a record call creates a missing tag from a new label. For clients, a new `tags[].label` can create the tag. For the other calls it is not documented. When you need a known `tag_id`, the safer default is to create the tag with `POST /tags` first.
- Whether `tags` on `PUT /clients/{id}`, `product_tags` or `account_tags` replaces the record's tags or adds to them.
- The format of the `product_tags` and `account_tags` strings.
- How to tag a company contact through the API. `COMPANY-CONTACT` is a `resource_type`, but no contact call in the spec takes tags.
