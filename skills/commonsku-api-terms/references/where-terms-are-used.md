# Where other calls use terms

Read this when a call takes or returns payment terms and you need to know which form to use. The live spec wins: check the field before you send a request. Each object's own skill covers the rest of that call.

## Term fields

| Field | Type | Notes |
| --- | --- | --- |
| `terms_id` | string, UUID | Always returned by `GET /terms` |
| `terms_name` | string | Always returned by `GET /terms` |

## Fields you can set

| Operation | Field | Form |
| --- | --- | --- |
| `POST /clients` | `terms` | The exact `terms_name` from `GET /terms`. Defaults to the default term in the admin settings |
| `PUT /clients/{id}` | `terms` | The exact `terms_name` from `GET /terms` |
| `PUT /sales_orders/{id}` | `terms_id` | UUID |
| `PUT /account/{id}` | `default_terms_id` | UUID. The `commonsku-api-accounts` skill covers the rest |

## Fields that are only returned

| Operations | Field |
| --- | --- |
| `GET /clients`, `GET /clients/{id}`, `DELETE /clients/{id}` and the client create and update responses | `terms`, the `terms_name` |
| `GET /sales_orders`, `GET /sales_orders/{id}`, `PUT /sales_orders/{id}` | `terms`, the `terms_name` |
| `GET /orders`, `GET /orders/{id}`, `POST /orders/upgrade` | `terms`, the `terms_name` |
| `GET /projects`, `POST /projects`, `GET /projects/{id}`, `PUT /projects/{id}` | `terms`, the `terms_name` |
| `GET /invoices`, `GET /invoices/{id}`, `PUT /invoices/{id}` | `terms`, string |
| `GET /bills`, `GET /bills/{id}`, `POST /bills` | `terms`, string |
| Responses that embed an order, such as item and breakdown updates | `order.terms_id`, UUID |
| `GET /account/{id}`, `POST /account`, `PUT /account/{id}` | `terms_id`, `terms_name`, `payment_terms_id`, `default_terms_id` |

## Not confirmed

The spec does not say these things. Test in a sandbox before you rely on them:

- Whether matching a client's `terms` against `terms_name` is case-sensitive.
- What the API returns when a client's `terms` or a sales order's `terms_id` is not on the distributor's list.
- Whether the `terms` string returned on invoices and bills is the `terms_name`.
- How `terms_id`, `payment_terms_id` and `default_terms_id` differ on an account.
