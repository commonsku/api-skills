---
name: commonsku-api-imprints
description: "List, create and update a company's imprint methods in commonsku through the API, such as screen print or embroidery. Use this when someone needs an imprint_id, adds or renames a decoration method, or asks whether the API can upload a file to an imprint."
---

# Imprints

## What this skill helps you do

This skill covers imprints in commonsku. An imprint is a decoration method, such as screen print or embroidery, not a file. You can list, create and update your company's imprints. This skill does not cover attaching imprints to products.

To add an artwork file to a decoration, use the `commonsku-api-files` and `commonsku-api-artwork` skills.

Check each operation's current fields in the spec you downloaded. This skill does not repeat them.

## Steps

### List imprints

1. Look up `GET /imprints` in the spec.
2. For a short list of IDs and labels, such as for a dropdown, pass `autocomplete=1`.
3. Otherwise, page with `max-results` and `start-index`, as the `commonsku-api-setup` skill describes. The default page size is 50.

### Create an imprint

1. Look up `POST /imprints` in the spec.
2. Send `default_supplier_id`, the only required field, and `imprint_name`, up to 50 characters.

commonsku takes the account from your API key. You cannot create an imprint for another company.

Example request, with made-up values:

```sh
curl -sS -X POST "$COMMONSKU_BASE_URL/imprints" \
  -H "x-api-key: <your-api-key>" \
  -H "Content-Type: application/json" \
  -d '{
    "default_supplier_id": "5d7e2a10-0000-4000-8000-000000000001",
    "imprint_name": "Example Screen Print",
    "imprint_front_end_enabled": 1
  }'
```

The response is `201` with the new imprint. Keep its ID.

### Update an imprint

Call `PUT /imprints/{id}`. The API key's user needs the `MODIFY-SETTINGS` capability, and the imprint must belong to your company.

## Gotchas

- Every imprint operation carries the spec's Beta Feature label.
- Send `imprint_front_end_enabled` as `0` or `1`. The spec gives its default as `false`.
- `ext_decoration_id`, `ext_artwork_id` and `ext_location_id` appear in responses, but no request can set them.
- `PUT /imprints/{id}` returns `403` when the imprint belongs to another company or the user lacks `MODIFY-SETTINGS`, and `404` when the imprint is not found or is inactive.

## Known issues

None known.

## What the API cannot do

- **Delete an imprint.** There is no `DELETE` operation for imprints.
- **Upload a file to an imprint.** Imprints hold no files.
- **Explain everything.** The spec documents no errors for `POST /imprints`, and does not say whether creating an imprint needs `MODIFY-SETTINGS`.

Last verified: 2026-11-26
