---
title: Jetpack CRM API Reference 

language_tabs: # must be one of https://git.io/vQNgJ
  - PHP

toc_footers:
  - <a href='https://jetpackcrm.com/pricing/' target='_blank' rel='noopener'>Entrepreneur Bundle</a>

includes:


search: true
---

# Introduction

Welcome to the Jetpack CRM API v2.0. The API can be used to talk to Jetpack CRM from another application.

All endpoints live under your CRM site's API root:

`https://{your-site}/zbs_api/{endpoint}`

## Requirements

* Jetpack CRM v6.0+
* WordPress v6.0+
* PHP 7.4+
* Pretty permalinks enabled in **Settings → Permalinks**. Default permalinks will not work because the API uses custom rewrite endpoints.

<aside class='info'>You do not need the WP REST API (WP API) plugin installed.</aside>

## Errors

The Jetpack CRM API returns the following error codes:

Error Code | Meaning
---------- | -------
400 | Bad Request. Your request is invalid or the endpoint is unknown.
401 | Unauthorized. Your API key or secret is wrong.
403 | Forbidden. You do not have permission to access this resource.
405 | Method Not Allowed. You used an HTTP method the endpoint doesn't accept.
418 | I'm a teapot. You tried the `BREW` method.

<aside class="info">Your web server or WordPress may also return standard HTTP errors (404, 500, 503, etc.) outside of the API's own error set.</aside>

## Parameters

