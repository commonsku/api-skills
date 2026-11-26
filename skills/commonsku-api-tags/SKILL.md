---
name: commonsku-api-tags
description: "List, create and rename the tags a commonsku distributor uses to label records such as clients, products and accounts, and find a tag_id or label another call needs. Use this when someone calls /tags, filters tags by label or resource, needs a valid tag, or asks how to tag a record through the API."
---

# Tags

## What this skill helps you do

A tag is a label the distributor defines once for its company, such as "Example VIP", and then applies to records like clients, products and accounts. This skill covers the tag list itself: `GET /tags`, `POST /tags` and `PUT /tags/{id}`. All three carry the spec label `Beta Feature`: access works as usual, but they may still change, so retest after updates.

You apply a tag on the record's own call, listed in `references/tags-on-other-objects.md`. Tagging users on a project note is a different feature: it takes `login_name` values, as the `commonsku-api-users` skill describes.

New to the commonsku API? Start with the `commonsku-api-setup` skill.

## Steps

### List or find tags

1. Look up `GET /tags` in the spec you downloaded. Check its current parameters and response schema.
2. Call `GET /tags` with `max_results` set. Without it, all tags return. Page with `start_index`, a zero-based offset.
3. To find a tag by name, pass `label` (a prefix match). Add `left_wildcard=1` for a contains match.
4. To see the tags used on one record or one record type, pass `resource_id` or `resource_type`. `references/fields-and-filters.md` lists the values and response shapes.

```sh
curl -sS "$COMMONSKU_BASE_URL/tags?label=Example&max_results=50&start_index=0" \
  -H "x-api-key: <your-api-key>" -H "Accept: application/json"
```

```json
{ "tags": [ { "tag_id": "9c1d2e3f-0000-4000-8000-000000000001", "label": "Example VIP",
  "description": null, "company_id": "9c1d2e3f-0000-4000-8000-0000000000aa",
  "occurrences": { "CLIENT": 3, "ALL": 3 }, "can_remove_tags": false } ] }
```

### Create a tag

1. Send `POST /tags` with `action` set to `create` and `label`. Both are required.
2. The response holds `tag_id` and `label`.

```json
{ "action": "create", "label": "Example VIP" }
```

### Rename a tag or set its description

1. Send `PUT /tags/{id}` with the `tag_id`, and `label`, `description` or both. The response holds the updated `tag`.

## Gotchas

- `POST /tags` returns the existing tag when the label already exists for the company. It does not fail and does not create a duplicate.
- A label cannot contain commas. `POST /tags` rejects it with a `400`.
- `POST /tags` needs the `CREATE-TAGS` capability. Without it you get a `400`, not a `403`.
- `POST /tags` cannot set a description. Use `PUT /tags/{id}` after.
- `GET /tags` pages with `max_results` and `start_index`, not the cursor paging other endpoints use.
- The shape of each tag in `GET /tags` depends on the parameters you pass. Check fields exist before reading them.

## Known issues

None known.

## What the API cannot do

- **Delete a tag.** No operation removes a tag.
- **Read one tag by ID.** There is no `GET /tags/{id}`. Filter `GET /tags` by `label`.
- **Tag an order.** Orders, sales orders and invoices only return `tags`.

## Reference files

| File | Read it when |
| --- | --- |
| `references/fields-and-filters.md` | You need every `GET /tags` filter, the response shapes, or the error codes |
| `references/tags-on-other-objects.md` | You apply tags to a client, product, account or shop, or read tags on another object |

Last verified: 2026-11-26
