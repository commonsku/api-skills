# Integration patterns

Confirm every operation named here against the live spec before you build.

## First, check for a built-in integration or Zapier

commonsku has built-in integrations with some systems, including accounting tools such as QuickBooks Online and Xero. They are set up in commonsku's own settings and do not need the API.

commonsku also has a Zapier integration with ready-made triggers and actions. When it covers the workflow, it is easier to set up than a custom build.

Before you design a custom build, have the user check the help centre at `https://help.commonsku.com` for the system they want to connect. Build against the API only for what those do not cover.

## Middleware is the normal shape

Nearly every commonsku integration has three parts:

1. Something reads from the source system, or receives a webhook from it.
2. Something reshapes the data to fit the destination.
3. Something writes to the destination.

All three usually live in one middleware layer. That can be an automation platform such as Zapier, n8n or Make, or a small service the customer's developer writes and hosts. commonsku does not host or run it.

"This needs middleware" is an ordinary yes, not a limit. When you describe a design, name the commonsku operations the middleware reads from and writes to.

## Webhook, poll or manual export

| Situation | Use |
| --- | --- |
| React when a purchase order is created or changed | Purchase Order Created and Modified webhooks |
| React when a sales order or estimate changes status | Sales Order Modified and Estimates Modified webhooks |
| Finance wants to review bills and invoices before they reach the accounting system | The manual Export to Webhook action on bills and invoices |
| Any other change, and as a safety net behind webhooks | A scheduled poll with `date_updated` filters |
| Putting data into commonsku | The REST API. Webhooks only send data out |

A scheduled poll removes the manual click. A job that calls `GET /invoices` or `GET /bills` for records changed since the last run can replace the Export to Webhook step. Both lists also accept `exported` and `date_exported` filters. Check the live spec for how they behave.

A sensible default for an accounting sync:

- Bills and invoices on the manual export, so a person approves what goes across.
- Purchase orders and sales orders on webhooks.
- Everything else on a scheduled poll.

## Match keys

Decide how records in the two systems match before you write any code.

| Record | Key to use |
| --- | --- |
| Client | `customer_number`. Set it to the other system's customer ID with `POST /clients` or `PUT /clients/{id}`, then filter on it |
| Vendor | The vendor account number on the supplier page in commonsku. It arrives as `vendor_id_number` on purchase order payloads |
| Invoice, purchase order, bill | The commonsku form number: invoice number, PO number or bill reference number |
| Sales order or estimate from a webhook | `order_uuid` |
| Anything fetched by path | The commonsku UUID |

Keep a mapping table in the middleware from commonsku UUIDs to the other system's IDs. Do not match on names. A corrected typo breaks the match.

## Common builds

### Accounting or ERP sync

This is the most common build. Data moves both ways, and what is possible depends on the object.

Out of commonsku:

- Invoices, bills, purchase orders and sales orders, by webhook, manual export or poll, as above.
- Clients and contacts, by a poll on `date_updated`, or by the Customer Information webhook for new records.

Into commonsku:

- Create and update clients, contacts and addresses.
- Set `credit_hold` on a client with `PUT /clients/{id}`. The API exposes only the flag. Calculate any credit limit in the other system, and switch the flag from there.
- Mark an invoice paid with `PUT /invoices/{id}`. `status_name` is the only settable field, and `Paid` is its only value.
- Create a bill against purchase order items:
  1. Call `GET /bills/prepare?po_number=...`.
  2. Fill in the bill reference number, bill date and due date in the payload it returns.
  3. Send that payload to `POST /bills`.
- Update shipping details on a purchase order with `PUT /purchase_orders/{id}/shipping`.
- Update a sales order's status and form fields with `PUT /sales_orders/{id}`.

### Production or decoration tracking

The purchase order webhooks drive this build. When a purchase order is created or changed, the payload goes to a board or production system. That system tracks the work in more detail than commonsku holds. The payload has the same fields as `GET /purchase_orders/{id}`, including `po_type`.

This goes one way. Sending status back into commonsku is a separate piece of work, limited to what the update operations accept.

### Orders from an outside storefront

There are two routes, and they are not equal:

- **Connected shop checkout.** `POST /shop/{id}/connected-checkout` creates a sales order from a submitted cart. It takes line items matched by `external_sku` to the shop's variants, the customer, the shipping address, `customer_po`, a discount and metadata. The shop must be a connected shop.
- **Build the order piece by piece.** Create the project with `POST /projects` at the stage you want. Then add items with `POST /items/product` and the related item operations. This takes many calls and more care. Say so. Do not present it as simple.

Ask which route the user's code is on before you answer a field question. The two routes accept different fields.

### Product and catalog data

The product operations cover products, costs, colours, sizes, imprints and custom fields. Catalog operations manage a client's catalog and its items. Look up suppliers with `GET /company` and `company_type=SUPPLIER`. There is no separate supplier endpoint.

## Habits that prevent most problems

- **Stay inside the rate limits.** Plan backfills and polling schedules around 2 requests a second, bursts of up to 4, and 20,000 requests a day per key. Some accounts have a higher daily limit. Ask `support@commonsku.com`. See the `commonsku-api-setup` skill.
- **Make every write idempotent.** Check for a record by its match key before you create one. Webhook deliveries and retried jobs arrive twice sooner or later.
- **Log each request and response with a timestamp**, without the API key. You need it to debug, and support asks for it when something looks like a bug.
- **Retry 5XX responses and timeouts with backoff.** Do not retry a 4XX unchanged. Fix the request.
- **Start one-way and small.** Get one record type flowing in one direction before you add the next.
- **Test on a record the business does not mind changing.**

## The half you cannot see

Whether a design works also depends on the other system. Does it expose the API, event or field the workflow needs, and under what limits? Read its official developer documentation. If it has none in public, say that its side could not be checked. Describe what the commonsku side can do, so the developer knows what to ask their vendor.
