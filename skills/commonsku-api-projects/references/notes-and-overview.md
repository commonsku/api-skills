# Project notes and the overview feed

These tables follow the live production spec. Check the spec you downloaded before you rely on them, because it wins if they differ.

## Add a note or task

`POST /projects/{id}/notes` adds a note to a project. The path `id` is the `project_id`.

| Field | Type | Notes |
| --- | --- | --- |
| `message_text` | string | Required. `@mentions` of user vanity names tag those users |
| `date_reminder` | date or null | `YYYY-MM-DD`. Turns the note into a task due on this date |
| `reminder_user` | string or null | `login_name` of the user the task is assigned to. Defaults to the caller when `date_reminder` is set |
| `tagged_users` | list or null | `login_name` values of the users to tag |

The `commonsku-api-users` skill says where to find `login_name` values. Tagged users are commonsku users, not tags from `GET /tags`.

Example request, with made-up values:

```json
{ "message_text": "Confirm the imprint colours with the client.",
  "date_reminder": "2027-03-01", "reminder_user": "rep@example.com" }
```

The response is `201`, with the note under `note`:

| Field | Notes |
| --- | --- |
| `note_id`, `message_id` | UUIDs |
| `note_type`, `note_subtype` | Such as `NOTE` |
| `parent_type`, `parent_id` | `parent_id` is the project ID |
| `message_text` | The text you sent |
| `date_reminder` | `0000-00-00 00:00:00` when the note is not a task |
| `reminder_user`, `contact_id`, `created_by` | UUIDs |
| `reminder_complete`, `urgent`, `active` | `0` or `1` |
| `date_complete`, `date_created` | Date-time strings, such as `2026-10-30 09:00:00` |

Errors:

| Code | Meaning, from the spec |
| --- | --- |
| `400` | Missing `message_text`, an invalid `date_reminder` or `tagged_users`, or a `login_name` that is not an active user of the company |
| `403` | The project belongs to another company |
| `404` | Project not found |

## Read the overview feed

`GET /projects/{id}/overview` returns the messages and notes posted on the project and on its orders, deposits, purchase orders and proofs. Pinned entries come first, then newest first. Entries come back under `projectfeed`.

| Field | Notes |
| --- | --- |
| `message_id`, `parent_type`, `parent_id`, `origin` | Identity of the entry |
| `details_type`, `details_id` | Such as `NOTE` or `EMAIL` |
| `details_parent_type`, `details_parent_id` | What the entry belongs to |
| `details_parent_form_number` | Form number of the order, deposit or purchase order |
| `details_parent_order_type` | Such as `SALES ORDER` |
| `subject` | String or null |
| `text` | The message as HTML |
| `date`, `latest_update` | Unix timestamps |
| `pinned` | `0` or `1` |
| `unfollowed` | Boolean |
| `actor` | The user who posted it |
| `note` | Note details, when `details_type` is `NOTE` |
| `email` | Email details, when `details_type` is `EMAIL` |
| `company_name`, `company_url` | Strings |
| `files`, `likes`, `replies` | Lists |
| `tags` | Tagged users |

Errors: `403` when the project belongs to another company, and `404` when it is not found.

## Not fully documented

- The fields inside `actor`, `note`, `email`, `files`, `likes`, `tags` and `replies`.
- Whether the feed pages. The spec lists no paging parameters for it.
