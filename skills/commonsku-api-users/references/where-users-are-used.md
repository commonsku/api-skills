# Where other calls take a user

Read this when a call asks for a user and you need to know which value to pass. The live spec wins: check the field's description before you send a request. Each object's own skill covers the rest of that call.

## User fields

| Field | Type | Notes |
| --- | --- | --- |
| `user_id` | string | Read-only. The value for fields that take a user ID |
| `login_name` | string | Always returned. The user's email address |
| `contact_first_name` | string | Read-only |
| `contact_last_name` | string | Read-only |

## Fields that take a `login_name`

| Operation | Field |
| --- | --- |
| `POST /clients`, `PUT /clients/{id}` | `client_rep_login_name`. Must be an active commonsku user |
| `POST /projects` | `created_by`, `order_rep`, `client_rep` |
| `PUT /projects/{id}` | `order_rep`, `client_rep` |
| `POST /projects/{id}/notes` | `reminder_user`, and each entry in `tagged_users` |

- On `PUT /projects/{id}`, changing `client_rep` requires the MODIFY-CLIENT-REP capability.
- On `POST /projects/{id}/notes`, `reminder_user` defaults to the caller when `date_reminder` is set.

## Fields that take a `user_id`

| Operation | Field |
| --- | --- |
| `POST /account`, `PUT /account/{id}` | `sales_rep_id`. The `commonsku-api-accounts` skill covers the rest |
| `GET /products` | `sales_rep_id` query parameter, for recent mode. Defaults to the current user |
| `GET /products` | `liked_by` query parameter, to filter to products a user liked |
| `GET /shop` | `client_rep_id` query parameter |

## Not confirmed

The spec does not say what these take. Test in a sandbox before you rely on them:

- `client_rep_id` and `sales_rep_id` on `PUT /shop/{id}`. The `commonsku-api-shops` skill covers the rest of that call.
