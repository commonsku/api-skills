# Tag fields, filters and errors

Read this when you need every `GET /tags` filter, the shape of a tag in a response, or what an error from a tag call means. The live spec wins: check it before you send a request.

## `GET /tags` parameters

Every parameter is optional. Results are always limited to the company the API key belongs to.

| Parameter | Type | What it does |
| --- | --- | --- |
| `resource_id` | UUID | Returns the tags on this one record |
| `resource_type` | `CLIENT`, `PRODUCT`, `ORDER`, `COMPANY-CONTACT`, `TENANT-ACCOUNT` or `SUPPLIER-ACCOUNT` | Returns every tag used on records of this type |
| `label` | string | Matches tags whose label starts with this text |
| `left_wildcard` | `0` or `1` | With `1`, `label` matches anywhere in the label instead of at the start |
| `simple` | any value | When present, returns the simple view below instead of the full view |
| `max_results` | integer | The most tags to return. Without it, all tags return |
| `start_index` | integer | Zero-based offset, for paging with `max_results` |

Page through the list once per run and cache it. Do not call `GET /tags` once per record.

## Response shapes

`GET /tags` returns `{ "tags": [ ... ] }`. What each entry holds depends on the parameters:

- With `resource_id` or `resource_type`, each tag is merged with data about the resource. The spec does not list those extra fields.
- Without them, each tag is the full view, or the simple view when `simple` is present.

### Full view

| Field | Type | Notes |
| --- | --- | --- |
| `tag_id` | UUID | |
| `label` | string | |
| `description` | string or null | |
| `company_id` | UUID | |
| `occurrences` | object | Count of tagged records, keyed by `TENANT-ACCOUNT`, `SUPPLIER-ACCOUNT`, `CLIENT`, `PRODUCT`, `COMPANY-CONTACT`, `ORDER` and `ALL` |
| `can_remove_tags` | boolean | Defaults to `false`. The spec does not say what it controls |

### Simple view

`tag_id`, `label`, `description` and `company_id`, as in the full view.

### Create and update responses

- `POST /tags` returns `tag_id` and `label`.
- `PUT /tags/{id}` returns `tag`, in the simple view.

## Fields you can send

| Operation | Field | Required | Notes |
| --- | --- | --- | --- |
| `POST /tags` | `action` | Yes | Must be `create` |
| `POST /tags` | `label` | Yes | Cannot contain commas |
| `PUT /tags/{id}` | `label` | No | The new label |
| `PUT /tags/{id}` | `description` | No | String or null |

## Errors

| Operation | Status | Cause |
| --- | --- | --- |
| `POST /tags` | `400` | A required field is missing, the label contains a comma, or the identity behind the key lacks the `CREATE-TAGS` capability |
| `PUT /tags/{id}` | `403` | The tag belongs to another company |
| `PUT /tags/{id}` | `404` | The tag ID is missing |

The `commonsku-api-setup` skill covers capabilities, and what to do when reads work but writes fail.

## Not confirmed

The spec does not say:

- Whether `PUT /tags/{id}` rejects a label with a comma, or a label another tag already uses.
- Whether `PUT /tags/{id}` needs a capability.
- What `can_remove_tags` controls.
- Which fields the `resource_id` and `resource_type` modes add to each tag.
