---
title: Zero BS CRM API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - PHP

toc_footers:
  - <a href='https://zerobscrm.com/extension-bundles/'>Entrepreneur's Bundle</a>

includes:


search: true
---

# Introduction

Welcome to the Zero BS CRM API v2.0. The API is currently in Beta and we would love your feedback on it. Use the API to talk to Zero BS CRM from another application.

The following table shows the API version present in each major version of Zero BS CRM

API Version    | ZBS Version | WP Version | Documentation
-------------- | ------------| ------------ | -------------
v2             | 2.70+ | 4.0+  | -
v1             | 2.00+ | 4.0+  | [v1 Docs](https://docs.zerobscrm.com/api/)

We are one of the only WordPress CRM's out there that offer a full API with detailed documentation. Tweet to us and say hi [@zerobscrm](https://twitter.com/zerobscrm)

## Requirements

## Errors

<aside class="danger">
Zero BS CRM API has a numnber of error codes which it returns if it detects something wrong.
</aside>

The Zero BS CRM API uses the following error codes:

Error Code | Meaning
---------- | -------
400 | Bad Request -- Your request is invalid.
401 | Unauthorized -- Your API key is wrong.
403 | Forbidden -- The resourcerequested is hidden for administrators only.
404 | Not Found -- The specified record could not be found.
405 | Method Not Allowed -- You tried to access a record with an invalid method.
406 | Not Acceptable -- You requested a format that isn't json.
410 | Gone -- The record requested has been removed from your database
418 | I'm a teapot.
429 | Too Many Requests -- You're requesting too many records! Slow down!
500 | Internal Server Error -- We had a problem with our server. Try again later.
503 | Service Unavailable -- You are temporarily offline for maintenance. Please try again later.

## Parameters

## Pagination

## Ownership / Assignment

## Libraries and Tools

Official Libraries
==================

# Authentication

> The API keys are passed via GET parameters to the request URLs e.g.:

```php
/{RESOURCE}?api_key={your_api_key}&api_secret={your_api_secret}
```


Zero BS CRM uses API keys to allow access to the API. You can learn how to generate your API keys by reading [How to Generate API Keys](https://zerobscrm.com/kb/knowledge-base/zero-bs-crm-api-key-and-api-secrets/)

The API keys give both *Read* and *Write* access to your install of Zero BS CRM. 


# Customers

## Create Customer 

To Create a new Customer in your Zero BS CRM using the API you need to send a JSON post request to the URL. This will create or update a customer. The unique key is their email address. On Sucess the response will return the details of the new Customer added.

### HTTP Request
<br/>
<span class='api-url'><span class='post'>POST</span> /create_customer?api_key={your_api_key}&api_secret={your_api_secret}</span>

```php
<?php
$data = array(
    'status' => 'Lead',    
    'prefix' => 'Mr',
    'fname'  => 'John', 			        
    'lname'  => 'Doe', 			        
    'suffix' => 'MSc',    		        
    'email'  => 'someone@somewhere.com',		

    'hometel' => '1234 567 89',  	
    'worktel' => '1234 567 89',  
    'mobtel'   => '1234 567 89',

    'addr1'    => 'Sample House', 
    'addr2'    => 'Sample Road', 
    'city'     => 'Sample City' 
    'county'   => 'Sample State', 
    'postcode' => 'P0ST C0D3', 
    'country'  => 'UK', 

    'secaddr_addr1'    => 'Sample House 2', 
    'secaddr_addr2'    => 'Sample Road 2',
    'secaddr_city'     => 'Sample City 2',
    'secaddr_county'   => 'Sample State 2', 
    'secaddr_postcode' => 'P1ST C1D3',
    'secaddr_country'  => 'USA', 

    'custom-field' => 'bacon'
    );

print_r($zerobscrm->post('customers', $data));
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
  "addr2": "Sample Town",
  "city": "Sampleton",
  "county": "Samples",
  "postcode": "SAM PL4",
  "hometel": "123 455",
  "worktel": "",
  "mobtel" : "555 135",
  "notes" : "Added from the API",
  "ID" : 135
}
```

## View Customers

```php
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get
```

> The above command returns JSON structured like this:

```json
[
  {
    "id": 1,
    "name": "Fluffums",
    "breed": "calico",
    "fluffiness": 6,
    "cuteness": 7
  },
  {
    "id": 2,
    "name": "Max",
    "breed": "unknown",
    "fluffiness": 5,
    "cuteness": 10
  }
]
```

Returns a list of customers. The meta information returned is the fields for the Customer (same as in create customer). If you have custom fields these are labelled cf1 (key): (value) in the meta element of the JSON response. Visit the URL in a browser window to inspect the response..

### HTTP Request

`POST /zbs_api/customers?api_key={your_api_key}&api_secret={your_api_secret}`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
include_cats | false | If set to true, the result will also include cats.
available | true | If set to false, the result will include kittens that have already been adopted.

<aside class="success">
Remember — a happy kitten is an authenticated kitten!
</aside>

## Get Customer

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get(2)
```

```shell
curl "http://example.com/api/kittens/2"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.get(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "name": "Max",
  "breed": "unknown",
  "fluffiness": 5,
  "cuteness": 10
}
```

This endpoint retrieves a specific kitten.

<aside class="warning">Inside HTML code blocks like this one, you can't use Markdown, so use <code>&lt;code&gt;</code> blocks to denote code.</aside>

### HTTP Request

`GET http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to retrieve

## Delete Customer

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.delete(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.delete(2)
```

```shell
curl "http://example.com/api/kittens/2"
  -X DELETE
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.delete(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "deleted" : ":("
}
```

This endpoint deletes a specific kitten.

### HTTP Request

`DELETE http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to delete

# Quotes

## Create Quote

## View Quotes
 
##  Get Quote

## Delete Quote

# Invoices

## Create Invoice

## View Invoices
 
##  Get Invoice

## Delete Invoice

# Transactions

## Create Transaction

## View Transactions
 
##  Get Transaction

## Delete Transaction

# Events

## Create Event

## View Events
 
##  Get Event

## Delete Event

