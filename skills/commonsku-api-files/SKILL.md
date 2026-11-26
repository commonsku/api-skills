---
name: commonsku-api-files
description: "Link files to commonsku records and update them through the API: attach a file by URL, link or copy an existing file, rename it or move it to a folder. Use this when someone attaches a file, image or artwork file to a client, product or other record, needs a file_id for another call, or asks whether the API can upload a file."
---

# Files

## What this skill helps you do

This skill covers files in commonsku. A file is its own record, linked to a parent record such as a client or product. Other records, such as artwork, point to a file by its `file_id` instead of holding the file. Create or find the file first, then pass its `file_id` to the call that needs it.

Check each operation's current fields in the spec you downloaded. This skill does not repeat them.

## Steps

### Link a file by URL

1. Look up `POST /file` in the spec.
2. Send JSON with `parent_type`, `parent_id` and `url`.
3. Keep the `file_id` from the response.

Example request, with made-up values:

```sh
curl -sS -X POST "$COMMONSKU_BASE_URL/file" \
  -H "x-api-key: <your-api-key>" \
  -H "Content-Type: application/json" \
  -d '{
    "parent_type": "CLIENT",
    "parent_id": "3f2b8c1d-0000-4000-8000-000000000001",
    "url": "https://files.example.com/logo.png",
    "file_display_name": "Example Promo Co logo"
  }'
```

Example response (`201`), shortened, with made-up values:

```json
{
  "file": {
    "file_id": "9a1e4b7c-0000-4000-8000-000000000002",
    "parent_type": "CLIENT",
    "parent_id": "3f2b8c1d-0000-4000-8000-000000000001",
    "original_url": "https://files.example.com/logo.png",
    "is_image": true
  }
}
```

### Link or copy an existing file

Call `POST /file` with `file_id`, `parent_type` and `parent_id`. Add `"duplicate": true` to link a copy instead of the same file.

### Rename a file or move it to a folder

Call `PUT /file/{id}`. It changes only `file_display_name` and `folder_id`.

### Read files back

There is no `GET /file`. Call `GET /folder` with `parent_type` and `parent_id`, or `GET /folder/{id}`. Folder responses list the files in each folder.

### Use a file in artwork

Pass the `file_id` as `artwork_file_id`. The `commonsku-api-artwork` skill gives the steps.

## Gotchas

- `POST /file` takes JSON only. It does not accept file bytes, multipart or base64. Host the file at a URL, or reuse an existing `file_id`.
- `parent_type` is free text. The spec gives `CLIENT` and `PRODUCT` as examples, with no full list.

## Known issues

None known.

## What the API cannot do

- **Upload file content.** Link a URL instead.
- **Get or delete a single file.** There is no `GET /file/{id}` or `DELETE` for files.
- **Tell you the limits.** The spec lists no accepted file types, no size limit, and does not say what commonsku does with a linked URL. Do not assume.

Last verified: 2026-11-26
