# Sandbox differences

Everything else in this skill describes production. This file lists only where the sandbox spec differed from the production spec for purchase orders, as of 2026-11-26. The sandbox spec can change at any time, so read it when you work in a sandbox. The `commonsku-api-setup` skill gives its address.

## Missing in sandbox

| What | Where it is missing |
| --- | --- |
| `next_action_type` filter | `GET /purchase_orders` |
| `next_action_date` filter | `GET /purchase_orders` |
| `next_action_type` field | Responses of all three operations |
| `next_action_date` field | Responses of all three operations |

## Same in both

The three operations, their paths, every other parameter, the sort values, the shipping update fields and the error responses matched.
