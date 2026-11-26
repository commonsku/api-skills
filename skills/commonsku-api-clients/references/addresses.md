# Client addresses

The detail for the client address operations in the `commonsku-api-clients` skill.

## Read addresses

- One client: `GET /clients/{id}/addresses`.
- Every client, changed since your last run: `GET /addresses/client?date_updated__gt=<time>&sort=date_updated`. Add `expand[]=client` to include the client in each row.

Both are paged. Set `page_limit` and follow the cursor, as the `commonsku-api-setup` skill describes.

## Create an address

Look up `POST /clients/{id}/addresses` and send the required fields: `address_name`, `address_company`, `address_line_1`, `address_city`, `address_state`, `address_postal`, `address_country` and `address_type`.

On create and update, do not send `address_id` or `client_id`. They are read-only.

```json
{
  "address_name": "Head office",
  "address_company": "Example Client Inc.",
  "address_line_1": "100 Example Street",
  "address_city": "Toronto",
  "address_state": "ON",
  "address_postal": "M1M 1M1",
  "address_country": "Canada",
  "address_type": "BOTH"
}
```

## Update or deactivate an address

- `PUT /addresses/{id}`: send the whole address, with every required field, not only the fields that changed.
- `DELETE /addresses/{id}`: deactivates the address.
