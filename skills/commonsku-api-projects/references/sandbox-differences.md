# Sandbox differences

Everything else in this skill describes production. This file lists only where the sandbox spec differed from the production spec for projects, as of 2026-11-26. The sandbox spec can change at any time, so read it when you work in a sandbox. The `commonsku-api-setup` skill gives its address.

## Missing in sandbox

| What | Where it is missing |
| --- | --- |
| `PUT /projects/{id}` | The whole operation. A sandbox cannot update a project |
| `customer_po` request field | `POST /projects`, along with its `400` error for a `customer_po` that is too long or sent with an `OPPORTUNITY` or `PRESENTATION` starting stage |

## Same in both

The other five operations, their paths, parameters, filters, sort and expand values, request fields, response fields and error responses matched. Responses in both still return `customer_po` on the project.
