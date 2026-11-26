---
name: commonsku-api-accounts
description: "Read, create and update commonsku account profiles for clients, suppliers and the distributor: account number, website, default terms, currency and tax, sales rep, status and tags. Use this when someone calls /account, sets a supplier account number or client defaults, or a profile update is rejected."
---

# Account profiles

## What this skill helps you do

The `/account` operations read and change the profile behind a client, a supplier or the distributor's own account. New to the commonsku API? Start with the `commonsku-api-setup` skill.

## Steps

### Read a profile

1. Look up `GET /account/{id}` in the spec you downloaded.
2. Pass `company_type` as `CLIENT`, `SUPPLIER` or `DISTRIBUTOR`. For a supplier, the ID is the supplier account ID.
3. For a client, `simple=true` leaves out addresses, child accounts and contact department groupings. Suppliers and the distributor always return the full profile.

### Update a profile

1. Look up `PUT /account/{id}`. Its accepted fields depend on `company_type`.
2. Send `company_type` in the query and in the body, plus only the fields you are changing.

Set a supplier account number (made-up values):

```sh
curl -sS -X PUT "$COMMONSKU_BASE_URL/account/<supplier-account-id>?company_type=SUPPLIER" \
  -H "x-api-key: <your-api-key>" \
  -H "Content-Type: application/json" \
  -d '{"company_type":"SUPPLIER","account_number":"10045"}'
```

### Create an account

Send `POST /account` with `company_type` and `company_name`. A client can also take `industry_id` and `sales_rep_id`. A supplier also needs a `division_id`, a `company_id` or a company name.

## Gotchas

- `company_type` decides which fields are valid.
- Numbers such as `sales_target`, `account_order_margin_minimum` and `artwork_hourly_rate` are sent as strings, to keep their precision.
- To clear a client's `parent_account_id` or a supplier's `sales_rep_id`, send an empty string or `"none"`.
- `account_profile` has a maximum length measured in bytes, not characters.
- A supplier profile can hold website and PromoStandards passwords. Treat responses as sensitive and never log them.
- A `403` means the commonsku user behind your API key cannot change that type of account. See the `commonsku-api-support` skill.

## Known issues

None known.

## What the API cannot do

- **Delete an account.** There is no delete operation for `/account`.

Last verified: 2026-11-26
