---
name: commonsku-api-clients
description: "Create, find, update and delete commonsku clients and their contacts and addresses. Use this when someone imports or syncs a client list, matches clients to another system by customer_number, manages client contacts or addresses, or a client create or update is rejected."
---

# Clients

## What this skill helps you do

This skill covers commonsku clients and their contacts and addresses. New to the commonsku API? Start with the `commonsku-api-setup` skill.

## Steps

### Create a client

Look up `POST /clients` in the spec you downloaded.

- Required: `client_name`, `terms` (see the `commonsku-api-terms` skill) and `industry_name` (see the `commonsku-api-industries` skill).
- Optional: `client_rep_login_name` (an active user's email address; see the `commonsku-api-users` skill), `customer_number`, `client_website`, `credit_hold`, `client_status` and `tags`.
- Do not send `client_id`, `primary_contact_email` or `client_portal_url`. They are read-only.

### Find a client

`GET /clients` filters on `customer_number` and `client_name` (exact match), `credit_hold` and `date_updated`. Store the other system's ID in `customer_number`, and use it to find the client again.

### Update a client

`PUT /clients/{id}`: send `client_name`, `terms` and `industry_name` every time, not only the fields that changed.

### Delete a client

`DELETE /clients/{id}` deletes the client and all its contacts and addresses. Confirm with the user before you call it.

### Contacts and addresses

A client's contacts and addresses each have list, create, update and deactivate operations. Before you call one, read its reference file, listed below.

## Gotchas

- The spec says `credit_hold` defaults to `false` when it is not provided. Send its current value on every client update, so a hold is not cleared by accident.
- Put one email address per contact. Two addresses in one field can stop that contact logging in to the client portal.
- A `403` on a create or update means the commonsku user behind the API key cannot make that change. See the `commonsku-api-support` skill.
- For a one-time import of a large client list, ask commonsku support about the client import. Through the API, each client, contact and address is a separate call against the daily limit (see the `commonsku-api-setup` skill).
- Send `address_type` as `SHIPPING`, `BILLING` or `BOTH`. Responses can show `Shipping`, `Billing` or `Both`, so compare without regard to case.
- `address_postal` holds the postal code or ZIP code.
- `DELETE` deactivates an address. It does not erase it.
- To set an order's addresses, use `PUT /sales_orders/{id}` or `POST /projects`, not these operations.

## Known issues

None known.

## What the API cannot do

- **Set a contact's phone numbers, or reactivate a contact.** `phones` and `active` are read-only.
- **Set a client's primary contact email directly.** `primary_contact_email` is read-only on the client.
- **Create a supplier or distributor address.** Only `POST /clients/{id}/addresses` creates an address.

## Reference files

| File | Read it when |
| --- | --- |
| `references/contacts.md` | The task reads or writes client contacts |
| `references/addresses.md` | The task reads or writes client addresses |

Last verified: 2026-11-26
