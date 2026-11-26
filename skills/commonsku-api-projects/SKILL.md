---
name: commonsku-api-projects
description: "List, read, create and update commonsku projects, add a note or task to a project, and read its overview feed, with /projects. Use this when someone creates a project at a starting stage, syncs projects to another system, finds a project by number, changes a project's reps, in-hands date, budget or event type, or posts a project note."
---

# Projects

## What this skill helps you do

A project holds a distributor's orders for one client and the messages about them. This skill covers the six project operations. The `commonsku-api-concepts` skill explains stages and how orders come from projects.

New to the commonsku API? Start with the `commonsku-api-setup` skill.

## Steps

### List, find or read projects

1. Check `GET /projects` in the spec you downloaded. Call it with `page_limit` and filters, and page with `cursor`. For a sync, filter on `date_updated__gt`.
2. To find a project by number, filter on `project_number`, then keep the record whose `project_number` matches.
3. To read one project, call `GET /projects/{id}` with the `project_id`.

### Create a project

1. Check the request body of `POST /projects` in the spec. Get `event_type` from the `commonsku-api-event-types` skill and user values from the `commonsku-api-users` skill.
2. Set `starting_stage`: `OPPORTUNITY`, `PRESENTATION`, `ESTIMATE` or `SALES ORDER`.
3. Show the request and get a go-ahead before you send it.

```json
{ "client_id": "3f2a1b4c-0000-4000-8000-000000000001", "project_name": "Example Spring Event",
  "event_type": "Example Event A", "starting_stage": "ESTIMATE", "budget": "2500",
  "inhands_date": "2027-03-15", "created_by": "rep@example.com",
  "order_rep": "rep@example.com", "client_rep": "rep@example.com" }
```

The response includes `data.form`, the order created at the starting stage.

### Update a project

Send `PUT /projects/{id}` with only the fields you are changing: `project_name`, `inhands_date`, `budget`, `event_type`, `order_rep` or `client_rep`. The project comes back under `project`.

### Add a note or task

Send `POST /projects/{id}/notes` with `message_text`. Add `date_reminder` to make it a task.

### Read the overview feed

Call `GET /projects/{id}/overview`. Entries come pinned first, then newest first.

## Gotchas

- `project_number` returns projects with that number or higher, not an exact match.
- Responses use different wrappers: `data`, `project`, `note` or `projectfeed`.
- An update to the in-hands date, budget, event type or reps also applies to every order in the project.
- Changing `client_rep` on update needs the `MODIFY-CLIENT-REP` capability, or it returns `403`.
- Create takes `budget` as a string; update takes a number.
- `customer_po` on create works only with an `ESTIMATE` or `SALES ORDER` starting stage.
- For each address on create, send an address book ID or an inline address, not both.

## Known issues

None known.

## What the API cannot do

- **Delete a project.** No operation deletes one.
- **Change a project's client, stage, addresses, `customer_po` or status.** Update accepts only the six fields above. For moving an order to a later stage, see the `commonsku-api-concepts` skill.
- **Edit, delete or list notes on their own.** Notes can only be added, and read through the overview feed.

## Reference files

| File | Read it when |
| --- | --- |
| `references/fields-and-filters.md` | You need every filter, sort value, create or update field, response field or error |
| `references/notes-and-overview.md` | You add a note or task, tag users, or read the overview feed |
| `references/sandbox-differences.md` | You work against a sandbox and an operation or field is missing |

Last verified: 2026-11-26
