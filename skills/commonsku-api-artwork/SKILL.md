---
name: commonsku-api-artwork
description: "Add, copy, read and update artwork on an order item's decoration locations in commonsku, including the proof type and the artwork file. Use this when someone attaches artwork through the API, an artwork file will not attach, or purchase order generation fails because a decoration location has no artwork."
---

# Artwork

## What this skill helps you do

Artwork belongs to a decoration location on an order item. These operations attach artwork, copy it between locations and update its details. New to the commonsku API? Start with the `commonsku-api-setup` skill.

## Steps

### Attach new artwork

1. Find the item's decoration location (`item_location_id`). To create a new location at the same time, send `item_location_id: "NEW"` with the `item_id`.
2. Add the file with `POST /file`, passing `url` as a direct link to the file. Keep the file ID it returns.
3. Look up `POST /artwork` in the spec you downloaded and send `item_location_id`, `imprint_id`, `proof_id`, `artwork_file_id`, `artwork_title`, `artwork_size`, `artwork_color`, `artwork_repeat`, `artwork_notes` and `artwork_dimension_id` (which can be null).

### Copy artwork

Send `POST /artwork` with `item_location_id`, `target_type: "COPY"` and `target_id` set to the artwork to copy.

```json
{ "item_location_id": "NEW", "item_id": "<item-uuid>", "target_type": "COPY", "target_id": "<artwork-uuid>" }
```

### Read or update artwork

`GET /artwork/{id}` and `PUT /artwork/{id}`. Send only the fields you are changing.

## Gotchas

- Pass `url` as a direct link to a single file.
- `proof_id` takes a proof UUID or one of the names `None`, `Email Proof`, `Digital Photo` or `Pre-production`. If the client must approve a proof, choose one of the last three: commonsku accepts a proof upload only when a proof is required ([proofing](https://help.commonsku.com/knowledge/proofing-process)).
- Every decoration location needs artwork before purchase orders can be generated in commonsku. If your integration creates locations without artwork, PO generation in commonsku fails with "Some decoration locations do not have any artwork associated."
- `artwork_uom` maps a unit-of-measure code to a quantity, for example `{"artwork_uom.S": 2}`. The spec doesn't list the valid codes.

## Known issues

None known.

## What the API cannot do

- **Send a proof to the client or read its approval.** The spec has no proofing operation.
- **Upload file contents directly.** `POST /file` takes a link or an existing file ID.

Last verified: 2026-11-26
