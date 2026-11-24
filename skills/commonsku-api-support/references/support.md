# When something goes wrong, and what commonsku support covers

## Contents

- What commonsku support covers
- Triage: implementation issue or platform bug
- Signs it may be a platform bug
- What to send
- Where to send it
- Report template
- How to talk about it

## What commonsku support covers

| commonsku technical support helps with | The customer's developer handles |
| --- | --- |
| The API returning something other than what the documentation describes | Writing, reviewing and debugging integration code |
| A webhook not sending, or sending something other than what is documented | Choosing, configuring and running middleware |
| Server errors on a valid request | Designing field mappings and workflows |
| | Anything in the other system |

This skill is for the right-hand column.

Questions about access also go to `support@commonsku.com`: whether an account has API access, a key that has not arrived or needs replacing, a sandbox, or permissions that seem to be missing. The getting-started article covers these: `https://help.commonsku.com/knowledge/commonsku-api-getting-started`

Some plans include more hands-on integration help. The getting-started article explains who it covers.

## Triage: implementation issue or platform bug

Work through these steps in order. Stop at the first one that explains the problem. Most problems end at step 1, 2 or 3.

### 1. Does the request match the live spec?

Look up the operation in the live spec. Compare the method, path, required fields, field types and allowed values. Check that the field being set is in the request body, not only in a response.

### 2. What does the status code say?

| Code | Check |
| --- | --- |
| 400, 422 | The request. Read the response body for detail. Compare required fields, types and allowed values with the spec |
| 401 | The key. Is the header named `x-api-key`? Is the value complete, with no quotes or spaces? Is the right environment variable loaded? Is it the key for this environment, sandbox or production? A request with no key returns 403 with `{"message":"Forbidden"}`, not the 401 the spec lists. A wrong key, or the other environment's key, can fail the same way |
| 403 | Read the operation's 403 description in the spec. The usual causes are a missing capability on the identity behind the key, a plan feature, or a record that belongs to another company. A missing or wrong key can also return 403. See the 401 row. The `Beta Feature` label itself does not limit access: a Beta endpoint works with your key like any other. It marks recently released endpoints that may still change, so retest after updates. If reads work but writes fail, see the note under this table |
| 404 | The ID. Path IDs are UUIDs, not form numbers. The record may be inactive, deleted, or in another account |
| 429 | A rate limit: 2 requests a second, with bursts of up to 4, and 20,000 a day per key. Some accounts have a higher daily limit. Ask `support@commonsku.com`. The response does not say how long to wait. Back off and retry. If 429s continue, the daily allowance is probably used up. It resets at 11:59 PM Eastern time. Check whether another integration on the same key is using it. See the `commonsku-api-setup` skill |
| 5XX | Go to step 5 |

**Reads work, writes fail.** If reads work but creates or updates fail with a permission error, it is usually an account set-up issue on commonsku's side, not your key or code. Email `support@commonsku.com` with the endpoint, request time and response body, say reads work but writes don't, and never include the API key.

### 3. Is this documented behaviour?

These look like faults but are not:

- **A webhook did not send for an edit.** Customer, vendor, bill and invoice webhooks send new records only. The estimate webhook sends on status changes only.
- **An export stays pending.** Bill and invoice exports need a specific JSON body in the response, not only a 200.
- **A signature never matches.** Compute the HMAC over the raw body before any JSON parsing, with exactly the password set in Settings > Webhooks & API. If that password is blank, there is nothing to verify against.
- **A field does not update.** It may be in the GET response and not in the update operation.
- **A date does not parse.** Unset dates arrive as `0000-00-00 00:00:00`, and webhook payloads use different date formats from each other.
- **Fewer rows than expected.** The list is paged. Follow the cursor.

### 4. Is the failure on the other side?

Read the middleware's own logs. Did the request leave? Did commonsku answer? Did the error come from the other system's API? If the other system rejects a mapping, it is not a commonsku issue.

### 5. Can one minimal request reproduce it?

Take the integration out of the picture. Make one `curl` call for one record, with the key in an environment variable.

- If the minimal call works, the problem is in the integration.
- If it fails the same way every time and the request matches the spec, it may be a platform bug.

## Signs it may be a platform bug

- A 5XX response, more than once, to a request that matches the spec.
- A response that contradicts the spec: a documented field missing, a documented filter ignored, a documented operation not found.
- Data that commonsku shows for a record but that is missing or different in the API response for the same record.
- An Active webhook with a reachable endpoint that sends nothing for a documented trigger, and the endpoint's logs show no request arrived.
- A signature that fails, computed over the raw body with the correct password, with one of the tested examples in the `commonsku-api-webhooks` skill.
- Something that worked before and stopped, with no change on the customer's side.

You cannot confirm a bug from the outside. Say "this looks like it may be a platform issue and is worth reporting". Do not say "this is a commonsku bug".

## What to send

A complete report gets action the first time. An incomplete one starts a back-and-forth.

- The company name as it appears in commonsku, the commonsku account subdomain, and the name and email of the person reporting.
- One sentence on what the integration was trying to do.
- The operation: method and path, or the webhook type.
- Sandbox or production.
- The exact time of one failing example, with timezone.
- The request: full URL with query string, and the body. **Remove the API key.**
- The response: status code and full body.
- One example record: its project, sales order, purchase order, invoice or bill number, or its UUID.
- What was expected and what happened, with a pointer to the part of the documentation it contradicts.
- How often it happens, when it started, and whether it ever worked.
- A minimal `curl` command that reproduces it, with a placeholder for the key.
- What is already ruled out.

For a webhook problem, also send:

- The webhook type, the Webhooks & API tab it is on, and whether it is Active.
- The endpoint's host name.
- What was done in commonsku to trigger it, to which record, and when.
- What the endpoint logged and what it returned.

Never send an API key or a webhook password to anyone, support included. If one was pasted into a chat, an email or a repository, treat it as exposed and replace it.

## Where to send it

- Email `support@commonsku.com`. This is the address for API and webhook bug reports.
- In commonsku, open Resources > Support for the support chat.

The help centre lists support hours as 9 AM to 8 PM EST, Monday to Friday, except Canadian public holidays. Email is available outside those hours. Check the current details at `https://help.commonsku.com/knowledge/how-to-search-commonskus-knowledge-base`.

## Report template

Fill this in from the conversation and the logs. Leave a line out only if it does not apply. Do not fill a gap with a guess. Ask for the missing detail.

```text
Subject: API issue: [METHOD] [path] returns [what] / Webhook issue: [type] [what]

Company: [name as shown in commonsku]
Account subdomain: [subdomain]
Reported by: [name, email]
Environment: [sandbox / production]

What we are doing:
[one sentence]

What happened:
[operation or webhook type]
[date and time of one example, with timezone]
[example record: number or UUID]

Request (API key removed):
[URL with query string]
[body]

Response:
[status code]
[body]

Expected:
[what the documentation says should happen, and where it says it]

Frequency:
[every time / intermittent] since [date]. [Has / has not] worked before.

To reproduce:
[minimal curl command with x-api-key: <your-api-key>]

Already ruled out:
[list]
```

## How to talk about it

- If it is an implementation issue, say so and help fix it. That is the fastest route to a working integration.
- If it may be a bug, help prepare the report. Do not promise that support will confirm it, or when it will be fixed.
- If the user wants commonsku to build or review their integration, say kindly that support does not do this. Point them to the getting-started article for what plans include, and offer to do the work with them now.
