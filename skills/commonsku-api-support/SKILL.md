---
name: commonsku-api-support
description: "Triage commonsku API and webhook failures, tell an implementation problem apart from a platform bug, and prepare a bug report for commonsku support. Use this when a commonsku API call or webhook does not behave as expected, or when someone asks what commonsku support covers or how to report a bug."
---

# When something goes wrong

## What this skill helps you do

This skill helps a developer work out why a commonsku API call or webhook failed, and decide whether to fix their own code or report a bug to commonsku support. For how to make a call, use the `commonsku-api-setup` skill.

## Who does what

Tell the user these three points once, early, when they plan a build. Do not repeat them in every answer.

- API and webhook access is a self-serve developer tool. The user needs development skills on their side: an in-house developer, a contractor, or whoever runs their middleware. commonsku does not build, configure, review or troubleshoot a customer's integration code, field mappings or middleware.
- commonsku technical support helps when the API or a webhook does not behave as documented. `references/support.md` explains how to tell the difference and what to send.
- Some plans include more hands-on integration help. The getting-started article explains it. Do not guess what the user's plan includes.

Then be the implementation help yourself.

## Steps

Go to `references/support.md` and triage before anyone contacts commonsku. Most failures are implementation issues the developer can fix. If the evidence points to a platform bug, use the checklist there to prepare a report.

## Reference files

| File | Read it when |
| --- | --- |
| `references/support.md` | Something fails, or the user asks what commonsku support covers or how to report a bug |

Last verified: 2026-11-18
