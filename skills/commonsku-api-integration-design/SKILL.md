---
name: commonsku-api-integration-design
description: "Design and scope integrations between commonsku and other systems, such as an ERP, accounting, e-commerce, CRM or project tool, directly or through middleware such as Zapier, n8n or Make. Use this when someone plans a commonsku sync, chooses between a webhook and polling, picks match keys, or asks whether commonsku can support an integration."
---

# Designing a commonsku integration

## What this skill helps you do

This skill helps a developer plan a build between commonsku and another system: pin down the job, choose the shape, build it in small steps, and answer whether commonsku can do something. For how to make a call, use the `commonsku-api-setup` skill.

## Steps

### 1. Pin down the job

Ask for whichever of these is missing. Do not assume.

- **The other system**, and a link to its API documentation.
- **The direction**: out of commonsku, into commonsku, or both.
- **The commonsku object**: client, contact, project, sales order, purchase order, invoice, bill, product, shop or another. What is possible differs by object.
- **The trigger**: an event in commonsku, a schedule, or a person clicking a button.

### 2. Choose the shape

Most commonsku integrations run through middleware. It reads from one system, reshapes the data and writes to the other. That is the normal design, not a workaround. `references/integration-patterns.md` covers webhook or poll, match keys and common builds.

Whether a design works also depends on the other system. If you have not read its documentation, say so.

### 3. Build small, then widen

1. Make one GET for a record the user can see in commonsku. This proves the key, base URL and headers.
2. Make one write to a record the user is happy to change, after they approve the exact request.
3. Add paging and loops last.

Make every write and every webhook handler safe to run twice. Retries and duplicate deliveries happen.

## Answer "can commonsku do X?"

Start with one verdict:

- **Documented**: the operation or webhook exists today. Name it.
- **Possible with a custom build**: the commonsku side exists, but the build needs middleware, mapping, or something from the other system that you could not verify.
- **Not documented**: it is not in the live spec or the webhook reference today. Say what you checked.

Then name the operations involved, what the answer depends on, and how confident you are and why.

If the closest documented operation covers a different case from the one asked about, say so before you offer it.

## Reference files

| File | Read it when |
| --- | --- |
| `references/integration-patterns.md` | You design a sync: middleware, webhook or poll, match keys, common builds |

Last verified: 2026-11-18