Every request must include your API credentials as query parameters (see [Authentication](#authentication)).

### Pagination

List endpoints accept:

* `page`: page number, starting at `1` (default: `1`)
* `perpage`: results per page (default: `10`)
* `order`: sort direction, `ASC` or `DESC` (default: `DESC`)

<aside class='info'>A few endpoints (notably <code>customers</code>) historically read pagination from the JSON request body instead of the query string. Endpoint-specific docs below note where this applies.</aside>

### Ownership / Assignment

Jetpack CRM has an ownership and assignment model: results can be restricted to records owned by, or assigned to, a specific CRM team member. Endpoints that support this accept the `assign`, `owned`, or `owner` parameters (see each endpoint for details).

### Other

* `zbs_query`: search phrase for endpoints that support filtering
* `replace_hyphens_with_underscores_in_json_keys`: set to `1` to rewrite hyphens to underscores in JSON keys (useful for integrations like Zapier that reject hyphenated keys)
* `external_api_name`: string identifier for the calling integration, surfaced in logs/webhooks

# Authentication

> The API key and secret are passed as query parameters on every request:

```php
/zbs_api/{endpoint}?api_key={your_api_key}&api_secret={your_api_secret}
```

Jetpack CRM uses an API key + secret pair to authenticate requests. Generate yours from the CRM admin (see [How to Generate API Keys](https://jetpackcrm.com/kb/knowledge-base/zero-bs-crm-api-key-and-api-secrets/)).

* Publishable key: `jpcrm_pk_...`
* Secret key: `jpcrm_sk_...`

The credentials grant both **read** and **write** access to your CRM, so treat the secret like a password.

<aside class='info'>Each CRM install has a single key/secret pair. Regenerating from the admin invalidates the previous pair.</aside>

# Status

## Get Status

A lightweight health-check endpoint. Use it to verify your API credentials and check the running CRM version before making any other calls.

### HTTP Request

<span class='api-url'><span class='get'>GET</span> /status</span>

Optional query parameters:

* `full`: if present (any value), the response also includes installed core modules and active pro extensions.

> JSON response example:

```json
{
  "success": true,
  "data": {
    "status": "Successful Connection",
    "message": "Your API Connection with Jetpack CRM is functioning correctly.",
    "crm_version": "6.7.2",
    "db_version": "4.3"
  }
}
```

> With `?full=1`:

```json
{
  "success": true,
  "data": {
    "status": "Successful Connection",
    "message": "Your API Connection with Jetpack CRM is functioning correctly.",
    "crm_version": "6.7.2",
    "db_version": "4.3",
    "modules": {
      "mail-campaigns": true,
      "client-portal": true
    },
    "extensions": {
      "stripe": { "name": "Stripe Sync", "version": "2.4.0" }
    }
  }
}
```

<aside class='info'>The response uses the standard WordPress <code>wp_send_json_success</code> envelope, with the payload living under the <code>data</code> key.</aside>

# Customers

## Create Customer

Creates or updates a customer. The unique key is the email address; if a matching contact exists it is updated, otherwise a new one is created. Pass `id` to update a specific contact regardless of email. On success the response echoes the data you sent plus the new `id` (if created).

### HTTP Request

<span class='api-url'><span class='post'>POST</span> /create_customer</span>

Send a JSON body. Common fields:

* `email` (string): customer email. Unique key for create-or-update.
* `id` (int): existing contact ID. If set, updates that record.
* `status` (string): one of `Lead`, `Customer`, `Refused`. Other custom statuses are also accepted.
* `prefix`, `fname`, `lname` (string): name fields.
* `addr1`, `addr2`, `city`, `county`, `postcode`, `country` (string): main address.
* `secaddr1`, `secaddr2`, `seccity`, `seccounty`, `secpostcode`, `seccountry` (string): second address.
* `hometel`, `worktel`, `mobtel` (string): phone numbers.
* `assign` (int): WordPress user ID of the team member to own this contact.
* `tags` (array): list of tag strings to attach.
* `{custom-field-slug}` (mixed): any custom field by its slug.

```php
<?php
$data = array(
    'status'   => 'Lead',
    'prefix'   => 'Mr',
    'fname'    => 'John',
    'lname'    => 'Doe',
    'email'    => 'someone@somewhere.com',

    'hometel'  => '1234 567 89',
    'worktel'  => '1234 567 89',
    'mobtel'   => '1234 567 89',

    'addr1'    => 'Sample House',
    'addr2'    => 'Sample Road',
    'city'     => 'Sample City',
    'county'   => 'Sample State',
    'postcode' => 'P0ST C0D3',
    'country'  => 'UK',

    'secaddr1'    => 'Sample House 2',
    'secaddr2'    => 'Sample Road 2',
    'seccity'     => 'Sample City 2',
    'seccounty'   => 'Sample State 2',
    'secpostcode' => 'P1ST C1D3',
    'seccountry'  => 'USA',

    'tags'         => array( 'newsletter', 'vip' ),
    'assign'       => 1,
    'custom-field' => 'bacon',
);
?>
```

> JSON response example:

```json
{
  "fname": "John",
  "lname": "Doe",
  "email": "johndoe@example.com",
  "status": "Lead",
  "prefix": "Mr",
  "addr1": "1 Sample Road",
  "city": "Sampleton",
  "postcode": "SAM PL4",
  "id": 135
}
```

## View Customers

Returns a paginated list of customers. Each entry includes core fields plus any custom fields keyed by slug. Optionally include related invoices, quotes, transactions, and tags.

### HTTP Request

<span class='api-url'><span class='post'>POST</span> /customers</span>

Send a JSON body. All fields optional:

* `page` (int): page number, default `1`.
* `perpage` (int): results per page, default `10`.
* `search` (string): filter by name/email substring.
* `owned` (int): WP user ID; only return contacts owned by this user.
* `company` (int): only return contacts attached to this company ID.
* `tags` (bool): include tags in each customer object. Default: `false`.
* `invoices`, `quotes`, `transactions` (bool): include related records in each customer object. Default: `false`.

```php
<?php
$data = array(
    'perpage'      => 20,
    'page'         => 1,
    'search'       => 'doe',
    'transactions' => true,
    'invoices'     => true,
    'quotes'       => true,
    'tags'         => true,
    'owned'        => 1,
    'company'      => 42,
);
?>
```

> JSON response example:

```json
[
  {
    "id": 20267,
    "owner": 1,
    "status": "Customer",
    "email": "john@example.com",
    "prefix": "Mr",
    "fname": "John",
    "lname": "Doe",
    "addr1": "1 Sample Road",
    "addr2": "",
    "city": "Sampleton",
    "county": "",
    "country": "UK",
    "postcode": "SAM PL4",
    "hometel": "123 455",
    "worktel": "",
    "mobtel": "555 135",
    "tw": "",
    "li": "",
    "fb": "",
    "created": "2024-04-25 03:33:38",
    "createduts": 1713994418,
    "lastupdated": 1713994418,
    "fullname": "John Doe",
    "name": "John Doe",
    "cf1": "Custom Field 1"
  }
]
```

## Search Customers

Search customers by free-text query against the deep meta (name, address, custom fields, etc.), or look up a single customer directly by email.

### HTTP Request

<span class='api-url'><span class='get'>GET</span> /customer_search</span>

Query parameters:

* `zbs_query` (string): search phrase. Returns matching customers.
* `email` (string): exact email lookup. Returns one customer with related invoices, transactions, and tags attached.
* `page`, `perpage`, `order`: standard [pagination](#pagination).
* `replace_hyphens_with_underscores_in_json_keys` (int): set to `1` to rewrite hyphenated JSON keys to underscores (useful for Zapier, etc.).

```php
<?php
// Free-text search:
// GET /zbs_api/customer_search?api_key=...&api_secret=...&zbs_query=doe

// Direct email lookup (includes financial data):
// GET /zbs_api/customer_search?api_key=...&api_secret=...&email=john@doe.com
?>
```

> JSON response example (`zbs_query` search returns an array):

```json
[
  {
    "id": 20267,
    "owner": 1,
    "status": "Customer",
    "email": "john@example.com",
    "fname": "John",
    "lname": "Doe",
    "fullname": "John Doe",
    "name": "John Doe",
    "created": "2024-04-25 03:33:38"
  }
]
```

> JSON response example (`email` lookup returns a single object with linked records):

```json
{
  "id": 20267,
  "owner": 1,
  "status": "Customer",
  "email": "john@example.com",
  "fname": "John",
  "lname": "Doe",
  "fullname": "John Doe",
  "created": "2024-04-25 03:33:38",
  "invoices": [],
  "transactions": [],
  "tags": []
}
```

# Companies

## Create Company

Creates or updates a company. The unique key is the email address; if a matching company exists it is updated, otherwise a new one is created. Pass `id` to update a specific company regardless of email. On success the response echoes the data you sent plus the new `id` (if created).

### HTTP Request

<span class='api-url'><span class='post'>POST</span> /create_company</span>

Send a JSON body. Common fields:

* `name` (string): company name.
* `email` (string): company email. Unique key for create-or-update.
* `id` (int): existing company ID. If set, updates that record.
* `status` (string): company status. If omitted, the CRM's default status is used.
* `addr1`, `addr2`, `city`, `county`, `postcode`, `country` (string): main address.
* `secaddr1`, `secaddr2`, `seccity`, `seccounty`, `secpostcode`, `seccountry` (string): second address.
* `maintel`, `sectel` (string): phone numbers.
* `tw`, `li`, `fb` (string): Twitter/LinkedIn/Facebook URLs.
* `assign` (int): WordPress user ID of the team member to own this company.
* `tags` (array): list of tag strings to attach.
* `{custom-field-slug}` (mixed): any custom field by its slug.

```php
<?php
$data = array(
    'name'     => 'Acme Inc.',
    'email'    => 'hello@acme.com',
    'status'   => 'Customer',

    'maintel'  => '1234 567 89',
    'sectel'   => '1234 567 90',

    'addr1'    => 'Sample House',
    'addr2'    => 'Sample Road',
    'city'     => 'Sample City',
    'county'   => 'Sample State',
    'postcode' => 'P0ST C0D3',
    'country'  => 'UK',

    'secaddr1'    => 'Sample House 2',
    'secaddr2'    => 'Sample Road 2',
    'seccity'     => 'Sample City 2',
    'seccounty'   => 'Sample State 2',
    'secpostcode' => 'P1ST C1D3',
    'seccountry'  => 'USA',

    'tags'         => array( 'partner', 'priority' ),
    'assign'       => 1,
    'custom-field' => 'bacon',
);
?>
```

> JSON response example:

```json
{
  "name": "Acme Inc.",
  "email": "hello@acme.com",
  "status": "Customer",
  "addr1": "Sample House",
  "city": "Sample City",
  "country": "UK",
  "id": 442
}
```

## View Companies

Returns a paginated list of companies.

### HTTP Request

<span class='api-url'><span class='post'>POST</span> /companies</span>

Send a JSON body. All fields optional:

* `page` (int): page number, default `1`.
* `perpage` (int): results per page, default `10`.
* `search` (string): filter by name/email substring.
* `owned` (int): WP user ID; only return companies owned by this user.
* `invoices`, `quotes`, `transactions` (bool): include related records and totals for each company. Default: `false`.

```php
<?php
$data = array(
    'perpage'      => 20,
    'page'         => 1,
    'search'       => 'acme',
    'invoices'     => true,
    'transactions' => true,
    'owned'        => 1,
);
?>
```

> JSON response example:

```json
[
  {
    "id": 442,
    "owner": 1,
    "status": "Customer",
    "name": "Acme Inc.",
    "email": "hello@acme.com",
    "addr1": "Sample House",
    "addr2": "Sample Road",
    "city": "Sample City",
    "county": "Sample State",
    "country": "UK",
    "postcode": "P0ST C0D3",
    "maintel": "1234 567 89",
    "sectel": "",
    "tw": "",
    "li": "",
    "fb": "",
    "created": 1713994418,
    "created_date": "2024-04-25",
    "lastupdated": 1713994418,
    "invoices_total": "1500.00",
    "invoices_count": 3,
    "transactions_total": "1500.00",
    "transactions_paid_total": "1500.00",
    "total_value": "1500.00"
  }
]
```

# Quotes

## View Quotes

Returns a paginated list of quotes.

### HTTP Request

<span class='api-url'><span class='get'>GET</span> /quotes</span>

Query parameters: standard [pagination](#pagination) (`page`, `perpage`, `order`).

> JSON response example:

```json
[
  {
    "id": 20069,
    "owner": 1,
    "id_override": "",
    "title": "New Website",
    "currency": "USD",
    "value": "500.00",
    "date": 1492185770,
    "date_date": "2024-04-14",
    "template": 3,
    "content": "<p>Quote body HTML...</p>",
    "notes": "",
    "send_attachments": false,
    "hash": "abcd1234",
    "lastviewed": 0,
    "lastviewed_date": false,
    "viewed_count": 0,
    "accepted": 0,
    "accepted_date": false,
    "created": 1492185770,
    "created_date": "2024-04-14",
    "lastupdated": 1492185770,
    "lastupdated_date": "2024-04-14",
    "status": -2
  }
]
```

The `status` field is computed from other fields:

* `-1`: draft (no template applied)
* `-2`: published, not yet accepted
* `1`: accepted

# Invoices

## View Invoices

Returns a paginated list of invoices.

### HTTP Request

<span class='api-url'><span class='get'>GET</span> /invoices</span>

Query parameters: standard [pagination](#pagination) (`page`, `perpage`, `order`).

> JSON response example:

```json
[
  {
    "id": 20069,
    "owner": 1,
    "id_override": "INV-0042",
    "parent": 0,
    "status": "Unpaid",
    "status_label": "Unpaid",
    "hash": "abcd1234",
    "currency": "USD",
    "addressed_from": "Acme Inc.",
    "addressed_to": "John Doe",
    "address_to_objtype": 1,
    "allow_partial": false,
    "allow_tip": false,
    "date": 1713994418,
    "date_date": "2024-04-24",
    "due_date": 1716586418,
    "due_date_date": "2024-05-24",
    "paid_date": 0,
    "net": "500.00",
    "discount": "0.00",
    "discount_type": "percent",
    "shipping": "0.00",
    "tax": "0.00",
    "total": "500.00",
    "hash_viewed_count": 0,
    "portal_viewed_count": 0,
    "created": 1713994418,
    "created_date": "2024-04-24 12:00:00",
    "lastupdated": 1713994418
  }
]
```

# Transactions

## Create Transaction

Creates or updates a transaction. If `email` matches an existing customer the transaction is attached to them; otherwise a new customer is created using the email and optional `fname`. Existing transactions with the same `orderid` are updated rather than duplicated.

### HTTP Request

<span class='api-url'><span class='post'>POST</span> /create_transaction</span>

Send a JSON body. Required:

* `orderid` (string): unique reference. Stored as `ref` on the transaction.
* `status` (string): e.g. `Completed`, `Refunded`, `Failed`. Allowed values come from CRM settings (Transactions tab).
* `total` (string|number): grand total.

Optional:

* `email` (string): customer email. Used to find or create the linked contact.
* `fname` (string): first name, used only when auto-creating a contact.
* `date` (int): Unix timestamp of the transaction. Auto-set if omitted.
* `type` (string): e.g. `Sale`, `Refund`.
* `title`, `desc` (string).
* `currency` (string): ISO code, e.g. `USD`.
* `net`, `fee`, `discount`, `tax`, `shipping`, `shipping_tax` (string|number).
* `{custom-field-slug}` (mixed): any custom field by its slug.

```php
<?php
$data = array(
    'orderid'  => 'ORDER-0042',
    'email'    => 'john@example.com',
    'fname'    => 'John',
    'status'   => 'Completed',
    'total'    => '79.99',
    'net'      => '59.99',
    'tax'      => '0.00',
    'fee'      => '10.00',
    'discount' => '10.00',
    'currency' => 'USD',
    'title'    => 'Your Item Name',
);
?>
```

> JSON response example (request body echoed back with new `id`):

```json
{
  "orderid": "ORDER-0042",
  "email": "john@example.com",
  "fname": "John",
  "status": "Completed",
  "total": "79.99",
  "net": "59.99",
  "tax": "0.00",
  "fee": "10.00",
  "discount": "10.00",
  "currency": "USD",
  "title": "Your Item Name",
  "id": 20286
}
```

## View Transactions

Returns a paginated list of transactions.

### HTTP Request

<span class='api-url'><span class='get'>GET</span> /transactions</span>

Query parameters: standard [pagination](#pagination) (`page`, `perpage`, `order`).

> JSON response example:

```json
[
  {
    "id": 20286,
    "owner": 1,
    "status": "Completed",
    "type": "Sale",
    "status_bool": 1,
    "type_accounting": "credit",
    "ref": "ORDER-0042",
    "origin": "api",
    "parent": 0,
    "title": "My Awesome Order",
    "desc": "",
    "date": 1714237915,
    "date_date": "2024-04-27 16:51:55",
    "currency": "USD",
    "net": "59.99",
    "fee": "10.00",
    "discount": "10.00",
    "tax": "0.00",
    "shipping": "0.00",
    "total": "79.99",
    "date_paid": 1714237915,
    "date_paid_date": "2024-04-27 16:51:55",
    "created": 1714237915,
    "created_date": "2024-04-27 16:51:55",
    "lastupdated": 1714237915
  }
]
```

Notes:

* `status_bool` is `1` if the status is configured to count as a successful transaction, `-1` otherwise. Configure which statuses count under **Settings → Transactions** in the CRM.
* `type_accounting` is a derived classification (`credit` / `debit` / etc.) based on the `type` value.

# Events

## Create Event

Creates or updates an event. Pass `id` to update an existing event.

### HTTP Request

<span class='api-url'><span class='post'>POST</span> /create_event</span>

Send a JSON body. All fields optional:

* `title` (string): event title.
* `customer` (int): contact ID this event is attached to.
* `notes` (string): event description. Stored as `desc` on the record.
* `from` (string): start datetime, format `m/d/Y H:00:00`.
* `to` (string): end datetime, same format.
* `notify` (int): set to `24` to schedule a reminder 24 hours before `from`. Any other value disables notifications.
* `complete` (int): `1` to mark complete, `0` for incomplete.
* `owner` (int): WordPress user ID owning the event. `-1` for no owner.
* `id` (int): existing event ID to update.

```php
<?php
$data = array(
    'title'    => 'Follow-up call',
    'customer' => 44149,
    'notes'    => 'Discuss renewal options',
    'from'     => '02/01/2024 14:00:00',
    'to'       => '02/01/2024 15:00:00',
    'notify'   => 24,
    'complete' => 0,
    'owner'    => 1,
);
?>
```

> JSON response example (request body echoed back with new `id`):

```json
{
  "title": "Follow-up call",
  "customer": 44149,
  "notes": "Discuss renewal options",
  "to": "02/01/2024 15:00:00",
  "from": "02/01/2024 14:00:00",
  "notify": 24,
  "complete": 0,
  "owner": 1,
  "id": 44155
}
```

## View Events

Returns a paginated list of events.

### HTTP Request

<span class='api-url'><span class='get'>GET</span> /events</span>

Query parameters: standard [pagination](#pagination) (`page`, `perpage`, `order`), plus:

* `owner` (int): filter to events owned by this WordPress user ID.

> JSON response example:

```json
[
  {
    "id": 44155,
    "owner": 1,
    "title": "Follow-up call",
    "desc": "Discuss renewal options",
    "start": 1706799600,
    "start_date": "2024-02-01 14:00:00",
    "end": 1706803200,
    "end_date": "2024-02-01 15:00:00",
    "complete": 0,
    "show_on_portal": 0,
    "show_on_cal": 1,
    "created": 1706796000,
    "created_date": "2024-02-01 13:00:00",
    "lastupdated": 1706796000
  }
]
```

Notes:

* On read, `notes` becomes `desc`, and `from`/`to` become `start`/`end` (as Unix timestamps), plus formatted `start_date`/`end_date`.

