---
name: commonsku-api-industries
description: "List the industries commonsku accepts and pick one for a client. Use this when someone creates or updates a commonsku client through the API, needs a valid industry_name, or asks which industries commonsku supports."
---

# Industries

## What this skill helps you do

This skill helps you work with the commonsku industry list. Every new client needs an industry, so check this list first to find a valid `industry_name`.

New to the commonsku API? Start with the `commonsku-api-setup` skill. It walks you through the API key, the base URL, the live spec and paging.

## Steps

### List the industries

1. Look up `GET /industries` in the spec you downloaded. Check its current parameters and response schema.
2. Call `GET /industries` with `page_limit` set.
3. Page with `cursor` until no further cursor comes back, as the `commonsku-api-setup` skill describes.
4. To sort, pass `sort=industry_name` or `sort=industry_id`. Add a `-` prefix for descending order, as in `sort=-industry_name`.

Example request:

```sh
curl -sS "$COMMONSKU_BASE_URL/industries?page_limit=50&sort=industry_name" \
  -H "x-api-key: <your-api-key>" \
  -H "Accept: application/json"
```

Example response, with made-up values:

```json
{
  "data": [
    { "industry_id": "3f2b8c1e-0000-4000-8000-000000000001", "industry_name": "Example Industry A" },
    { "industry_id": "3f2b8c1e-0000-4000-8000-000000000002", "industry_name": "Example Industry B" }
  ],
  "meta": { "page_limit": 50 }
}
```

### Pick an industry for a client

1. List the industries.
2. Pass the chosen `industry_name` when you create a client. `POST /clients` requires it. `PUT /clients/{id}` also accepts it.

## Gotchas

- commonsku may add industries in the future. Existing industries stay supported. Fetch the list. Do not hard-code it.
- For this list, `page_limit` runs from 1 to 100 and defaults to 20. That is lower than the general maximum in the `commonsku-api-setup` skill.
- Clients take the industry's name in `industry_name`. Accounts take its ID in `industry_id`, as the `commonsku-api-accounts` skill describes.
- The spec does not say whether `industry_name` matching ignores case or spacing. Send the name exactly as `GET /industries` returns it.

## Known issues

None known.

## What the API cannot do

- **Create, change or delete an industry.** The API has only one industry operation, `GET /industries`.
- **Tell you about an unknown industry name.** The spec documents no error for an `industry_name` that is not on the list.

Last verified: 2026-11-26
