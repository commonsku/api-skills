---
name: commonsku-api-setup
description: "Set up and make calls to the commonsku REST API: the API key, base URL, sandbox, live spec, rate limits, paging, dates and status codes. Use this before the first call to the commonsku API in a session, and whenever someone writes or reviews code that calls it, asks for an API key, hits a 429, or asks whether a commonsku field, endpoint or filter exists or can be set."
---

# Setting up commonsku API calls

## What this skill helps you do

This skill holds the rules for every call to the commonsku REST API: how to authenticate, where the base URL and spec come from, how to stay inside the rate limits, and how to page.

It needs:

- Network access to `platform.commonsku.com`, or the sandbox address, for API calls.
- Network access to `api.commonsku.com`, to read the live spec.

## Rules for every answer

- Treat the live spec as the only authority for the REST API. Fetch it in each session before you answer anything specific. Do not rely on training data, an earlier conversation or a reference snapshot.
- If you cannot reach the spec, say so. Ask the user to download the JSON and share it. Mark anything taken from a snapshot as unconfirmed until then.
- Webhooks are not in the spec. Use the `commonsku-api-webhooks` skill for them.
- Do not invent. If an endpoint, field, filter or webhook event is not in the live spec or the webhook reference, say it is not documented, and stop.
- Do not speak for commonsku. Make no roadmap claims, give no dates, say nothing about what a plan includes or costs, and promise no fix. Say only what the documentation says today.

## Auth header

Send the API key in an `x-api-key` header on every request.

```sh
curl -sS "$COMMONSKU_BASE_URL/clients?page_limit=5" \
  -H "x-api-key: <your-api-key>" \
  -H "Accept: application/json"
```

commonsku's team issues API keys. There is no screen in commonsku where a customer creates one. To request API access, or a key that has not arrived, the user emails `support@commonsku.com`.

Keep the key in an environment variable or a secret manager. Never print it, log it or commit it.

## Base URL and sandbox

| Environment | Base URL |
| --- | --- |
| Production | `https://platform.commonsku.com/v1` |
| Sandbox | `https://platform.01.commonskusandbox.com/v1`, the same for every account |

Set `COMMONSKU_BASE_URL` to the production or sandbox URL. The examples in these skills use it.

The base URL is `servers[0].url` in the spec you downloaded. The table shows today's values. If they ever differ, the spec wins.

Older integrations may use a different server address. It still works for now but will be retired. Move existing integrations to these base URLs.

Production is the live commonsku account. Writes there change real clients, orders and invoices.

A sandbox may be available, depending on the account. The user asks `support@commonsku.com`. A sandbox is a separate environment with its own key, and its data never syncs with production.

The sandbox spec is at `https://api.01.commonskusandbox.com/public_api_doc.php?format=json`. The sandbox spec can differ from production, especially for newly released operations. When working in sandbox, read the sandbox spec.

Unless the user says they are working in a sandbox, treat every POST, PUT and DELETE as a change to live data. Show the exact request and get a go-ahead before you send a write.

## Where the docs live

The API reference is at `https://api.commonsku.com/public_api_doc.php`. It is a JavaScript viewer, so a plain fetch of it returns an empty page. Use the spec instead, as the next section says.

Access, keys and FAQs are in commonsku's getting-started article: `https://help.commonsku.com/knowledge/commonsku-api-getting-started`

Webhooks are not in the spec.

## Download the full spec

Download the OpenAPI spec with `curl`. Do not use a web-fetch tool, because web-fetch tools cut the spec short without warning. Request the docs page from the section above with `?format=json` added to its address, and save the response to a file.

Before you call an endpoint, look up its current path, parameters, and schema in that file. Do not guess endpoint names.

## Check what commonsku exposes

Look up operations in the spec file you downloaded. The spec is about 1 MB, so search the file rather than reading it whole into context.

- For one operation, read its parameters and its request body.
- For one field, check where it appears: as a parameter, in a request body, or only in responses.

Read `references/api-basics.md` for keys, rate limits, paging, filters, dates and status codes.

## Rate limits and backoff

- The limits are 2 requests a second, with bursts of up to 4, and 20,000 requests a day, per API key.
- Some accounts have a higher daily limit. The user asks `support@commonsku.com`.
- Each account has one key. Every integration on the account shares the same allowance.
- The daily count resets at 11:59 PM Eastern time.
- A request over a limit returns `429 Too Many Requests`. The response does not say how long to wait.
- On a 429, wait, then double the wait after each further 429: for example 1, 2, 4 and 8 seconds, then 1 minute. Set a cap.
- Never retry at once in a loop.
- If 429s continue for a long time, the daily allowance is probably used up. Stop, and resume after the reset.
- Send requests one at a time. Pace requests at no more than 2 a second. Never send them in parallel loops.

## Paging

List operations use one of two styles. Check the operation's parameters in the spec to see which one it uses.

- **Cursor style**, used by most list operations: set `page_limit` on every request, no higher than the documented maximum of 250. Pass the returned `meta.cursor` back as `cursor` until no further cursor comes back.
- **Offset style**, used by `GET /products`, `GET /imprints` and `GET /company`: `start-index` is a zero-based offset and `max-results` is the page size.

The spec does not describe the last page. Confirm the stopping condition on a short list before you run a large sync.

Never fetch everything. Where an operation supports it, filter with `date_updated__gt` to fetch only what changed since the last run.

## Gotchas

- A field in a GET response is not always settable. It is settable only if it is also in the request body of a POST or PUT operation. Check the request body before you say a field can be set.
- The `Beta Feature` label itself does not limit access: a Beta endpoint works with your key like any other. It marks recently released endpoints that may still change, so retest after updates.
- If reads work but creates or updates fail with a permission error, it is usually an account set-up issue on commonsku's side, not your key or code. Email `support@commonsku.com` with the endpoint, request time and response body, say reads work but writes don't, and never include the API key.
- An unset date in a response is the string `0000-00-00 00:00:00`. Most date parsers reject it. Test for it and treat it as empty.

## What the API cannot do

- **Create an API key.** commonsku's team issues keys. The user emails `support@commonsku.com`.

## Reference files

| File | Read it when |
| --- | --- |
| `references/api-basics.md` | You set up calls: docs, base URL, keys, rate limits, sandbox, paging, filters, dates, status codes |

Last verified: 2026-11-18
