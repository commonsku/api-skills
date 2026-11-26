---
name: commonsku-api-users
description: "List the commonsku users on an account and find the login_name or user_id another call needs. Use this when someone assigns a client rep, order rep or sales rep, assigns a note's task, or tags users on a note through the API, needs a valid commonsku user, or asks whether the API can create or change users."
---

# Users

## What this skill helps you do

A user is a person who signs in to the commonsku account the API key belongs to, such as a sales rep. Users are not client contacts. The API can only list users. Other calls take a user's `login_name` or `user_id` to assign a rep or a note's task, or to tag users on a note, so list users first to find a valid value.

New to the commonsku API? Start with the `commonsku-api-setup` skill. It walks you through the API key, the base URL, the live spec and paging.

## Steps

### List the users

1. Look up `GET /users` in the spec you downloaded. Check its current parameters and response schema.
2. Call `GET /users` with `page_limit` set. Its maximum is 50.
3. Page with `cursor` until no further cursor comes back, as the `commonsku-api-setup` skill describes.
4. To sort, pass `sort=user_id`, or `sort=-user_id` for descending order. No other sort exists.

Example request:

```sh
curl -sS "$COMMONSKU_BASE_URL/users?page_limit=50" \
  -H "x-api-key: <your-api-key>" \
  -H "Accept: application/json"
```

Example response, with made-up values:

```json
{
  "data": [
    {
      "user_id": "7c1d2e3f-0000-4000-8000-000000000001",
      "login_name": "you@example.com",
      "contact_first_name": "Alex",
      "contact_last_name": "Example"
    }
  ],
  "meta": { "page_limit": 50 }
}
```

### Find one user

1. List the users and match on `login_name`, or on first and last name, in your own code.
2. Pass the value the target call asks for. Check that field's description in the spec: some fields take `login_name`, others take `user_id`. `references/where-users-are-used.md` lists the fields known today.

## Gotchas

- `GET /users` returns only active users with a `login_name`. Deactivated users are not included.
- `GET /users` has no filters and no get-by-ID operation. Page through the list once, cache it, and look users up locally. Do not call it once per record.
- `page_limit` on this operation tops out at 50, lower than many other list operations.
- `login_name` is an email address. Fields that ask for a user's email take this value.
- Match on `login_name`, the value other calls take, rather than on a name.

## Known issues

None known.

## What the API cannot do

- **Create, change, deactivate or delete a user.** The API has only one user operation, `GET /users`.
- **Read a user's role, permissions or active status.** The response holds only `user_id`, `login_name`, `contact_first_name` and `contact_last_name`.

## Reference files

| File | Read it when |
| --- | --- |
| `references/where-users-are-used.md` | You need to know which field on another call takes a `login_name` and which takes a `user_id` |

Last verified: 2026-11-25
