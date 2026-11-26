# Purchase order filters and fields

These tables follow the live production spec. Check the spec you downloaded before you rely on them, because it wins if they differ.

## Contents

- `GET /purchase_orders` filters
- Sort and expand
- Purchase order fields
- Shipping update fields
- Responses and errors

## `GET /purchase_orders` filters

| Filter | Type | Notes |
| --- | --- | --- |
| `status_name` | string | The spec does not list the allowed values |
| `customer_number` | string | The client's ID in another system. See the `commonsku-api-setup` skill |
| `purchase_order` | number | Not fully documented. The spec does not say how it differs from `po_number` |
| `po_number` | number | The PO number |
| `so_number` | number | The sales order number |
| `project_number` | number | The project number |
| `po_date_created` | date | Range suffixes `__lt`, `__lte`, `__gt`, `__gte`, such as `po_date_created__lt=2026-01-01` |
| `po_shipping_date` | date | Same range suffixes |
| `po_in_hands_date` | date | Same range suffixes |
| `date_updated` | date-time | Same range suffixes, such as `date_updated__gt=2026-01-01T10:00:00Z` |
| `next_action_type` | string | Exact match on the next action type label |
| `next_action_date` | date | Same range suffixes |
| `tracking_numbers` | string | Contains match on a tracking number |
| `cursor` | string | Paging cursor |
| `page_limit` | number | Items per page |

## Sort and expand

- `sort` sorts ascending. Prefix a value with `-` to sort descending. Allowed values: `po_date_created`, `po_in_hands_date`, `po_shipping_date`, `customer_number`, `purchase_order`, `so_number`, `project_number`, `status_name`, `date_updated`.
- The spec's own sort example, `-date_created`, is not in that list. Use a listed value, such as `-date_updated`.
- `expand[]=client` adds client data. It is the only expand value.

## Purchase order fields

`GET /purchase_orders`, `GET /purchase_orders/{id}` and `PUT /purchase_orders/{id}/shipping` return the same purchase order object. Most fields have no description in the spec. The descriptions below are the spec's.

| Group | Fields |
| --- | --- |
| Identity and status | `purchase_order_uuid`, `purchase_order`, `po_type`, `po_status`, `po_total` (string), `currency`, `po_date_created`, `date_updated`, `notes` |
| Supplier | `vendor`, `vendor_mapping_id`, `vendor_account_uuid`, `vendor_id_number`, `fob_point` |
| Supplier address | `address_uuid`, `address_name`, `address_company`, `address_line_1` to `address_line_4`, `address_city`, `address_state`, `address_postal`, `address_country` |
| People | `production_rep_uuid`, `production_rep`, `order_rep_uuid`, `order_rep`, `client_rep_uuid`, `client_rep` |
| Client and order | `customer_uuid`, `customer_name`, `customer_number`, `parent_customer_name`, `project_number`, `project_name`, `event_type`, `so_number` |
| Shipping (settable) | `date_shipped`, `shipping_cost` (string), `date_shipdatecurrent`, `shipping_overage` (string), `shipping_tracking_id` |
| Shipping (read only) | `po_ship_method`, `po_ship_method_other`, `po_ship_method_details`, `po_ship_date_option`, `po_in_hands_date`, `po_inhands_firm`, `po_shipping_date`, `po_split_shipping_url`, `po_ship_account`, `po_shipping_account_number`, `tracking_numbers` |
| Ship-to | `po_ship_to_company_uuid`, `po_ship_to_company_name`, `po_ship_to_ref_id`, `po_ship_to_address_uuid`, `po_ship_to_address_name`, `po_ship_to_address_company`, `po_ship_to_address_line_1` to `_line_4`, `po_ship_to_address_city`, `po_ship_to_address_state`, `po_ship_to_address_postal`, `po_ship_to_address_country` |
| Next action | `next_action_type`, `next_action_date` |
| Contacts | `billing_contact_name`, `billing_contact_email`, `shipping_contact_name`, `shipping_contact_email` |
| Billing and shipping addresses | `billing_address_*` and `shipping_address_*`: `uuid`, `name`, `company`, `line_1` to `line_4`, `city`, `state`, `postal`, `country` |
| Items | `items[]`, below |

Fields the spec describes:

| Field | Spec description, in short |
| --- | --- |
| `po_inhands_firm` | Whether the in-hands date is marked firm. Present only when the PO has shipping |
| `po_split_shipping_url` | Link to the uploaded split-ship spreadsheet. Present only when the shipping destination is Split Ship |
| `po_ship_account` | The third-party shipping account's name, or `Supplier's Account` when shipping on the supplier's own account |
| `po_ship_to_ref_id` | The supplier's `vendor_mapping_id` when shipping to a supplier, or the client's `customer_number` when shipping to a client |
| `next_action_type` | Label of the next action type. Empty when none is set or the type is inactive |
| `next_action_date` | Next action date as `YYYY-MM-DD HH:MM:SS`. Empty when none is set |
| `tracking_numbers` | One tracking number per shipment on the PO |

### Items

Each entry in `items[]` has `product_code`, `product_name`, `item_name`, `po_item_uuid`, `item_uuid` and `product_image_url`, plus:

- `breakdowns[]`: `item_type`, `quantity`, `unit_cost`, `size_name`, `color_name`, `subtotal`, `external_sku`. Size and colour IDs are not here; the `commonsku-api-breakdowns` skill says where to get them.
- `artworks[]`: `artwork_location_description`, `artwork_logo_name`, `artwork_size`, `artwork_color`, `costs`, and `item_costs[]` with `item_cost_id`, `item_code`, `quantity`, `unit_cost`, `subtotal`, `proof_required`.

## Shipping update fields

`PUT /purchase_orders/{id}/shipping` accepts only these fields. Only the fields in the body change.

| Field | Type |
| --- | --- |
| `date_shipped` | date |
| `shipping_cost` | number |
| `date_shipdatecurrent` | date |
| `shipping_overage` | number |
| `shipping_tracking_id` | string |

The spec does not describe `date_shipdatecurrent` or `shipping_overage`. Not fully documented: confirm their meaning before you write to them.

## Responses and errors

| Operation | Responses in the spec |
| --- | --- |
| `GET /purchase_orders` | `200` with `data` (a list) and `meta` |
| `GET /purchase_orders/{id}` | `200` with `data` and `meta` |
| `PUT /purchase_orders/{id}/shipping` | `200` with `purchase_order`; `400` invalid parameter; `403` missing `MODIFY-PURCHASE-ORDER` capability or tenant mismatch; `404` purchase order not found |

The spec lists no error responses for the two `GET` operations. The general status codes are in the `commonsku-api-setup` skill.
