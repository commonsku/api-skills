---
name: commonsku-api-shops
description: "List, read, create and update commonsku shops, including connected shops, with /shop, /shop/{id} and /shops/connected. Use this when someone looks up a shop or its shop_id, creates a pop-up, company, marketing or connected shop, changes shop settings, or gets a 400, 403 or 404 from a shop call."
---

# Shops

## What this skill helps you do

A shop is a distributor's online store, where a client's people order products. This skill covers the shop itself: `GET /shop`, `GET /shop/{id}`, `POST /shop`, `PUT /shop/{id}` and `POST /shops/connected`. Every one except `GET /shop` carries the spec label `Beta Feature`: access works as usual, but the operations may still change, so retest after updates.

To submit or read checkouts (`POST /shop/{id}/connected-checkout`, `GET /shop/{id}/checkout`, `GET /checkout/{id}`), use the `commonsku-api-checkouts` skill.

New to the commonsku API? Start with the `commonsku-api-setup` skill.

## Steps

### List or find shops

1. Check the current parameters of `GET /shop` in the spec you downloaded.
2. Call it with `page_limit` set (at most 250) and filters such as `client_id`, `type` or `search`. Page with `cursor`.
3. `references/fields-and-filters.md` lists every filter and field.

```sh
curl -sS "$COMMONSKU_BASE_URL/shop?type=Pop-up&page_limit=50&sort=-date_created" \
  -H "x-api-key: <your-api-key>" -H "Accept: application/json"
```

```json
{ "data": [ { "shop_id": "5a6b7c8d-0000-4000-8000-000000000001", "shop_name": "Example Promo Co Spring Shop",
  "shop_number": 1042, "type": "Pop-up", "status_name": "Example Open", "order_count": 12 } ],
  "meta": { "cursor": "<opaque>", "page_limit": 50 } }
```

### Read one shop

Call `GET /shop/{id}`. The full shop object comes back under `data`.

### Create a shop

- **Pop-up, company or marketing shop:** send `POST /shop` with `shop_name` (required), `shop_type` (`popup`, `company` or `marketing`) and usually `client_id`.
- **Connected shop**, for orders from a storefront commonsku does not host: send `POST /shops/connected` with `shop_name` and `client_id`, both required.

```json
{ "shop_name": "Example Promo Co Spring Shop", "shop_type": "popup",
  "client_id": "5a6b7c8d-0000-4000-8000-0000000000c1" }
```

### Update a shop

Send `PUT /shop/{id}` with only the fields you are changing. Check each one in the spec first.

## Gotchas

- The response wrapper differs: `POST /shop` and `PUT /shop/{id}` return `shop`, `GET /shop/{id}` returns `data`, and `POST /shops/connected` returns the shop fields at the top level.
- The `type` filter and the `shop_type` create field use different values: `Pop-up` to filter, `popup` to create.
- `POST /shop` rejects a connected `shop_type` with a `400`. Use `POST /shops/connected`.
- `shop_tags` on `PUT` replaces every tag on the shop. Send the full list. The `commonsku-api-tags` skill covers tags.
- `is_headless` and `external_store_type` on `PUT` need the `FEATURE-ADVANCED` or `FEATURE-ENTERPRISE` capability, or you get a `403`.
- `auto_upgrade_orders_to_sales_order` on `PUT` needs the `ENABLE-CONNECTED-SHOPS` flag. On a connected shop, a missing flag gives a `403`. On other shops, the field is silently dropped. Read the shop back to confirm.

## Known issues

None known.

## What the API cannot do

- **Delete a shop.** No operation deletes a shop.
- **List shop status IDs.** No operation lists them. Read `status_id` from `GET /shop/{id}`.

## Reference files

| File | Read it when |
| --- | --- |
| `references/fields-and-filters.md` | You need every `GET /shop` filter, the shop fields, what each call can set, or the error codes |
| `references/shop-types.md` | You choose between a normal and a connected shop, or connect an external store |

Last verified: 2026-11-26
