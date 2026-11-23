---
name: commonsku-api-webhooks
description: "Set up, verify and handle commonsku webhooks and manual exports: types, payloads, signatures, responses and mapping. Use this when someone sets up or debugs a commonsku webhook, verifies a webhook signature, handles a bill or invoice export, or asks which commonsku events can be sent out."
---

# commonsku webhooks

## What this skill helps you do

This skill helps a developer receive commonsku webhooks: choose a webhook type, verify its signature, respond so commonsku records the delivery, and map its payload. Webhooks are not in the OpenAPI spec, so `references/webhooks.md` is the reference for them. For REST API calls, use the `commonsku-api-setup` skill.

## Gotchas

- Webhook payloads use different date formats from each other. Do not share one date parser across them.
- Customer, vendor, bill and invoice webhooks send new records only. Estimates Modified sends on status changes only.
- Bill and invoice exports need a 2XX and a specific JSON body in the response. Without the body, the export stays pending.
- Compute a webhook signature over the raw body before you parse the JSON.

## What the API cannot do

- **Write into commonsku with a webhook.** Webhooks only send data out. Use the REST API to write.
- **Send later edits by webhook.** Edits to customers, vendors, bills and invoices after they are first sent, and estimate changes other than status, do not send. Poll with `date_updated` filters.

## Reference files

| File | Read it when |
| --- | --- |
| `references/webhooks.md` | The task involves webhooks: types, set-up, signatures, responses, mapping |

Last verified: 2026-11-18
