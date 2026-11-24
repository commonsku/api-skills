# Endpoint map

A snapshot of the live spec from 2026-11-18, with `PUT /projects/{id}` added on 2026-11-24. The live spec wins. The API changes, so confirm anything you rely on against the live spec, downloaded as the `commonsku-api-setup` skill describes.

## What you can read and write, by object

"Read" means list and get. Where the table shows no create, update or delete for an object, the spec has no operation for it.

| Object | Read | Create | Update | Delete | Notes |
| --- | --- | --- | --- | --- | --- |
| Clients | Yes | Yes | Yes | Yes | `customer_number` and `credit_hold` are settable |
| Client addresses | Yes | Yes | Yes | Yes | Per client, or all clients' addresses in one list |
| Client contacts | Yes | Yes | Yes | Yes | Per client, or all clients' contacts in one list |
| Projects | Yes | Yes | Yes | No | Created at a chosen starting stage, with optional billing and shipping address. Update covers the name, in-hands date, budget, event type, order rep and client rep. The in-hands date, budget, event type and reps also apply to every order in the project. Changing the client rep needs the `MODIFY-CLIENT-REP` capability. Notes can be added. A project overview feed can be read |
| Opportunities, Presentations, Estimates | Yes | Through project creation or an upgrade | No | No | `GET /orders` needs `order_type`. `POST /orders/upgrade` moves an order to a later stage |
| Sales orders | Yes | Through project creation, an upgrade, or a connected shop checkout | Yes | No | Update covers status, dates, addresses, contacts, terms, currency, tax, `customer_po` and a few form fields |
| Invoices | Yes | By upgrading a sales order | Status only | No | The only settable field is `status_name`, and its only value is `Paid` |
| Purchase orders | Yes | No | Shipping info only | No | Ship dates, shipping cost and overage, tracking ID |
| Bills | Yes | Yes | No | No | Created against purchase order items. Call `GET /bills/prepare` first and send back its payload with the blanks filled in |
| Order items | Get one | Yes | Yes | No | Related operations cover item costs, item locations, item decorations, breakdowns and tax amounts. Item costs and breakdowns can be deleted |
| Artwork and files | Artwork only | Yes | Yes | No | Files are uploaded and linked to a parent record |
| Products | Yes | Yes | Yes | No | Related operations cover costs, colours, sizes, imprints and custom fields. Search is a separate operation |
| Catalogs | Yes | Yes | No | Items only | Items can be listed, added and removed. A CSV export can be generated |
| Imprint methods | Yes | Yes | Yes | No | |
| Shops | Yes | Yes | Yes | No | Includes connected shops, connected shop checkout, and reading checkouts |
| Tags | Yes | Yes | Yes | No | |
| Companies | Yes | No | No | No | Lists clients or suppliers by `company_type`. This is how suppliers are looked up |
| Accounts | Get one | Yes | Yes | No | |
| Reference lists | Yes | No | No | No | Industries, payment terms, event types, users, folders |

Two points catch people out:

- **No operation creates a purchase order.** Sales orders and invoices have no create operation of their own either. They come from creating a project at a stage, upgrading an order, or a connected shop checkout.
- **A field can appear in a GET response and still not be settable.** The update operations above accept fewer fields than the objects they return. Check the request body.

## Every operation

"Spec label" shows `Beta Feature` where the spec lists it. The `Beta Feature` label itself does not limit access: a Beta endpoint works with your key like any other. It marks recently released endpoints that may still change, so retest after updates.

