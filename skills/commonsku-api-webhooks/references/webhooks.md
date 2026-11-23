# Webhooks

Webhooks are not in the OpenAPI spec. This file is the reference for them.

commonsku adds webhooks over time, so this file may not list every type. If the user describes a webhook this file does not cover, say it is not documented here. Do not guess.

## Contents

- How webhooks work
- Webhook types
- Payloads
- Verifying the signature
- Responding to a delivery
- Mapping notes by webhook
- What webhooks do not cover

## How webhooks work

A webhook sends data out of commonsku to another system when something happens. Each delivery is an HTTP POST to a URL the customer provides. The customer's system or middleware hosts that endpoint, not commonsku. commonsku must be able to reach it.

### Set one up

1. In commonsku, click the profile icon (top right) > **Settings**.
2. Open the **Webhooks & API** tile.
3. Open the tab for the webhook you need, such as Purchase Orders, Orders or Finance.
4. In the row for the webhook type, enter the URL that receives the payloads, and switch it to **Active**.
5. Enter a password in the **Password** field. commonsku uses it to sign each delivery. See Verifying the signature.

If the Settings menu or the Webhooks & API tile is not visible, the user may not have permission to change company settings. They ask an admin on the account, or `support@commonsku.com`.

### While testing

The password can stay blank while the endpoint is being built, so a signature mismatch does not reject deliveries. Skip signature checks in the test handler too. Before real data flows:

1. Set a password in Webhooks & API.
2. Turn signature checks on in the handler.
3. Send a test delivery to confirm the two match.

With a blank password, the endpoint cannot tell whether a delivery came from commonsku.

Webhooks only send data out. To write into commonsku, use the REST API.

## Webhook types

There are eight webhook types, which send on their own, and two manual exports, which a person starts.

### The eight webhook types

| Webhook | When it sends | Creates or updates |
| --- | --- | --- |
| Customer Information | A customer record is sent to your system | Create only. Later edits are not sent |
| Vendors | A vendor is sent to your system | Create only. Vendor details are not updated |
| Bill Creation | A bill is created | Create only |
| Invoice Creation | An invoice is created | Create only |
| Purchase Order Created | A new purchase order is created | New record |
| Purchase Order Modified | An existing purchase order is updated | Update |
| Sales Order Modified | An existing sales order's status changes | Update |
| Estimates Modified | An existing estimate's status changes | Update |

### The two manual exports

| Export | When it sends | Creates or updates |
| --- | --- | --- |
| Bill Export | A user selects bills and chooses Export to Webhook | Create only. Supports bulk |
| Invoice Export | A user selects invoices and chooses Export to Webhook | Create only. Supports bulk |

Notes:

- **The manual exports need a setting change first.** Switch the account's accounting settings to "Custom Webhook". Then a user selects one or more records in the Bills or Invoices list and chooses "Export to Webhook". Someone still has to click: the setting changes where the export goes, not whether it is manual. If the account already exports to a built-in accounting integration, find out what the switch means for it before you change it.
- **Estimates Modified sends on status changes only.** It does not send when items are added or removed or shipping details change.
- **Sales Order Modified is documented as sending on status changes.** The documentation does not say it sends for other edits. Do not rely on it for them.
- **The exact trigger for Customer Information and Vendors is not documented** beyond the fact that they create records. Confirm the behaviour with a test record.

## Payloads

| Webhook | Shape |
| --- | --- |
| Bills | `{"bills": [ ... ]}`. Each entry has the shape the API returns for a bill. A bulk export puts several bills in one array |
| Invoices | `{"invoices": [ ... ]}`. Each entry has the shape of `GET /invoices/{id}` |
| Purchase orders | The shape of `GET /purchase_orders/{id}`, plus `webhook_event_type`. The documented example value for a new purchase order is `PO.NEW` |
| Sales orders and estimates | Their own shape, described in the mapping notes below |

Look up `GET /invoices/{id}` and the same for the other types in the live spec to see the field lists.

The documentation does not give the `webhook_event_type` value for Purchase Order Modified. It does not publish a full schema for the sales order and estimate payloads. Do not guess either. Point the webhook at a test endpoint, capture a real delivery, and build from that.

## Verifying the signature

commonsku signs every delivery with the password set in Webhooks & API:

- The signature is in the `X-commonsku-Hmac-SHA256` header.
- It is the HMAC-SHA256 of the JSON request body, keyed with the password, then base64-encoded.

The endpoint computes the same value and compares. If they match, the request came from commonsku and did not change on the way. If they do not match, reject it.

Two details decide whether this works:

- **Sign the raw body exactly as it arrived.** Compute the HMAC over the received bytes before you parse them. Parsing the JSON and serializing it again changes spacing and key order, and the signature no longer matches.
- **Compare in constant time.** Use the language's timing-safe comparison, not `==`. An ordinary comparison leaks how much of the string matched.

Python:

```python
import base64, hashlib, hmac


def is_valid_signature(raw_body: bytes, header_value: str, password: str) -> bool:
    digest = hmac.new(password.encode("utf-8"), raw_body, hashlib.sha256).digest()
    expected = base64.b64encode(digest).decode("ascii")
    return hmac.compare_digest(expected, header_value or "")
```

