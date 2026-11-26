---
name: commonsku-api-companies
description: "Find clients and suppliers by name and read a company profile with /company and /company/get. Use this when someone needs a client's or supplier's commonsku IDs, wants a client's addresses, contacts and shipping accounts in one call, or a company lookup returns 400 or 404."
---

# Companies

## What this skill helps you do

`/company` searches clients and suppliers, and `/company/get` returns one company's profile. To create or change clients, use the `commonsku-api-clients` skill; for account settings, the `commonsku-api-accounts` skill. New to the commonsku API? Start with the `commonsku-api-setup` skill.

## Steps

### Search

1. Send `GET /company` with `company_type` set to `CLIENT` or `SUPPLIER`.
2. Narrow it with `company_name`. `search` is a flag, not a search term: when present, it runs a name search.
3. Page with `start-index` (from 0) and `max-results` (default 50), and sort with `order-by` and `order-dir`.

Client results include `client_id`, `client_name`, `account_status_id` and `primary_contact_id`. Supplier results include `division_id`, `division_name`, `parent_name`, `website` and `on_csku`.

### Read one profile

Send `GET /company/get` with `id` and `company_type` (`CLIENT`, `SUPPLIER` or `TENANT`). A client profile includes addresses, phones, contacts, shipping accounts, tags and the parent account. A supplier profile includes `supplier_id`, `supplier_name` and `feature_level`.

## Gotchas

- Paging here uses `start-index` and `max-results`, not the cursor most other operations use.
- `company_type=DISTRIBUTOR` returns `400` on `GET /company`.
- Client records belong to your account. Another account's client ID returns `404`. Supplier profiles can be read by any account.
- On `GET /company/get`, a company that isn't found returns `400`, and a missing `id` or `company_type` returns `404`. That's the reverse of most operations.
- Suppliers are listed by division. One supplier can appear several times, with the supplier's name in `parent_name`.

## Known issues

None known.

## What the API cannot do

- Create or update a company through these operations.

Last verified: 2026-11-26
