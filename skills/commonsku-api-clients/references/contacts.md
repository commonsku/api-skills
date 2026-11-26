# Client contacts

The detail for the client contact operations in the `commonsku-api-clients` skill.

## Read contacts

- One client: `GET /clients/{id}/contacts`.
- Every client, changed since your last run: `GET /contacts/client?date_updated__gt=<time>&sort=date_updated`. Add `expand[]=client` to include the client in each row.

Both are paged. Set `page_limit` and follow the cursor, as the `commonsku-api-setup` skill describes.

## Create or update a contact

Look up `POST /clients/{id}/contacts` or `PUT /contacts/{id}` and send the required fields: `contact_first_name`, `contact_last_name`, `contact_email` and `department_name`.

`department_name` must be one of the values the spec lists. On 2026-11-26 they were `Marketing`, `Other`, `Sales`, `Accounting`, `Human Resources`, `Executive`, `Shop`, `Purchasing`, `Unknown` and `Administration`.

Do not send `client_id`, `contact_id`, `phones` or `active`. They are read-only.

## Deactivate a contact

`DELETE /contacts/{id}` deactivates the contact.
