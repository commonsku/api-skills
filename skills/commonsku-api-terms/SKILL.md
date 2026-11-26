---
name: commonsku-api-terms
description: "List the payment terms a commonsku distributor allows and find the terms_name or terms_id another call needs. Use this when someone sets payment terms on a client or sales order through the API, needs a valid terms value, or asks whether the API can create or change terms."
---

# Payment terms

## What this skill helps you do

Payment terms, such as Net 30, say when a client must pay. The distributor sets up its list of allowed terms in the commonsku admin settings, and the list can change over time. The API can only read this list. Other calls take a term by name or by ID, so list the terms first to find a valid value.

New to the commonsku API? Start with the `commonsku-api-setup` skill. It walks you through the API key, the base URL, the live spec and paging.

## Steps

### List the terms

1. Look up `GET /terms` in the spec you downloaded. Check its current parameters and response schema.
2. Call `GET /terms` with `page_limit` set. It defaults to 20, and its maximum is 100.
3. Page with `cursor` until no further cursor comes back, as the `commonsku-api-setup` skill describes.
4. To sort, pass `sort=terms_name` or `sort=terms_id`. Add a `-` prefix for descending order, as in `sort=-terms_name`.

Example request:

```sh
curl -sS "$COMMONSKU_BASE_URL/terms?page_limit=100&sort=terms_name" \
  -H "x-api-key: <your-api-key>" \
  -H "Accept: application/json"
```

Example response, with made-up values:

```json
{
  "data": [
    { "terms_id": "5a6b7c8d-0000-4000-8000-000000000001", "terms_name": "Example Net 30" },
    { "terms_id": "5a6b7c8d-0000-4000-8000-000000000002", "terms_name": "Example Prepaid" }
  ],
  "meta": { "page_limit": 100 }
}
```

### Set terms on another object

1. List the terms and pick one.
2. Pass the value the target call asks for. Check that field in the spec: some take the name, others the ID.
   - `POST /clients` and `PUT /clients/{id}` take `terms`. Pass the exact `terms_name` from `GET /terms`. It's required when you create a client. Leaving it out returns 400.
   - `PUT /sales_orders/{id}` takes `terms_id`, the UUID from `GET /terms`.
3. `references/where-terms-are-used.md` lists every operation that takes or returns terms.

## Gotchas

- `GET /terms` returns only active terms for the distributor. Inactive terms are not included.
- Clients take `terms` as the exact `terms_name`. Sales orders take `terms_id`, a UUID. Do not send one where the other is expected.
- Reads of clients, sales orders, orders and projects return `terms` as the `terms_name`. Invoices and bills return `terms` as a string, not `terms_id`.
- The list belongs to the distributor and can change. Do not hard-code it. Page through it once, cache it for the run, and look terms up locally. Do not call it once per record.
- `GET /terms` has no filters and no get-by-ID operation.

## Known issues

None known.

## What the API cannot do

- **Create, rename or delete a term.** The API has only one terms operation, `GET /terms`. The distributor changes the list in the commonsku admin settings.
- **Set terms on a project, invoice or bill.** Those operations only return `terms`.

## Reference files

| File | Read it when |
| --- | --- |
| `references/where-terms-are-used.md` | You need to know which call takes or returns a term, and in which form |

Last verified: 2026-11-26
