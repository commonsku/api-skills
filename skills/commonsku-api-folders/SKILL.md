---
name: commonsku-api-folders
description: "Read commonsku folders and the files in them through the API: list a record's folders, or get one folder by ID. Use this when someone needs a folder_id, wants to list the files attached to a client, product or other record, or asks whether the API can create or change folders."
---

# Folders

## What this skill helps you do

This skill covers folders in commonsku. A folder groups files that belong to a parent record. The API only reads folders. To put a file in a folder, or move it out, use the `commonsku-api-files` skill.

Check each operation's current parameters and response schema in the spec you downloaded. This skill does not repeat them.

## Steps

### List a record's folders and their files

1. Look up `GET /folder` in the spec.
2. Call it with `parent_type` and `parent_id`. Both are required.
3. Each folder in the response has a `files` array.

Example request, with made-up values:

```sh
curl -sS "$COMMONSKU_BASE_URL/folder?parent_type=<parent-type>&parent_id=3f2b8c1d-0000-4000-8000-000000000001" \
  -H "x-api-key: <your-api-key>" \
  -H "Accept: application/json"
```

Example response (`200`), shortened, with made-up values:

```json
[
  {
    "folder_name": "Example folder",
    "display_order": 1,
    "files": [
      { "file_id": "9a1e4b7c-0000-4000-8000-000000000002", "file_display_name": "Example logo" }
    ]
  }
]
```

### Get one folder

Call `GET /folder/{id}` with the folder's ID. The response holds a `folder` array. Each row combines the folder's fields with a file's fields.

### Put a file in a folder

The folder must already exist. Pass its `folder_id` to the file operations, as the `commonsku-api-files` skill describes.

## Gotchas

- `parent_type` is free text. The spec does not list the allowed values.
- The spec's `GET /folder` response types each folder's `folder_id` and `parent_id` as integers. They are UUID strings, as everywhere else. Treat them as strings.

## Known issues

None known.

## What the API cannot do

- **Create, rename or delete a folder.** There are only the two `GET` operations. Folders are made in commonsku.
- **Explain everything.** The spec does not say what the optional `client_id` parameter does, or what causes a `400`.

Last verified: 2026-11-26
