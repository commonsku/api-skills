# API basics

This file holds the full rules for every call. Where it and the live spec disagree, the live spec is right.

## Contents

- Where the documentation lives
- Base URL
- Authentication and API keys
- Rate limits
- Sandbox and production
- Paging
- Filtering and incremental sync
- Identifiers
- Dates and times
- Status codes
- What is documented elsewhere

## Where the documentation lives

| What | Where |
| --- | --- |
| API reference for people | `https://api.commonsku.com/public_api_doc.php` |
| OpenAPI 3.1 spec for tools | `https://api.commonsku.com/public_api_doc.php?format=json` |
| Link to one operation | `https://api.commonsku.com/public_api_doc.php#/operations/<operationId>` |
| Access, keys and FAQs | `https://help.commonsku.com/knowledge/commonsku-api-getting-started-access-and-faqs` |

The reference page is a JavaScript viewer. A plain fetch of it returns an empty page. Read the `?format=json` version instead.

The spec is about 1 MB. Download it whole with `curl`:

```sh
curl -sS -o commonsku-spec.json "https://api.commonsku.com/public_api_doc.php?format=json"
```

If the JSON does not parse, the download was cut off. Download it again before you decide anything is missing.

## Base URL

Read the request host from `servers[0].url` in the live spec each time you set up. It is not always the host that serves the docs. Do not work it out from the docs address, and do not copy it from older code.

## Authentication and API keys

Send the API key in an `x-api-key` header on every request.

```sh
curl -sS "$COMMONSKU_BASE_URL/clients?page_limit=5" \
  -H "x-api-key: $COMMONSKU_API_KEY" \
  -H "Accept: application/json"
```

### Get a key

commonsku's team creates API keys. There is no screen in commonsku where a customer creates one.

To request API access, or a key that has not arrived, the user emails `support@commonsku.com` with the company name and commonsku account details. The getting-started article lists the steps.

Share the key only with the developer who builds the integration, by a secure method. If a key is exposed, the user contacts support to replace it.

### The `Beta Feature` label

Some operations list `Beta Feature` under `security` in the spec. On 2026-11-18, 47 of 107 operations had it. They include the product, artwork, imprint, breakdown, tag, tax-amount and company operations, some item operations and most shop operations.

The label needs nothing extra on the key. Every operation is available to any key with API access. A 403 on one of these operations has the same causes as anywhere else.

### Capabilities

Many operations check a permission on the identity behind the key. The spec calls it a capability, for example `CREATE-BILL` or `MODIFY-PURCHASE-ORDER`. A few fields depend on plan features. Each operation's 403 description says which applies.

If reads work and a write returns 403 on an operation that supports writes, the user asks support to check the permissions on the account.

## Rate limits

- 20,000 requests a day.
- 100 requests a minute.

The limits apply per API key, and each account has one key. Every integration, script and middleware flow on that key shares the allowance. A heavy job in one can stop the others. The daily count resets at 11:59 PM Eastern time.

A request over a limit returns `429 Too Many Requests`. Design for the limits from the start:

- Fetch lists with a large page size, not many small calls.
- Filter with `date_updated` so each run fetches only what changed.
- Use webhooks for events. Do not poll every few seconds.
- Budget bulk jobs. Writes are one call per record, so a 10,000-record backfill uses half the daily allowance. Spread it over more than one day if the integration must also run normally.
- Send requests one at a time. Never send them in bursts or in parallel loops.
- On a 429, wait, then double the wait after each further 429: for example 1, 2, 4 and 8 seconds, then 1 minute. Set a cap. The response does not say how long to wait.
- Never retry at once in a loop.
- If 429s continue for a long time, the daily allowance is probably used up. Pause the job and resume after the reset. Log where it stopped so it can continue.
- Count requests in the integration, so it knows how close it is to the daily limit.

## Sandbox and production

Production is the live commonsku account. Writes there change real clients, orders and invoices.

A sandbox may be available, depending on the account. The user asks `support@commonsku.com`. A sandbox is a separate system: keys, records and permissions do not carry across. Confirm the URL and key for each environment.

Without a sandbox, test writes on records the business does not mind changing.

## Paging

List operations use one of two styles. Check the operation's parameters in the spec to see which one it uses.

