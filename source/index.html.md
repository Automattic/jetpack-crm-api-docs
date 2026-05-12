---
title: Jetpack CRM API Reference 

language_tabs: # must be one of https://git.io/vQNgJ
  - PHP

toc_footers:
  - <a href='https://jetpackcrm.com/extension-bundles/'>Entrepreneur's Bundle</a>

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

<aside class='info'>The legacy second-address keys (<code>secaddr_addr1</code>, <code>secaddr_city</code>, ...) are still accepted for backward compatibility.</aside>

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

Returns a paginated list of customers. Each entry includes core fields plus all custom fields, keyed by slug, in the `meta` object. Optionally include related invoices, quotes, transactions, and tags.

### HTTP Request

<span class='api-url'><span class='post'>POST</span> /customers</span>

Send a JSON body. All fields optional:

* `page` (int): page number, default `1`.
* `perpage` (int): results per page, default `10`.
* `search` (string): filter by name/email substring.
* `owned` (int): WP user ID; only return contacts owned by this user.
* `company` (int): only return contacts attached to this company ID.
* `tags` (bool): include tags in each customer object.
* `invoices`, `quotes`, `transactions` (bool): include related records in each customer object.

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

Creates a transaction and assigns the transaction to a Customer if the customer exists (identified by email). Creates a new customer if the email does not exist. POST data must be sent as a JSON post.

### HTTP Request
<br/>
<span class='api-url'><span class='post'>POST</span> /create_transaction</span>

### Get Parameters
* api_key = {your_api_key}
* api_secret = {your_api_secret}

```php
<?php
$data = array(
    'orderid' => 'uniqueid',    
    'email' => 'customers email',    
    'status' => 'completed, etc',
    'total' => 'total of the transaction',
    'item_title' => 'Transaction Title',
    'net'  => 'net value',
    'fee' => 'fee for transaction',
    'discount' => 'value of discount',
    'tax' => 'value of tax',   

    );
?>
```

> JSON response example:

```json
{
  "orderid": "John",
  "email": "Doe",
  "fname": "John",
  "status": "Completed",
  "total": "79.99",
  "item_title": "Your Item Name",
  "net": "59.99",
  "tax": "0.00"
  "fee": "10.00",
  "discount": "10.00",
  "tax_rate": "0"
}
```

## View Transactions
 
Retreives a list of the last 20 Transactions generated by the CRM owner. This is useful if you want to display recent transactions.

### HTTP Request
<br/>
<span class='api-url'><span class='get'>GET</span> /transactions</span>

### Get Parameters
* api_key = {your_api_key}
* api_secret = {your_api_secret}

> JSON response example:

```json
{
     "id":20286,
     "created":"2017-04-27 16:51:55",
     "meta":{
            "status":"succeeded",
            "orderid":"ch_1ADFow40goDHn2SuOM9h5Lsn",
            "customer":"20284",
            "total":69.3,
            "customer_name":"",
            "date":"2017-04-27",
            "currency":"USD",
            "item_title":"My Awesome Order",
            "net":0,
            "tax":0,
            "fee":0,
            "discount":0,
            "tax_rate":0
            },
        "customerid":"20284"
}
```

##  Get Transaction

Coming Soon.

## Delete Transaction

Coming Soon.

# Events

## Create Event

Creates an event (task) in the CRM. POST data must be sent as a JSON post.

### HTTP Request
<br/>
<span class='api-url'><span class='post'>POST</span> /create_event</span>

### Get Parameters
* api_key = {your_api_key}
* api_secret = {your_api_secret}


```php
<?php
$data = array(
				'title' => event title
				'customer' => ID of the customer the event is for (if any)
				'notes' => customer notes string
				'to' => to date, format date('m/d/Y H') . ":00:00";
				'from' => from date, format date('m/d/Y H') . ":00:00";
				'notify' => 0 or 24 (never or 24 hours before)
				'complete' => 0 or 1 (boolean),
				'owner' => who owns the event (-1 for no one)
				'event_id' => 
			);
?>
```

> JSON response example:

```json
{
      "title" : "My Event From the API",
      "customer" : 44149,
      "notes" : "Here are some notes for the event",
      "from" : "2018-02-01 03:30",
      "to" : "2018-02-01 04:30",
      "notify" : 24,
      "owner" : 1
}
```


## View Events

Retreives a list of the last 20 Events  generated by the CRM owner. This is useful if you want to display recent events.
 
### HTTP Request
<br/>
<span class='api-url'><span class='get'>GET</span> /events</span>

### Get Parameters
* api_key = {your_api_key}
* api_secret = {your_api_secret}

> JSON response example:

```json
[
  {
    "id": 44155,
    "created": "2018-02-11 04:37:02",
    "title": "My Event From the API",
    "meta": {
      "title": "My Event From the API",
      "customer": "44149",
      "notes": "Here are some notes for the event",
      "to": "2018-02-11 21:00",
      "from": "2018-02-11 20:00",
      "notify": "1",
      "complete": "-1",
      "owner": "1",
      "event_id": 44155
    },
    "actions": {
      "notify": "1",
      "complete": "-1"
    },
    "customer": false
  },
  {
    "id": 38143,
    "created": "2018-01-19 02:11:15",
    "title": "Demo with Wendy",
    "meta": {
      "from": "2018-01-06 02:00:00",
      "to": "2018-01-06 02:15:00",
      "notes": "",
      "customer": 37792
    },
    "actions": {
      "notify": -1,
      "complete": -1
    },
    "customer": false
  },
  {
    "id": 38137,
    "created": "2018-01-19 02:09:00",
    "title": "Demo with Mark",
    "meta": {
      "from": "2018-01-20 13:00:00",
      "to": "2018-01-20 13:15:00",
      "notes": "Demo with mark for Mail Campaigns",
      "customer": 3085
    },
    "actions": {
      "notify": -1,
      "complete": -1
    },
    "customer": false
  }
]
```


##  Get Event

Coming Soon.

## Delete Event

Coming Soon.