Node.js:

```javascript
const crypto = require("crypto");

function isValidSignature(rawBody, headerValue, password) {
  const expected = crypto.createHmac("sha256", password).update(rawBody).digest("base64");
  const a = Buffer.from(expected);
  const b = Buffer.from(headerValue || "");
  return a.length === b.length && crypto.timingSafeEqual(a, b);
}
```

PHP:

```php
$json = file_get_contents('php://input');
$expected = base64_encode(hash_hmac('sha256', $json, $password, true));
if (!hash_equals($expected, $_SERVER['HTTP_X_COMMONSKU_HMAC_SHA256'] ?? '')) {
    http_response_code(401);
    exit;
}
```

In a web framework, make sure the handler can read the unparsed body. Many frameworks parse JSON before your code runs, and need a setting to keep the raw bytes.

## Responding to a delivery

commonsku reads the response to decide whether the delivery worked.

- **Any response that is not 2XX is a failure.** Return the real error code when something goes wrong.
- **Purchase order, sales order and estimate webhooks need only a 2XX.** They expect no response body. This includes both purchase order webhooks.
- **Bill and invoice exports need a 2XX and a specific JSON body.** Without the body, commonsku keeps waiting and does not update the export status in the account.

Bill:

```json
[{"ref_number": 123, "result": "pass", "result_text": "Export successfully"}]
```

Invoice:

```json
[{"invoice_no": 123, "result": "pass", "result_text": "Export successfully"}]
```

The body must have exactly this structure for the type. `ref_number` and `invoice_no` name the record being acknowledged. The documented example shows a number and says no more. Confirm which value it expects with a test export.

The documentation does not cover these points. Test them. Do not assume.

- What to return for a bulk export. The response is an array, so one object per record is the natural reading. Try it with two records before you rely on it.
- What value `result` takes for a failed record.
- Whether commonsku retries a failed delivery, and how long it waits for a response.

Because of the last point, acknowledge fast and do slow work afterwards. Make the handler safe to receive the same record twice.

## Mapping notes by webhook

Most of the work in a webhook integration is mapping, not the trigger.

### Customers

- Match existing customers on company name or email. If the client's `customer_number` is kept in step with the other system through the API, use it instead. It is more reliable.
- Map payment terms directly. Where there is no exact match, apply a default such as Net 30.
- Map currency names to the other system's values.

### Vendors

- Store the vendor's account number on the supplier page in commonsku. Map it to the vendor ID in the other system. If the vendor does not exist there, use the company name as the vendor name.
- Match existing vendors on vendor name or account number.
- The Orders Email field is mandatory. Map it to the bill email.
- Map terms the same way as for customers.

### Bills

- Use the commonsku bill reference number as the ID, to avoid duplicates.
- If the other system can create items when a bill arrives, do that. If not, map every line to one existing non-inventory item.
- `cost` is the rate. The line total is cost × quantity.
- Each item has a `taxes` array of tax labels, for example `["HST"]`. The tax amount is given only for the whole bill, in `total_taxes`. There is no tax amount per line.
- Each item has a billing address. Use the first one as the bill's billing address.

### Invoices

- Use the commonsku invoice number as the ID.
- Ship date and in-hands date are available to map.

### Purchase orders

- Use the commonsku PO number as the ID.
- `vendor_id_number` holds the vendor account number from the supplier page.
- A purchase order has no tax.
- There are four addresses: the vendor's (`address_*`), the destination (`po_ship_to_address`), the client's billing address (`billing_address`) and the client's shipping address (`shipping_address`).

### Sales orders and estimates

- Use `order_uuid` as the ID. The same record sends more than once as its status changes. Update the existing record. Do not create a new one.
- Link to the project with `project_number` and `project_name`. Link to the client with `customer_uuid`, `customer_name` and `customer_number`.
- `status_name` is the current status, for example "Client Approved" or "Invoiced".
- `cost` and `price` are per unit. There is no line subtotal. `total_subtotal` is for the whole order.
- Several lines can share one `item_uuid` when set-up, freight or decoration charges apply to the same product. Use `item_uuid` with `description` to tell lines apart.
- Tax is given per item, by jurisdiction label and amount, and for the order in `total_taxes`.
- Addresses are in `billto_*` and `shipto_*` fields. There is no vendor address.
- On an estimate, fields such as `po_number`, `ship_date` and `ship_method` may be empty.

### Dates differ between payloads

- Sales order and estimate payloads use `YYYY-MM-DD`, and `date_updated` is a UTC timestamp.
- Bill and invoice payloads use `MM/DD/YYYY`.

Do not share one date parser across them.

## What webhooks do not cover

- Edits to customers, vendors, bills and invoices after they are first sent.
- Changes to an estimate other than its status.
- Anything not in the types table above.

For these, poll the REST API on a schedule with `date_updated` filters. See the `commonsku-api-setup` and `commonsku-api-integration-design` skills.