### Cursor style

Clients, addresses, contacts, projects, orders, sales orders, purchase orders, invoices, bills, catalogs, shops and the reference lists use the cursor style.

- `page_limit` sets the page size. The spec does not agree with itself about the default, so always set it. The shared parameter definition gives a maximum of 250.
- `cursor` requests the next page.
- `sort` takes a field name for ascending order, or the name with a `-` prefix for descending. Each operation lists the fields it can sort on.
- Responses have a `data` array and a `meta` object. On list operations that document it, `meta` holds `cursor`, `page_limit`, `page_count` and `next_page`.

Pass the returned `meta.cursor` back as `cursor` until no further cursor comes back. The spec does not describe the last page. Confirm the stopping condition on a short list before you run a large sync.

### Offset style

`GET /products`, `GET /imprints` and `GET /company` use the offset style. `start-index` is a zero-based offset and `max-results` is the page size. Response shapes differ by operation, for example `products` with `total_products`.

## Filtering and incremental sync

Use `date_updated` to fetch what changed since the last run. These list operations accept it: `GET /clients`, `/addresses/client`, `/contacts/client`, `/projects`, `/orders`, `/sales_orders`, `/purchase_orders`, `/invoices`, `/bills` and `/catalogs`.

Date filters take a comparison suffix: `__lt`, `__lte`, `__gt` or `__gte`.

```text
GET /sales_orders?date_updated__gt=2026-01-01T10:00:00Z&sort=date_updated&page_limit=100
```

Other date filters, such as `po_date_created` or `order_date`, use the same suffixes with a plain date such as `2026-01-01`.

Store the time of each successful run. Overlap the next window a little. Processing a record twice is harmless if writes are idempotent. Missing one is not.

Some list operations accept `expand[]=client` to include client data in each row. This saves a second call.

`GET /orders` returns presentations, estimates and opportunities, and requires `order_type`. Sales orders and invoices have their own list operations.

## Identifiers

- Path IDs are UUIDs that commonsku assigns.
- People in the business use form numbers: project number, sales order number, PO number, invoice number. These are filters, such as `project_number` and `po_number`. They are not path IDs.
- `customer_number` on a client is a free-text field for the client's ID in another system. Set it with `POST /clients` or `PUT /clients/{id}`. Filter on it in the client, order, sales order, purchase order and invoice lists. Use it as the cross-reference key for an ERP or accounting sync.

## Dates and times

- Filters documented as date-time take ISO 8601 with a zone, as in `2026-01-01T10:00:00Z`.
- Many response fields use the spec's `NaiveDateTime`: a local date-time with no offset, such as `2026-09-29 14:30:00`. The spec says to read it in the application timezone, and does not name that timezone. Check a record whose time you know before you rely on it.
- An unset `NaiveDateTime` is the string `0000-00-00 00:00:00`. Most date parsers reject it. Test for it first and treat it as empty.
- Webhook payloads use different date formats from each other. See the `commonsku-api-webhooks` skill.

## Status codes

These are the codes the spec documents, plus 429 for rate limits. Each operation lists the codes it can return and what they mean there.

| Code | What it means in this API |
| --- | --- |
| 200, 201 | Success. Some creates return 201 |
| 204, 207, 412 | Catalog item operations: nothing added, partly added, or nothing added because the input was not valid |
| 400 | A parameter is missing or not valid, or the record's state does not allow the action |
| 401 | The API key is missing or not valid |
| 403 | The identity behind the key lacks a capability or plan feature, or the record belongs to another company |
| 404 | Not found. On some operations, a record from another account also returns 404, not 403 |
| 422 | The request body or query failed validation |
| 429 | A rate limit was reached. See Rate limits |
| 500 | Server error |

A 400, 401, 403, 404 or 422 almost always means the request needs a fix. Use the `commonsku-api-support` skill to work through them.

## What is documented elsewhere

- Rate limits are not in the spec. This file gives them.
- Access, keys and sandbox availability are in the getting-started article: `https://help.commonsku.com/knowledge/commonsku-api-getting-started-access-and-faqs`
- Webhooks are not in the spec. See the `commonsku-api-webhooks` skill.

Anything not in the spec, this skill or that article is not documented. Say so. Do not fill the gap with a guess.
