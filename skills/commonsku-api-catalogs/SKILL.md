---
name: commonsku-api-catalogs
description: "Read and manage a client's Client Catalog through the API: find or create the catalog, list items and look them up by external SKU, add or remove items, and export the catalog as CSV. Use this when someone calls /catalogs, matches a store SKU to a catalog item, or a catalog export fails."
---

# Client catalogs

## What this skill helps you do

A Client Catalog keeps a client's reorderable products on the client's record in commonsku, with an external SKU on each variant. It needs the Advanced plan or higher, and users need catalog permissions in commonsku ([Client Catalog](https://help.commonsku.com/knowledge/client-catalog)). New to the commonsku API? Start with the `commonsku-api-setup` skill.

## Steps

### Find or create a client's catalog

1. Send `POST /catalogs` with `client_id`. It returns `201` when it creates a catalog and `200` with the existing one, so it's safe to call before every sync.
2. To list catalogs, use `GET /catalogs`. Filter by `client_id` or `date_updated`, and add `expand=client` for the client's details.

### List or look up items

Use `GET /catalogs/{id}/items`.

- `external_sku` finds the item with a variant carrying that exact SKU (not case-sensitive).
- `search` matches item name, description or external SKU.
- Hidden items are left out by default (`hidden=0`). Send `hidden=1` for only hidden items, or `all=1` for everything.
- `status` filters on how many entries still have no external SKU.

Both list operations are paged; follow the cursor as the `commonsku-api-setup` skill describes.

### Add items

Send `POST /catalogs/{id}/items` with `item_ids`, an array of item IDs. Read the status:

- `201`: all added.
- `207`: some added. `failedIds` gives each failed ID with the reason.
- `204` or `412`: nothing added.

### Remove an item

Send `DELETE /catalogs/{id}/items?item_id=<item-id>`. It removes one item per call.

### Export as CSV

Send `POST /catalogs/{id}/generate-csv` with `{"type":"all"}`, `"published"` or `"unpublished"`.

## Gotchas

- New items arrive unpublished. In commonsku, an item needs an external SKU before it can be published, and only published items can be used on presentations, estimates, sales orders and invoices.
- `external_sku` sits on each size and colour variant. A SKU lookup returns the parent item.
- The catalog's `date_updated` may not change when an item inside it changes. Items carry their own `date_updated`, so read items when you need them rather than mirroring the catalog.
- `store_published` and the other `store_` fields belong to inventory stores. They don't show whether a catalog item is live; `hidden` does.
- A catalog item can be added to an order only once. Combine quantities for the same item before adding it.
- An external SKU doesn't carry over when a presentation becomes an estimate or sales order, so it must be set again there.

## Known issues

- **CSV export returns an error body (confirmed 2026-11-26).** CSV export can return `200` with a `text/csv` content type but the body `{"error":"OK"}` instead of the file. Export from the client's Catalog tab in commonsku instead (Export All, Export Published or Export Unpublished).

## What the API cannot do

- Publish or unpublish a catalog item. Do it on the client's Catalog tab.
- Use catalog items in Shops.

Last verified: 2026-11-26