| Area | Method | Path | Summary | Spec label |
| --- | --- | --- | --- | --- |
| account | POST | `/account` | Create an account | |
| account | GET | `/account/{id}` | Retrieve an account profile | |
| account | PUT | `/account/{id}` | Update an account | |
| address | GET | `/addresses/client` | list all client addresses | |
| address | DELETE | `/addresses/{id}` | delete address | |
| address | PUT | `/addresses/{id}` | update address | |
| artwork | POST | `/artwork` | post artwork | Beta Feature |
| artwork | GET | `/artwork/{id}` | get artwork | Beta Feature |
| artwork | PUT | `/artwork/{id}` | update artwork | Beta Feature |
| bill | GET | `/bills` | list bills | |
| bill | POST | `/bills` | create bill | |
| bill | GET | `/bills/prepare` | prepare bill | |
| bill | GET | `/bills/{id}` | get bill | |
| breakdown | POST | `/breakdown` | create breakdown | Beta Feature |
| breakdown | DELETE | `/breakdown/{id}` | delete breakdown | Beta Feature |
| breakdown | PUT | `/breakdowns/{id}` | update item breakdown | Beta Feature |
| catalogs | GET | `/catalogs` | list catalogs | |
| catalogs | POST | `/catalogs` | Create or retrieve a catalog | |
| catalogs | POST | `/catalogs/{id}/generate-csv` | Generate a CSV export of catalog items | |
| catalogs | DELETE | `/catalogs/{id}/items` | Delete a single catalog item | |
| catalogs | GET | `/catalogs/{id}/items` | List items in a catalog | |
| catalogs | POST | `/catalogs/{id}/items` | Add items to a catalog | |
| client | GET | `/clients` | list clients | |
| client | POST | `/clients` | create client | |
| client | DELETE | `/clients/{id}` | delete a client | |
| client | GET | `/clients/{id}` | get client | |
| client | PUT | `/clients/{id}` | update client | |
| client | GET | `/clients/{id}/addresses` | list client addresses | |
| client | POST | `/clients/{id}/addresses` | create client address | |
| client | GET | `/clients/{id}/contacts` | list client contacts | |
| client | POST | `/clients/{id}/contacts` | create client contact | |
| company | GET | `/company` | list companies | Beta Feature |
| company | GET | `/company/get` | get company | Beta Feature |
| contact | GET | `/contacts/client` | list all client contacts | |
| contact | DELETE | `/contacts/{id}` | delete contact | |
| contact | PUT | `/contacts/{id}` | update contact | |
| event_type | GET | `/event_types` | list event types | |
| file | POST | `/file` | upload file | |
| file | PUT | `/file/{id}` | update file | |
| folders | GET | `/folder` | list folders | |
| folders | GET | `/folder/{id}` | get folder | |
| imprint | GET | `/imprints` | list imprints | Beta Feature |
| imprint | POST | `/imprints` | create imprint | Beta Feature |
| imprint | PUT | `/imprints/{id}` | update imprint | Beta Feature |
| industry | GET | `/industries` | list industries | |
| item | PUT | `/item-decorations/{id}` | update item decoration | |
| item | GET | `/item-location/{id}` | get item location | Beta Feature |
| item | POST | `/item-locations` | create item location | Beta Feature |
| item | PUT | `/item-locations/{id}` | update item location | Beta Feature |
| item | GET | `/item/{id}` | get item | Beta Feature |
| item | PUT | `/item/{id}` | update item | Beta Feature |
| item | POST | `/items/product` | create product item | |
| item cost | POST | `/item_costs` | create item cost | |
| item cost | DELETE | `/item_costs/{id}` | delete item cost | |
| item cost | GET | `/item_costs/{id}` | get item cost | |
| item cost | PUT | `/item_costs/{id}` | update item cost | |
| order | GET | `/invoices` | list invoices | |
| order | GET | `/invoices/{id}` | get invoice | |
| order | PUT | `/invoices/{id}` | update invoice | |
| order | GET | `/orders` | list orders | |
| order | POST | `/orders/upgrade` | upgrade order | |
| order | GET | `/orders/{id}` | get order | |
| order | GET | `/sales_orders` | list sales orders | |
| order | GET | `/sales_orders/{id}` | get sales order | |
| order | PUT | `/sales_orders/{id}` | update sales order | |
| product | GET | `/products` | list products | Beta Feature |
| product | POST | `/products` | post product | Beta Feature |
| product | GET | `/products/{id}` | get product | Beta Feature |
| product | PUT | `/products/{id}` | put product | Beta Feature |
| product-color | GET | `/product-colors` | list product colors | Beta Feature |
| product-color | POST | `/product-colors` | add product color | Beta Feature |
| product-color | PUT | `/product-colors` | set product color image | Beta Feature |
| product-cost | GET | `/product-costs` | list product costs | Beta Feature |
| product-cost | POST | `/product-costs` | create or update product cost | Beta Feature |
| product-custom-field | POST | `/product-custom-fields` | create product custom field | Beta Feature |
| product-custom-field | PUT | `/product-custom-fields/{id}` | update product custom field | Beta Feature |
| product-custom-field-value | GET | `/product-custom-field-values` | list product custom field values | Beta Feature |
| product-custom-field-value | POST | `/product-custom-field-values` | create product custom field value | Beta Feature |
| product-custom-field-value | GET | `/product-custom-field-values/{id}` | get product custom field value | Beta Feature |
| product-imprint | GET | `/product-imprints` | list product imprints | Beta Feature |
| product-imprint | POST | `/product-imprints` | add product imprint | Beta Feature |
| product-size | GET | `/product-sizes` | list product sizes | Beta Feature |
| product-size | POST | `/product-sizes` | add product size | Beta Feature |
| products | GET | `/product/search` | search products | Beta Feature |
| project | GET | `/projects` | list projects | |
| project | POST | `/projects` | create project | |
| project | GET | `/projects/{id}` | get project | |
| project | PUT | `/projects/{id}` | update project | |
| project | POST | `/projects/{id}/notes` | create project note | |
| project | GET | `/projects/{id}/overview` | get project overview feed | |
| purchase order | GET | `/purchase_orders` | list purchase orders | |
| purchase order | GET | `/purchase_orders/{id}` | get purchase order | |
| purchase order | PUT | `/purchase_orders/{id}/shipping` | update purchase order shipping info | |
| shop | GET | `/checkout/{id}` | get checkout | Beta Feature |
| shop | GET | `/shop` | list shops | |
| shop | POST | `/shop` | create shop | Beta Feature |
| shop | GET | `/shop/{id}` | get shop | Beta Feature |
| shop | PUT | `/shop/{id}` | update shop | Beta Feature |
| shop | GET | `/shop/{id}/checkout` | list shop checkouts | Beta Feature |
| shop | POST | `/shop/{id}/connected-checkout` | submit connected shop checkout | Beta Feature |
| shop | POST | `/shops/connected` | create connected shop | Beta Feature |
| tag | GET | `/tags` | list tags | Beta Feature |
| tag | POST | `/tags` | create tag | Beta Feature |
| tag | PUT | `/tags/{id}` | update tag | Beta Feature |
| tax-amount | POST | `/tax-amount` | create tax amount | Beta Feature |
| tax-amount | PUT | `/tax-amount/{id}` | update tax amount | Beta Feature |
| terms | GET | `/terms` | list terms | |
| user | GET | `/users` | list users | |
