---
name: commonsku-api-concepts
description: "Explain what commonsku names and objects mean in the API and how they relate: supplier, distributor and client, order stages, invoices and bills, tenants and companies, IDs and form numbers, and what each object can and cannot do. Use this when someone works with commonsku API data and needs to know what a record is, which object to use, or whether the API can create or change it."
---

# commonsku names and objects

## What this skill helps you do

This skill explains the commonsku words an agent meets in the API, how the objects relate, and what the API can and cannot do with each object. For how to make a call, use the `commonsku-api-setup` skill.

## commonsku terms

- A supplier makes or decorates branded merchandise and sells it to a distributor. The distributor sells to its client. API users are usually distributors, and the API calls their customers `clients`.
- An order moves through these stages: Opportunity, Presentation, Estimate, Sales Order, Invoice. A Presentation is a distributor's product proposal to a client.
- An invoice is what the distributor sends its client. A bill is a vendor's invoice to the distributor, matched against purchase order items.
- In the spec, a tenant is the commonsku account that the API key belongs to. A company is broader: `company_type` makes it a client or a supplier.

## Gotchas

- `GET /orders` returns opportunities, presentations and estimates, and requires `order_type`. Sales orders and invoices have their own list operations.
- Path IDs are UUIDs. Form numbers, such as a PO number or invoice number, are filters, not path IDs.

## What the API cannot do

- **Create a purchase order.** There is no create operation for purchase orders. `PUT /purchase_orders/{id}/shipping` updates only shipping details.
- **Create a sales order or an invoice directly.** Create a project at the stage you want, upgrade an order with `POST /orders/upgrade`, or submit a connected shop checkout.
- **Change an invoice beyond marking it paid.** `PUT /invoices/{id}` accepts only `status_name`, and its only value is `Paid`.
- **Update a project.** Projects can be created and given notes, but have no update operation.
- **Look up suppliers on their own endpoint.** Use `GET /company` with `company_type=SUPPLIER`.
- **Set a credit limit.** The API exposes only the `credit_hold` flag on a client. Calculate the limit in the other system, and switch the flag from there.

## Reference files

| File | Read it when |
| --- | --- |
| `references/endpoint-map.md` | You want a quick picture of what can be read, created and updated. The live spec wins |

Last verified: 2026-11-18
