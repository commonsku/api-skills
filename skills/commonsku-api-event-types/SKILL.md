---
name: commonsku-api-event-types
description: "List the event types commonsku accepts and pick one for a project. Use this when someone creates or updates a commonsku project through the API, needs a valid event_type, or asks which event types commonsku supports."
---

# Event types

## What this skill helps you do

This skill helps you work with the commonsku event type list. Every new project needs an event type, so check this list first to find a valid `event_type`.

## Steps

### List the event types

1. Look up `GET /event_types` in the spec you downloaded. Check its current parameters and response schema.
2. Call `GET /event_types` with `page_limit` set.
3. Page with `cursor` until no further cursor comes back, as the `commonsku-api-setup` skill describes.
4. To sort, pass `sort=event_type` or `sort=event_type_id`. Add a `-` prefix for descending order, as in `sort=-event_type`.

Example request:

```sh
curl -sS "$COMMONSKU_BASE_URL/event_types?page_limit=50&sort=event_type" \
  -H "x-api-key: <your-api-key>" \
  -H "Accept: application/json"
```

Example response, with made-up values:

```json
{
  "data": [
    { "event_type_id": "7c4d2a9e-0000-4000-8000-000000000001", "event_type": "Example Event A" },
    { "event_type_id": "7c4d2a9e-0000-4000-8000-000000000002", "event_type": "Example Event B" }
  ],
  "meta": { "page_limit": 50 }
}
```

### Pick an event type for a project

1. List the event types.
2. Pass the chosen `event_type` name when you create a project. `POST /projects` requires it. `PUT /projects/{id}` also accepts it.

## Gotchas

- Projects take the event type's name in `event_type`, not its `event_type_id`. An unknown name returns `400`.
- Changing `event_type` with `PUT /projects/{id}` also changes it on every order in the project.
- commonsku may add event types in the future. Existing event types stay supported. Fetch the list. Do not hard-code it.

## Known issues

None known.

## What the API cannot do

- **Create, change or delete an event type.** The API has only one event type operation, `GET /event_types`.

Last verified: 2026-11-25
