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

To be able to run the Zero BS CRM API you'll need the following on your WordPress install

* Zero BS CRM v2.70+
* WordPress v4.4+
* Pretty permalinks in Settings >Permalinks so that custom enpoints are supported. **Default Permalinks will not work**
* allow_url_fopen must be enabled on your server

<aside class='info'> Please note that you are not required to install the WP REST API (WP API) plugin.</aside>  

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

The Zero BS CRM API needs certain parameters to be able to return results. These are

* api_key = {your_api_key}
* api_secret = {your_api_secret}

Additionally, some endpoints also accept

* zbs_query = {your_search_query}

## Pagination

## Ownership / Assignment

Zero BS CRM has an ownership and assignment model. This means that you can restrict results to be only for a certain owner (i.e. assigned to a certain CRM team member). To do this you pass either the "assign", "owned" or "owner" (see endpoints for more information)

## Libraries and Tools

Official Libraries
==================

To make your development with Zero BS CRM and the API easier. The following libraries are available

* PHP - [Zero BS CRM PHP API SDK](https://github.com/zero-bs-crm/api-php-sdk)


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
<span class='api-url'><span class='post'>POST</span> /create_customer</span>

### Get Parameters
* api_key = {your_api_key}
* api_secret = {your_api_secret}


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
    'assign' => '1'
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

Returns a list of customers. The meta information returned is the fields for the Customer (same as in create customer). If you have custom fields these are labelled slug (key): (value) in the meta element of the JSON response. Visit the URL in a browser window to inspect the response.

### HTTP Request
<br/>
<span class='api-url'><span class='post'>POST</span> /customers</span>

### Get Parameters
* api_key = {your_api_key}
* api_secret = {your_api_secret}


```php
<?php
$data = array(
    'perpage' => '20',    
    'page' => '1',
    'search' => 'string',
    'transactions' => 1,
    'invoices' => 1,
    'quotes' => 1,
    'owned' => '1'
    );
?>
```

> JSON response example:

```json
{
     "id":20267,
     "created":"2017-04-25 03:33:38",
     "name":"John Doe",
     "filterTot":83,
     "filterPages":5,
     "meta":{
             "zbsc_status":"Lead",
             "status":"Customer",
             "prefix":"",
             "fname":"John",
             "lname":"Doe",
             "cf1":"Custom Field 1",
             "addr1":"",
             "addr2":"",
             "city":"",
             "county":"",
             "postcode":"",
             "country":"",
             "secaddr_addr1":"",
             "secaddr_addr2":"",
             "secaddr_city":"",
             "secaddr_county":"",
             "secaddr_postcode":"",
             "secaddr_country":"",
             "hometel":"",
             "worktel":"",
             "mobtel":"",
             "email":"email@email.com",
             "notes":""
        }
}
```

## Search Customers

Searches the customers based on the query string $_GET['zbs_query']. Use this to Deep Search the meta values for the customers.

### HTTP Request
<br/>
<span class='api-url'><span class='get'>GET</span> /customer_search</span>

### Get Parameters
* api_key = {your_api_key}
* api_secret = {your_api_secret}
* zbs_query = {your_search_query}


> JSON response example:

```json
{
     "id":20267,
     "created":"2017-04-25 03:33:38",
     "name":"John Doe",
     "filterTot":83,
     "filterPages":5,
     "meta":{
             "zbsc_status":"Lead",
             "status":"Customer",
             "prefix":"",
             "fname":"John",
             "lname":"Doe",
             "cf1":"Custom Field 1",
             "addr1":"",
             "addr2":"",
             "city":"",
             "county":"",
             "postcode":"",
             "country":"",
             "secaddr_addr1":"",
             "secaddr_addr2":"",
             "secaddr_city":"",
             "secaddr_county":"",
             "secaddr_postcode":"",
             "secaddr_country":"",
             "hometel":"",
             "worktel":"",
             "mobtel":"",
             "email":"email@email.com",
             "notes":""
        }
}
```

## Delete Customer

Coming soon.

# Quotes

Quotes are what you use in Zero BS CRM to win work. The API for Quotes is in early development.

## Create Quote

Coming soon.

## View Quotes

Retreives a list of the last 20 Quotes generated by the CRM owner. This is useful if you want to display recent quotes.

### HTTP Request
<br/>
<span class='api-url'><span class='get'>GET</span> /quotes</span>

### Get Parameters
* api_key = {your_api_key}
* api_secret = {your_api_secret}


> JSON response example:

```json
 {
    "id":20069,
    "created":"2017-04-14 16:42:50",
    "zbsid":"1",
    "meta":{"name":"New Website",
            "val":500,
            "date":"14.04.2017",
            "notes":""},
    "customerid":"19967"
}
```
 
##  Get Quote

Coming soon.

## Delete Quote

Coming soon.

# Invoices

## Create Invoice

Coming soon.

## View Invoices

Retreives a list of the last 20 Invoices generated by the CRM owner. This is useful if you want to display recent invoices.

### HTTP Request
<br/>
<span class='api-url'><span class='get'>GET</span> /invoices</span>

### Get Parameters
* api_key = {your_api_key}
* api_secret = {your_api_secret}

> JSON response example:

```json
 {
    "id":20069,
    "created":"2017-04-14 16:42:50",
    "zbsid":"1",
    "meta":{"name":"New Website",
            "val":500,
            "date":"14.04.2017",
            "notes":""},
    "customerid":"19967"
}
```
 
##  Get Invoice

Coming Soon.

## Delete Invoice

Coming Soon.

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

