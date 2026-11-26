# Project filters and fields

These tables follow the live production spec. Check the spec you downloaded before you rely on them, because it wins if they differ.

## Contents

- `GET /projects` filters
- Sort and expand
- Create fields
- Update fields
- Project fields
- Responses and errors
- Not fully documented

## `GET /projects` filters

| Filter | Type | Notes |
| --- | --- | --- |
| `client_id` | string | Projects of one client |
| `project_number` | string | Projects with this number or higher. Not an exact match |
| `date_created` | date | On or after this date. Range suffixes `__lt`, `__lte`, `__gt`, `__gte`, such as `date_created__lt=2026-01-01` |
| `date_inhandsdate` | date | In-hands date on or after this date. Same range suffixes |
| `date_updated` | date-time | Same range suffixes, such as `date_updated__gt=2026-01-01T10:00:00Z` |
| `cursor` | string | Paging cursor |
| `page_limit` | number | Items per page |

## Sort and expand

- `sort` sorts ascending. Prefix a value with `-` to sort descending, such as `-date_created`. Allowed values: `job_number`, `date_created`, `date_inhandsdate`.
- The sort value for the project number is `job_number`, while the filter and field are `project_number`.
- `expand[]=client` adds client data. It is the only expand value.

## Create fields

`POST /projects` creates a project for an existing client.

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `client_id` | string (UUID) | Yes | The client. The `commonsku-api-clients` skill covers clients |
| `project_name` | string | Yes | |
| `event_type` | string | Yes | A name from `GET /event_types`. See the `commonsku-api-event-types` skill |
| `starting_stage` | string | Yes | `OPPORTUNITY`, `PRESENTATION`, `ESTIMATE` or `SALES ORDER` |
| `budget` | string | Yes | |
| `inhands_date` | date | Yes | `YYYY-MM-DD` |
| `created_by` | email | Yes | Email of the user creating the project |
| `order_rep` | email | Yes | Email of the order rep |
| `client_rep` | email | Yes | Email of the client rep |
| `client_contact` | email | No | Email of the client contact |
| `project_status` | string or null | No | The spec does not list the allowed values |
| `customer_po` | string | No | Up to 100 characters. Saved on the project and its form. Only with an `ESTIMATE` or `SALES ORDER` starting stage |
| `billing_address_id` | string (UUID) | No | An address from the client's address book. Do not send with `billing_address` |
| `billing_address` | object | No | A one-off address, below. Do not send with `billing_address_id` |
| `shipping_address_id` | string (UUID) | No | An address from the client's address book. Do not send with `shipping_address` |
| `shipping_address` | object | No | A one-off address, below. Do not send with `shipping_address_id` |

An inline address has `address_line_1` (required), `address_line_2` to `address_line_4`, `address_city`, `address_state`, `address_postal` and `address_country`.

The `commonsku-api-users` skill says which user value each user field takes. The `commonsku-api-clients` skill covers the client's address book and contacts.

## Update fields

`PUT /projects/{id}` updates only the fields in the request body.

| Field | Type | Notes |
| --- | --- | --- |
| `project_name` | string | Cannot be empty |
| `inhands_date` | date | `YYYY-MM-DD`. Also applied to every order in the project |
| `budget` | number | Also applied to every order in the project |
| `event_type` | string | A name from `GET /event_types`. Also applied to every order in the project |
| `order_rep` | string | `login_name` of the order rep. Also applied to every order in the project |
| `client_rep` | string | `login_name` of the client rep. Needs the `MODIFY-CLIENT-REP` capability. Also applied to every order in the project |

Send one value per field, not a list.

## Project fields

`GET /projects`, `GET /projects/{id}`, `POST /projects` and `PUT /projects/{id}` return the same project object.

| Group | Fields |
| --- | --- |
| Identity | `project_id`, `distributor_id`, `project_number`, `project_name`, `project_status`, `event_type`, `currency`, `customer_po`, `terms` |
| Client | `client_id`, `client_name` |
| Reps | `client_rep_id`, `client_rep_first_name`, `client_rep_last_name`, `client_rep_email`, `order_rep_id`, `order_rep_first_name`, `order_rep_last_name`, `order_rep_email` |
| Billing contact | `billing_contact_id`, `billing_contact_first_name`, `billing_contact_last_name`, `billing_contact_email` |
| Addresses | `billing_address_id`, `shipping_address_id`, each a UUID or null |
| Dates and budget | `date_created` (date), `date_inhandsdate` (date), `date_updated` (date-time), `total_budget` (string) |

- `terms` is the terms name. The `commonsku-api-terms` skill explains it.
- `POST /projects` also returns `form`: `form_id`, `form_number`, `form_type`, and the form's `billing_contact`, `billing_address`, `shipping_contact` and `shipping_address`. It also returns the full `billing_address` and `shipping_address`, when set.

Response wrappers:

| Operation | Wrapper |
| --- | --- |
| `GET /projects` | `data`, a list, and `meta` |
| `GET /projects/{id}` | `data` and `meta` |
| `POST /projects` | `data` |
| `PUT /projects/{id}` | `project` |

## Responses and errors

| Operation | Code | Meaning, from the spec |
| --- | --- | --- |
| `POST /projects` | `400` | An invalid or missing value: client, starting stage, dates, budget or event type; a `customer_po` over 100 characters or sent with an `OPPORTUNITY` or `PRESENTATION` starting stage; an email that does not match a user or contact; or an invalid address or address ID |
| `PUT /projects/{id}` | `400` | No field to update, a list instead of a single value, an empty `project_name`, an invalid `inhands_date` or `budget`, an unknown `event_type`, or a `login_name` that is not an active user of the company |
| `PUT /projects/{id}` | `403` | The project belongs to another company, or `client_rep` was sent without the `MODIFY-CLIENT-REP` capability |
| `PUT /projects/{id}` | `404` | Project not found |

The spec lists no error codes for `GET /projects` and `GET /projects/{id}`.

## Not fully documented

- The allowed values of `project_status`.
- Whether `POST /projects` needs a capability to set `client_rep`. The spec names `MODIFY-CLIENT-REP` only for update.
