# Tax amount fields, responses and related calls

Check each operation's current schema in the spec you downloaded. Both operations carry the spec label `Beta Feature`.

## `POST /tax-amount`

Request body, required:

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `parent_type` | string | Yes | Only `ITEM` |
| `parent_id` | UUID | Yes | The `item_id` of the order item to tax |
| `tax_id` | UUID | Yes | A tax that belongs to your company |
| `percent` | number | No | Only with the reserved automation tax ID. Overrides the computed value |
| `amount` | number | No | Only with the reserved automation tax ID. Overrides the computed value |

| Status | Meaning in the spec |
| --- | --- |
| `201` | Created. Returns `tax_amount`, `item` and `order` |
| `400` | `parent_type`, `parent_id` or `tax_id` is missing, or `percent` or `amount` was sent with a non-automation `tax_id` |
| `403` | `tax_id` belongs to another company, or the override was used without the required feature access |
| `404` | Item not found, or `tax_id` does not exist |

## `PUT /tax-amount/{id}`

`{id}` is the `tax_amount_id`. Request body, required:

| Field | Type | Notes |
| --- | --- | --- |
| `tax_id` | UUID | Changing it to the reserved automation tax ID needs the `FEATURE-ADVANCED` or `FEATURE-ENTERPRISE` capability and the `ENABLE-CONNECTED-SHOPS` user flag |

| Status | Meaning in the spec |
| --- | --- |
| `200` | Updated. Returns `tax_amount`, `item`, `order` and `taxes`, the tax now on this tax amount |
| `403` | The tax amount belongs to another company, or the automation tax ID was set without the capability and user flag above |
| `404` | Tax amount not found |

## The `tax_amount` object

| Field | Type |
| --- | --- |
| `tax_amount_id` | UUID |
| `parent_id` | UUID |
| `parent_type` | string: `ITEM`, `BILL_ITEM`, `CREDIT_MEMO_ITEM`, `VENDOR_CREDIT_ITEM`, `CREDIT_MEMO` or `VENDOR_CREDIT` |
| `tax_id` | UUID |
| `percent` | string |
| `amount` | string or number |
| `display_order` | integer |

The same shape appears in `item.tax_amounts[]`.

## Where the API returns tax IDs

There is no taxes endpoint. These responses include a `tax_id`:

- `GET /item/{id}`: `tax_amounts[].tax_id` and `options[].tax_amounts[].tax_id`.
- Item-changing calls such as `PUT /item/{id}`, `POST /item_costs` and `POST /breakdown`: `item.tax_amounts[].tax_id` and `order.tax_id`.
- `PUT /tax-amount/{id}`: `taxes[].tax_id`, with `label`, `percent`, `tax_exempt` and `active`. It holds only the tax on that tax amount, not every tax your company has.

## Related calls

- An order's default tax: `PUT /sales_orders/{id}` takes `tax_id`.
- A client's default tax: see the `commonsku-api-accounts` skill.
- Bill lines: `POST /bills` takes `items[].tax_amounts[].tax_id`. The spec says the percent and amount are calculated from the tax.
- Order items in general: see the endpoint map in the `commonsku-api-concepts` skill.

## Not confirmed

The spec does not answer these. Test in a sandbox, if your account has one, before you rely on them:

- The value of the reserved automation tax ID. The spec does not publish it.
- Which feature access the `POST` override needs. The spec says only "additional feature access". The capability and flag named for `PUT` may apply, but the spec does not say so.
- What happens when `percent` or `amount` is sent with another `tax_id`. The operation description says they are ignored. The field descriptions and the `400` response say the call fails. Do not send them.
