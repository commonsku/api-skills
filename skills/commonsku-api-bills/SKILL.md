---
name: commonsku-api-bills
description: "Create bills from supplier invoices by vouching them against purchase orders in commonsku, and list or read bills. Use this when someone automates supplier bill entry, calls /bills or /bills/prepare, or a bill is rejected, missing lines or shows the wrong status."
---

# Bills

## What this skill helps you do

A bill is a supplier's invoice to the distributor, vouched against purchase order lines. These operations create bills and read them back. New to the commonsku API? Start with the `commonsku-api-setup` skill.

## Steps

### Create a bill

1. Send `GET /bills/prepare?po_number=<PO number>`. Use the PO form number, or the number prefixed with the project number. The response is shaped like the create request and lists every product and cost line on the PO.
2. Fill in `bill_reference_number` (the supplier's invoice number), `bill_date_billdate` and `bill_date_due`. They are pre-filled when a matching PromoStandards invoice exists.
3. Set each line's `quantity` and `unit_cost` to match the supplier invoice. Lines already fully billed come back at 0 or less.
4. Send the payload to `POST /bills`. Read-only fields are ignored, so lines can go back as they came.

### List or read bills

- `GET /bills` filters on `date_updated`, `status_name`, `exported`, `transaction_date`, `bill_due`, `date_exported`, `vendor_id_number` and `ref_number`. Its `page_limit` maximum is 50, not 250.
- `GET /bills/{id}` returns one bill.
- To react to new bills instead of polling, see the Bill Creation webhook in the `commonsku-api-webhooks` skill.

## Gotchas

- A bill has two states: Vouched and Exported. `POST /bills` creates a Vouched bill. It becomes Exported only through Bill Export in commonsku. There is no Received state.
- `bill_reference_number` must be unique for each supplier division. A repeat returns `400`.
- Add every cost, such as inbound freight, to the sales order before you create the first bill for that PO. A sales order change shows on the PO only after the PO is regenerated, and a PO with a bill attached cannot be regenerated until the bill is deleted ([help article](https://help.commonsku.com/knowledge/purchase-orders)).
- For Connected+ and connected suppliers that offer electronic POs, commonsku can create the bill automatically when it fully matches the PO ([Automated Billing](https://help.commonsku.com/knowledge/automated-billing)). Before creating a bill, call `GET /bills/prepare` with the PO number: lines already fully billed come back with a quantity of 0 or less.
- Every line's order must use the same currency, and each tax must belong to your account.
- A `403` naming `CREATE-BILL` or `COMPANY-TYPE-DISTRIBUTOR` means the commonsku user behind your API key lacks that permission. See the `commonsku-api-support` skill.

## Known issues

None known.

## What the API cannot do

- **Change or delete a bill.** There is no update or delete operation. Change it in commonsku.
- **Mark a bill exported.** That happens only through Bill Export.
- **Bill a project with no PO.** The API vouches only against PO lines.
- **Add a cost line to a PO only.** Costs are added to the sales order.

Last verified: 2026-11-26
