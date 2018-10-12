---
title: Memento Payments API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell: curl

# toc_footers:
#   - <a href='https://www.mementopayments.com'>Memento Payments</a>

# includes:
#   - errors

search: true
---

# Introduction

> API Endpoint

```shell
https://api.mementopayments.com
```

The Memento Payments API is designed around [REST](https://en.wikipedia.org/wiki/Representational_state_transfer).
Authentication is based on a pair of authentication and session tokens.
[JSON](http://www.json.org/) is returned by all API responses, including errors.

All examples in this reference are based on the Memento Payments Cloud endpoint.

# HTTP Header

> Example Header

```shell
Product-ID: e9a91b62-990f-41a8-9045-2a6a1464caa9
Correlation-ID: f058ebd6-02f7-4d3f-942e-904344e8cde5
Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC...
```

You need to specify the following fields in the header for each request:

| Header Field | Type | Description |
| ------------ | ---- | ----------- |
| Product-ID | UUID | This is the ID of the product for which all HTTP requests are made. |
| Correlation-ID | UUID | Each request needs to include a uniquely generated ID.  |
| Authentication | Token | This is the bearer token for the authenticated user. (See Authentication chapter) |

# Authentication

To successfully connect to the platform the user needs an authentication token and a session token. The authentication token is used to create a new session token (initially and when the current one expires) and the session token is used for all operations.

All requests to the API need to be accompanied by an Authorization header:

`Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC...`

## The authentication object

> Example Response

```shell
{
  "id": "8d3f94b0-87d0-497f-810c-9b150d42ed05",
  "status": "approved",
  "token": "wxKj3JV6ET1dXVou77675tMqC...",
  "error_code": "",
  "error_message": "",
  "expires_at": "2017-10-19T17:02:03.181879Z"
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| id | uuid | The unique identifier for the authentication. |
| status | string | The status of the authentication.<br>`pending`<br>`approved`<br>`rejected`|
| token | string | The authentication token. |
| error_code | string | The error key, in case of an error. |
| error_message | string | The error message, in case of an error. | 
| expires_at | time | The time when the authentication token expires, if set. |

## The session object

> Example Response

```shell
{
  "token": "d2LRgT827mEcwXlSoEMztc8If...",
  "created_at": "2017-10-18T17:02:03.181879Z",
  "expires_at": "2017-10-19T17:02:03.181879Z"
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| id | uuid | The unique identifier for the session. |
| token | string | The authentication token. |
| expires_at | time | The time when the session expires, if set. |
| created_at | time | The time when the session was created. |

## Get authentication token

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/tokens" \
  -H "Content-Type: application/json" \
  -d $'{
  "identity": {
      "type": "username",
      "value": "jondough"
  },
  "authenticator": "password",
  "secret": "123456",
  "device": {
      "id": "582a5abb-1335-4794-4855-11e067b8c55e",
      "make": "iPhone",
      "model": "iPhone6,2",
      "os_name": "iOS",
      "os_version": "8.0"
  }
}'
```

> Example Response

```shell
{
  "id": "8d3f94b0-87d0-497f-810c-9b150d42ed05",
  "status": "pending",
  "token": "wxKj3JV6ET1dXVou77675tMqC..."
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| identity.type | string | The name of the identity.`required`<br>`phone`<br>`username` |
| identity.value | string | The value which to look up the user by, e.g. a username. `required` |
| authenticator | string | The name of the authenticator. Can be `password`, `sms` or a custom authenticator. `required` |
| secret | string | The secret required for the authenticator. `required` |
| device | Device | The user device information. `required` |

Post identity type + value (e.g. phone number), type of authentication (e.g. "sms") and device. The response will include an ID and status for lookup.

### HTTP Request

`POST` `/v1/tokens`

## Get authentication token – Step 2

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/tokens/8d3f94b0-87d0-497f-810c-9b150d42ed05/secret" \
  -H "Content-Type: application/json" \
  -d $'{
  "secret": "111111",
  "pin": "1234"
}'
```

> Example Response

```shell
{
  "id": "8d3f94b0-87d0-497f-810c-9b150d42ed05",
  "status": "approved",
  "token": "wxKj3JV6ET1dXVou77675tMqC...",
  "expires_at": "2017-10-19T17:02:03.181879Z"
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| secret | string | The secret required to authenticate. `required` |
| pin | string | The PIN for the user wanting to authenticate. `required` |

Post the secret (e.g. verification code) and PIN (depends on the authenticator type).

### HTTP Request

`POST` `/v1/tokens/{id}/secret`

## Get authentication token status

> Example Request

```shell
curl "https://api.mementopayments.com/v1/tokens/{id}" \
  -H "Content-Type: application/json"
}
```

> Example Response

```shell
{
  "id": "8d3f94b0-87d0-497f-810c-9b150d42ed05",
  "status": "pending"
}
```

### HTTP Request

`GET` `/v1/tokens/{id}`

## Get session token

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/tokens" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

> Example Response

```shell
{
  "token": "d2LRgT827mEcwXlSoEMztc8If...",
  "created_at": "2017-10-18T17:02:03.181879Z",
  "expires_at": "2017-10-19T17:02:03.181879Z"
}
```

Send an authentication token as an Authorization header and receive a session token as well as the date and time when the session expires.

### HTTP Request

`POST` `/v1/sessions`

### Errors

|HTTP Status|Explanation|
|-----------|-----------|
|400 Bad Request|The Authentication Token was found, but the user account is locked or invalid.|
|401 Unauthorized|The Authentication Token was not found or is invalid.|

## Verify session token

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/sessions/verify" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

Verify that a specific session token is valid by sending the token as an Authorization header. Returns 200 OK if the session token is valid.

### HTTP Request

`POST` `/v1/sessions/verify`

### Errors

|HTTP Status|Explanation|
|-----------|-----------|
|401 Unauthorized|The Session Token was not found or has expired.|

# Response Codes & Errors

|Code|Definition|Returns|Remarks|
|----|----------|-------|-------|
|200 |OK|The requested data.||
|201 |Created|The newly created object.||
|400 |Bad Request|An Error object.|General error within the API.|
|401 |Unauthorized|Empty body.||
|404 |Not Found|Empty body.|Wrong URL or current user does not own specific object being manipulated.|
|422 |Unprocessable Entity|A ValidationError object.|Validation errors, missing parameters or incorrect data.|

<!--
TODO: List error codes (eg unknown_exception)
TODO: List validation error codes
-->

## The error object

> Example Response

```shell
{
  "errors": [
    {
      "code": "unknown_exception",
      "message": "An unknown exception occurred."
    }
  ]
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| code | string | The error code. |
| message | string | The translated error message. |

## The validation error object

> Example Response

```shell
{
  "errors": [
    {
      "field": "username",
      "code": "already_taken",
      "message": "Username has already been taken"
    }
  ]
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| field | string | The name of the field that has an error. |
| code | string | The validation error code describing which validation failed. |
| message | string | The translated error message. |

# Filtering

Filtering, pagination and sorting is done with query string parameters.

| Parameter | Description | Format |
| --------- | ----------- | ------ |
| page | Which page to get. | page=1 |
| limit | How many items to get per page. | limit=100 |
| filter | Result filtering. | filter=status:eq:open,name:like:john |
| search | Full text search across multiple fields, where available. | search=john |  
| sort | Which field and direction to sort the results. | sort=created_at:desc,name:asc |

## Operators

| Operator | Description |
| -------- | ----------- |
| eq | Equals |
| gt | Greater than |
| gte | Greater than or equal |
| lt | Less than |
| lte | Less than or equal |
| in | Any of [list] |
| like | Partial text search |

## Example

`GET` `/v1/pools?filter=status:in:[open;closed],created_at:lt:2018-01-01,name:like:john&sort=name:asc`

# Announcements

## The announcement object

> Example Response

```shell
{
  "id": 1,
  "type": "general",
  "title": "Updated Terms of Use",
  "message": "Lorem ipsum dolor sit amet, consectetur adipiscing elit.",
  "button_label": "Open Terms of Use",
  "button_url": "http://www.mementopayments.com/terms",
  "is_dismissible": true,
  "is_fullscreen": false,
  "created_at": "2017-09-04T12:26:43.403883Z",
  "updated_at": "2017-09-04T12:26:43.403883Z"
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| id | uuid | The unique identifier for the announcement. |
| type | string | The type of announcement.<br>`general`|
| title | string | The announcement title. |
| message | string | The announcement message body. |
| button_label | string | If a button is optional or required, this is the button label. |
| button_url | string | If a button is option or required, this is the URL which the button opens. |
| is_dismissible | boolean | Whether the announcement can be dismissed or not. |
| is_fullscreen | boolean | Whether the announcement should be displayed full screen or not. |
| webview_url | string | If a web view should be displayed within the message, this is the URL to render. |
| created_at | time | The time when the announcement was created. |
| updated_at | time | The time when the announcement was updated. |

## Get a list of announcements

> Example Request

```shell
curl "https://api.mementopayments.com/v1/announcements" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```
 
Get a list of all announcements relevant to the user that were sent after the time defined by the `start_at` filter.

### HTTP Request

`GET` `/v1/announcements`

### Filtering

| Attribute | Type | Operators | Values |
| --------- | ---- | --------- | ------ |
| start_at | time | gt | YYYY-MM-DD HH:MM:SS |

## Acknowledge an announcement

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/announcement/{id}/ack" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

Acknowledge an announcement by ID on behalf of the current user.

### HTTP Request

`POST` `/v1/announcements/{id}/ack`

# Contacts

## The contact object

> Example Response

```shell
{
  "id": "d6edcdba-f3f1-4249-96bc-bb977fde27fb",
  "user_id": "fbc521f9-aea8-4da7-840a-1e13ec924b28",
  "first_name": "John",
  "last_name": "Dough",
  "full_name": "John Dough",
  "country": "UK",
  "created_at": "2017-09-04T12:25:52.43349Z",
  "updated_at": "2017-09-04T12:25:52.43349Z"
}
```

A contact can be either a reference to a User or an independent object with a person's name, emails and/or phone numbers.

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| id | uuid | The unique identifier for the announcement. |
| user_id | uuid | The User object representing the contact, if the contact is a registered user. |
| first_name | string | The first name of the contact. |
| last_name | string | The last name of the contact. |
| full_name | string | The full name of the contact. |
| country | string | Two letter ISO 3166-1 alpha-2 country code representing the country the contact is located in. |
| created_at | time | The time when the contact was created. |
| updated_at | time | The time when the contact was updated. |

## Get a list of contacts

> Example Request

```shell
curl "https://api.mementopayments.com/v1/contacts" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

Get a list of all of the user's contacts.

### HTTP Request

`GET` `/v1/contacts`

## Get a contact

> Example Request

```shell
curl "https://api.mementopayments.com/v1/contacts/{id}" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

Get a single contact by ID.

### HTTP Request

`GET` `/v1/contacts/{id}`

## Create a contact

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/contacts" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "user_id": "d6edcdba-f3f1-4249-96bc-bb977fde27fb"
}'
```

Create a new contact by providing a user ID.

### HTTP Request

`POST` `/v1/contacts`

### Create contact

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| user_id | uuid | The unique identifier of the user. `required` |

## Delete a contact

> Example Request

```shell
curl -X DELETE "https://api.mementopayments.com/v1/contacts/{id}" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

Delete an existing contact.

### HTTP Request

`DELETE` `/v1/contacts/{id}`

# Devices

## The device object

> Example Response

```shell
{
  "id": "582a5abb-1335-4794-4855-11e067b8c55e",
  "make": "iPhone",
  "model": "iPhone6,2",
  "os_name": "iOS",
  "os_version": "8.0"
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| id | uuid | The unique identifier for the device. |
| make | string | The device make. |
| model | string | The device model. |
| os_name | string | The name of the OS running on the device. |
| os_version | string | The version of the OS running on the device. |

## Get the current device

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/devices/current" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "id": "582a5abb-1335-4794-4855-11e067b8c55e",
  "make": "iPhone",
  "model": "iPhone6,2",
  "os_name": "iOS",
  "os_version": "8.0",
  "apn_device_token": "{APPLE-PUSH-NOTIFICATION-TOKEN}",
  "gcm_device_token": "{GOOGLE-CLOUD-MESSAGING-TOKEN}"
}'
```

Get the user's current device.

### HTTP Request

`GET` `/v1/devices/current`

## Update the current device

> Example Request

```shell
curl -X PUT "https://api.mementopayments.com/v1/devices/current" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "apn_device_token": "{APPLE-PUSH-NOTIFICATION-TOKEN}",
  "gcm_device_token": "{FIREBASE-CLOUD-MESSAGING-TOKEN}"
}'
```

Update the user's current device.

### HTTP Request

`PUT` `/v1/devices/current`

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| id | uuid | The unique identifier for the device. `required` |
| make | string | The device make. `required` |
| model | string | The device model. `required` |
| os_name | string | The name of the OS running on the device. `required` |
| os_version | string | The version of the OS running on the device. `required` |
| apn_device_token | string | Token for the Apple Push Notification service. |
| gcm_device_token | string | Token for the Firebase Cloud Messaging service. |

# Fees

## Calculate payment fee

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/fees/calculate" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "amount": 10.5,
  "source": {
    "funding_source_id": "d4e07e85-bb7e-485d-b13a-cd6ee18ff599",
    "currency": "EUR"
  },
  "destination": {
    "funding_source_id": "9f5bbafc-cab6-47f4-8489-c8e007ab4288",
    "currency": "USD"
  }
}'
```

Calculate the fee when sending money from one funding source to another.

### HTTP Request

`POST` `/v1/fees/calculate`

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| amount | float | The amount being paid. `required` |
| source | FeeFundingSource | The source from which payment is made and the currency it's made in. `required` |
| destination | FeeFundingSource | The destination to which payment is made and the currency it should be received in. `required` |

### FeeFundingSource

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| funding_source_id | uuid | The unique identifier of the funding source handling the payment. `required` |
| currency | string | Three-letter [ISO currency code](https://www.iso.org/iso-4217-currency-codes.html). Must be a supported currency. `required` |

# Funding Sources

## The funding source object

> Example Response

```shell
{
  "id": "46d679a5-c221-4d91-89ee-da7eff58ed21",
  "type": "bank_account",
  "description": "My funding source",
  "meta": "{\"color\": \"blue\"}",
  "gateway": "bank_of_london",
  "bank_account": {
    "id": "e21dac67-a93f-4681-6572-a6819c747135",
    "country": "IS",
    "account_number": "053526210380",
    "owner_id": "2103805079",
    "enabled": true,
    "created_at": "2017-09-04T12:25:53.085206Z",
    "updated_at": "2017-09-04T12:25:53.085206Z"
  },
  "balances": [
    {
      "id": "a0bcfb20-99fd-465d-6e23-2e19e8952420",
      "amount": 10.0,
      "amount_in": 15.0,
      "amount_out": 5.0,
      "currency": "EUR",
      "created_at": "2017-09-04T12:26:43.398646Z",
      "updated_at": "2017-09-04T12:26:43.398646Z"
    }
  ],
  "currencies": ["EUR"],
  "allows_in": false,
  "allows_out": true,
  "enabled": true,
  "verified": true,
  "verified_at": "2017-09-04T12:26:43.398646Z",
  "created_at": "2017-09-04T12:26:43.398646Z",
  "updated_at": "2017-09-04T12:26:43.398646Z"
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| id | uuid | The unique identifier for the funding source. |
| type | string | The type of funding source.<br>`unclaimed funds`<br>`virtual`<br>`bank_account`<br>`card`<br>`crypto_address`|
| description | string | The title of the funding source, determined by the user. |
| meta | string | A JSON object which can store meta data used by the client. |
| gateway | string | The name of the gateway being used for the funding source type. |
| bank_account | BankAccount | The bank account this funding source connects to. |
| card | Card | The debit or credit card this funding source connects to. |
| crypto_address | CryptoAddress | The address this funding source connects to. |
| balances | array | A list of the current funding source balance per currency. |
| currencies | array | A list of currencies which the funding source can send and receive funds in. |
| allows_in | boolean | Whether funds can be received by the funding source. |
| allows_out | boolean | Whether funds can be sent from the funding source. |
| enabled | boolean | Whether the funding source can receive and/or send funds. |
| verified | boolean | Whether the funding source has been verified by the user. |
| verified_at | time | The time when the funding source was verified, if it has been verified. |
| created_at | time | The time when the funding source was created. |
| updated_at | time | The time when the funding source was updated. |

## The funding source balance object

```shell
{
  "id": "a0bcfb20-99fd-465d-6e23-2e19e8952420",
  "amount": 10.0,
  "amount_in": 15.0,
  "amount_out": 5.0,
  "currency": "EUR",
  "created_at": "2017-09-04T12:26:43.398646Z",
  "updated_at": "2017-09-04T12:26:43.398646Z"
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| id | uuid | The unique identifier for the funding source balance. |
| amount | float | The difference between amounts deposited to and withdrawn from the funding source in the specific currency. |
| amount_in | float | The total amount deposited to this funding source in the specific currency. |
| amount_out | float | The total amount withdrawn from this funding source in the specific currency. |
| currency | string | Three-letter [ISO currency code](https://www.iso.org/iso-4217-currency-codes.html). |
| created_at | time | The time when the funding source balance was created. |
| updated_at | time | The time when the funding source balance was updated. |

## The bank account object

```shell
{
  "id": "e21dac67-a93f-4681-6572-a6819c747135",
  "country": "IS",
  "account_number": "053526210380",
  "owner_id": "2103805079",
  "enabled": true,
  "created_at": "2017-09-04T12:25:53.085206Z",
  "updated_at": "2017-09-04T12:25:53.085206Z"
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| id | uuid | The unique identifier for the bank account. |
| country | string | Two letter ISO 3166-1 alpha-2 country code representing the country the bank account is located in. |
| account_number | string | The unique bank account identifier within the bank where it is hosted. The format depends on the bank. |
| owner_id | string | An optional unique identifier for the person who owns the bank account. The format depends on the bank. |
| enabled | boolean | Whether the bank account is enabled. |
| created_at | time | The time when the bank account was created. |
| updated_at | time | The time when the bank account was updated. |

## The card object
```shell
{
  "id": "335563f9-5249-4339-6d31-078e29fd5f04",
  "status": "active",
  "type": "debit",
  "brand": "MasterCard",
  "owner_id": "2103805079",
  "expiration_month": 10,
  "expiration_year": 2020,
  "masked_number": "1234 56** **** 1234",
  "verified": false,
  "verified_at": "0001-01-01T00:00:00Z",
  "created_at": "2017-09-04T12:25:52.43349Z",
  "updated_at": "2017-09-04T12:25:52.43349Z"
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| id | uuid | The unique identifier for the card. |
| status | string | The card status.<br>`active`<br>`expired`<br>`rejected`<br>`cancelled` |
| type | string | The type of card.<br>`debit`<br>`credit` |
| brand | string | The card brand, e.g. `MasterCard`, `VISA`, etc. |
| owner_id | string | An optional unique identifier for the person who owns the card. The format depends on the card processor. |
| expiration_month | integer | Two digit number representing the card's expiration month. |
| expiration_year | integer | Four digit number representing the card's expiration year. |
| masked_number | string | The masked card number. The format depends on the card processor. |
| verified | boolean | Whether the card has been verified. |
| verified_at | time | The time when the card was verified, if it has been verified. |
| created_at | time | The time when the bank account was created. |
| updated_at | time | The time when the bank account was updated. |

<!-- ## The crypto address object

```shell
```

TODO: Lýsa -->

## Get a list of funding sources

> Example Request

```shell
curl "https://api.mementopayments.com/v1/funding_sources" \
  -H "Content-Type: application/json" 
```

Get a list of all funding sources.

### HTTP Request

`GET` `/v1/funding_sources`

### URL Parameters

|Name|Type|Description|
|----|----|-----------|
|page|int|Item pagination.|
|limit|int|Number of items to return per page.|
|sort|string|Sort the results by `created_at`, `updated_at`.|
|filter|string|Filter the results.|

### Filtering

|Attribute|Type|Operators|Values|
|---------|----|---------|------|
|owner|boolean|eq|true, false|
|status|string|eq, in|pending, enabled, disabled, rejected, all `default: all`|
|type|string|eq, in|bank_account, card, virtual, all `default: all`|

## Get a funding source

> Example Request

```shell
curl "https://api.mementopayments.com/v1/funding_sources/{id}" \
  -H "Content-Type: application/json" 
```

Get a single funding source by ID.

### HTTP Request

`GET` `/v1/funding_sources/{id}`

## Get funding source transactions

> Example Request

```shell
curl "https://api.mementopayments.com/v1/funding_sources/{id}/transactions" \
  -H "Content-Type: application/json" 
```

> Example Response

```shell
[
  {
    "id": "add5c52a-0c57-4d5c-7525-db14566f2f1a",
    "out_user_id": "dd72ebb8-db1f-4442-b203-095ac9ded974",
    "in_user_id": "1c478b12-288a-4ea0-831d-1e36639300da",
    "out_funding_source_id": "d4097613-3b63-4dbb-befe-2211b9dc821a",
    "in_funding_source_id": "b1f6a7de-7a8b-4c3f-a908-a02e16f8e529",
    "payment_id": "745ad357-c7dc-478d-a46b-a97ebd9de4c7",
    "status": "active",
    "amount": 50.0,
    "currency": "EUR",
    "tracking_code": "DEF456",
    "error": false,
    "gateway_response": {
      "id": "79d90419-dc82-4093-6afc-65f8b206fea0",
      "amount": 50.0,
      "currency": "EUR",
      "authorization_code": "1234",
      "reference_code": "ABC123",
      "tracking_code": "DEF456",
      "processor_datetime": "2017-09-04T12:25:53.35114Z",
      "created_at": "2017-09-04T12:25:53.35114Z",
      "updated_at": "2017-09-04T12:25:53.35114Z"
    },
    "expires_at": "2017-10-04T12:25:48.827724Z",
    "created_at": "2017-09-04T12:25:48.827724Z",
    "updated_at": "2017-09-04T12:25:48.827724Z"
  }
]
```

Get a list of transactions for a funding source, both deposits and withdrawals.

### HTTP Request

`GET` `/v1/funding_sources/{id}/transactions`

### URL Parameters

|Name|Type|Description|
|----|----|-----------|
|page|int|Item pagination.|
|limit|int|Number of items to return per page.|
|sort|string|Sort the results by `created_at`, `updated_at`.|
|filter|string|Filter the results.|

### Filtering

|Attribute|Type|Operators|Values|
|---------|----|---------|------|
|currency|string|eq|Three-letter [ISO currency code](https://www.iso.org/iso-4217-currency-codes.html). Must be a supported currency.|
|deposits_only|boolean|eq|true, false|

## Create a funding source

> Example Request (with BankAccount as type)

```shell
curl -X POST "https://api.mementopayments.com/v1/funding_sources" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "description": "My Bank Account",
  "type": "savings_account",
  "bank_account": {
    "country": "UK",
    "swift": "AAABBCCDDD",
    "iban": "GB98MIDL07009312345678",
    "account_number": "",
    "nin": ""
  },
}
```

> Example Request (with Card as type)

```shell
curl -X POST "https://api.mementopayments.com/v1/funding_sources" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "description": "My Default Card",
  "type": "standard_credit_card",
  "card": {
    "brand": "MeCard",
    "last_digits": "1234",
    "expiration_month": 11,
    "expiration_year": 2020,
    "token": "9724017303484431"
  }
}
```

Create a new funding source.

### HTTP Request

`POST` `/v1/funding_sources`

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| description | string | The title of the funding source, only visible to the user. `required` |
| type | string | The name of the funding source type.`required` |
| bank_account | BankAccount | If the type is `bank_account` this object is required. |
| card | Card | If the type is `card` this object is required. |

### BankAccount

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| country | string | Two letter ISO 3166-1 alpha-2 country code representing the country the bank is located in. `required` |
| swift | string | The SWIFT code of the bank. `required` |
| iban | string | The IBAN code of the bank account. This is required if `account_number` is empty. |
| account_number | string | The account number of the bank account, in a format which the bank gateway understands. This is required if `iban` is empty. |
| nin | string | The National Identification Number of the bank account owner. This may be required by the bank gateway. |

### Card

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| brand | string | The name of the card brand. |
| last_digits | string | The last 4 digits of the card number. |
| expiration_month | integer | Two digit number representing the card's expiration month. `required` |
| expiration_year | integer | Four digit number representing the card's expiration year. `required` |
| token | string | The tokenized cardholder data used by the card processor gateway. `required` |

## Update a funding source

> Example Request (with BankAccount as type)

```shell
curl -X PUT "https://api.mementopayments.com/v1/funding_sources/{id}" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "description": "My Updated Bank Account"
}
```

> Example Request (with Card as type)

```shell
curl -X PUT "https://api.mementopayments.com/v1/funding_sources/{id}" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "description": "My Updated Card",
  "card": {
    "expiration_month": 11,
    "expiration_year": 2025
  }
}
```

Update an existing funding source. If the source is a bank account, only its description can be updated. If the source is a card, its description and expiration date can be updated.

### HTTP Request

`PUT` `/v1/funding_sources/{id}`

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| description | string | The title of the funding source, only visible to the user. `required` |

### Card

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| expiration_month | integer | Two digit number representing the card's expiration month. `required` |
| expiration_year | integer | Four digit number representing the card's expiration year. `required` |

## Delete a funding source

> Example Request

```shell
curl -X DELETE "https://api.mementopayments.com/v1/funding_sources/{id}" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

Delete an existing funding source.

### HTTP Request

`DELETE` `/v1/funding_sources/{id}`

## Verify funding source

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/funding_sources/{id}/verify" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "code": "abc123"
}
```

Verify a funding source using a specific code, which can, for example, be sent to the user's card statement.

### HTTP Request

`POST` `/v1/funding_sources/{id}/verify`

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| code | string | The verification code. `required` |

# Images

## The image object

> Example Response

```shell
{
  "id": "75cc21be-fe47-4702-74bc-07b84beed5fb",
  "url": "https://{imagehost}/ui/moments/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
  "full_screen_url": "https://{imagehost}/full/moments/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
  "thumbnail_url": "https://{imagehost}/thumbnail/moments/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
  "created_at": "2017-09-04T12:26:43.403883Z",
  "updated_at": "2017-09-04T12:26:43.403883Z"
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| id | uuid | The unique identifier for the image. |
| url | string | The URL of the UI (cropped) version of the image. |
| full_screen_url | string | The URL of the full screen version of the image. |
| thumbnail_url | string | The URL of the thumbnail version of the image. |
| created_at | time | The time when the image was created. |
| updated_at | time | The time when the image was updated. |

## Get an image

## Upload an image

# Moments

## The moment object

> Example Reponse

```shell
{
  "id": "ad2636c3-82fe-4c45-af2d-d6324b2e618f",
  "status": "open",
  "type": "payment",
  "object_id": "c5d8701e-05cf-4b15-52bf-1cf76c3d84f2",
  "title": "Payment request",
  "note": "Message from user",
  "amount": 10.0,
  "total_amount": 20.0,
  "currency": "EUR",
  "image": {
    "id": "75cc21be-fe47-4702-74bc-07b84beed5fb",
    "url": "https://{imagehost}/ui/moments/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
    "full_screen_url": "https://{imagehost}/full/moments/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
    "thumbnail_url": "https://{imagehost}/thumbnail/moments/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
    "created_at": "2017-09-04T12:26:43.403883Z",
    "updated_at": "2017-09-04T12:26:43.403883Z"
	},
  "participation": {
    "count": {
      "invited": 0,
      "paid": 2,
      "pending": 2,
      "rejected": 1,
      "total": 5
    },
    "first_names": ["Arnar", "Oskar", "Jon"]
  },
  "is_owner": true,
  "read_at": "2015-09-04T12:26:43.48788Z",
  "created_at": "2015-09-04T12:26:43.35539Z",
  "updated_at": "2015-09-04T12:26:43.48788Z"
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| id | uuid | The unique identifier for the moment. |
| status | string | The moment status.<br>`open`<br>`closed`|
| type | string | The moment type. <br>`payment`<br>`request`<br>`pool`|
| object_id | uuid | The unique identifier of the object which the moment refers to. |
| title | string | The title of the moment. |
| note | string | A message accompanying the object which the moment refers to. |
| amount | float | The amount of the moment. This can either be the full amount or partial amount. |
| total_amount | float | The full amount of the moment. |
| currency | string | Three-letter [ISO currency code](https://www.iso.org/iso-4217-currency-codes.html). |
| image | Image | An optional moment image or a group photo of the participants. |
| participation | Participation | Participation information for the moment. |
| is_owner | boolean | Whether the current user is the owner of (i.e. initiated) the moment. |
| read_at | time | The time when the moment was opened and read. Nil if unread. |
| created_at | time | The time when the moment was created. |
| updated_at | time | The time when the moment was updated. |

## Get a list of moments

> Example Request

```shell
curl "https://api.mementopayments.com/v1/moments" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

Get a list of all moments.

### HTTP Request

`GET` `/v1/moments`

### URL Parameters

| Name | Type | Description |
| ---- | ---- | ----------- |
| page | int | Item pagination. |
| limit | int | Number of items to return per page. |
| sort | string | Sort the results by `created_at`, `updated_at`. |
| filter | string | Filter the results. |

### Filtering

| Attribute | Type | Operators | Values |
| --------- | ---- | --------- | ------ |
| open | boolean | eq | true, false |

## Get a moment

> Example Request

```shell
curl "https://api.mementopayments.com/v1/moments/{id}" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

Get a single moment by ID.

### HTTP Request

`GET` `/v1/moments/{id}`

# Money Pools

## The money pool object

> Example Response

```shell
{
  "id": "c5d8701e-05cf-4b15-52bf-1cf76c3d84f2",
  "status": "open",
  "amount": 60.00,
  "currency": "EUR",
  "description": "Money Pool Title",
  "detailed_description": "Money Pool Description",
  "is_public": true,
  "has_unique_participants": true,
  "allows_optional_amount": true,
  "minimum_user_amount": 50.00,
  "maximum_user_amount": 150.00,
  "contribution_options": [
    {
      "id": "a0bcfb20-99fd-465d-6e23-2e19e8952420",
      "title": "Option A",
      "amount": 50.00
    }
  ],
  "image": {
    "id": "75cc21be-fe47-4702-74bc-07b84beed5fb",
    "url": "https://{imagehost}/ui/pools/c5d8701e-05cf-4b15-52bf-1cf76c3d84f2.jpg",
    "full_screen_url": "https://{imagehost}/full/pools/c5d8701e-05cf-4b15-52bf-1cf76c3d84f2.jpg",
    "thumbnail_url": "https://{imagehost}/thumbnail/pools/c5d8701e-05cf-4b15-52bf-1cf76c3d84f2.jpg",
    "created_at": "2017-09-04T12:26:43.403883Z",
    "updated_at": "2017-09-04T12:26:43.403883Z"
	},
  "owner": {
    "id": "add5c52a-0c57-4d5c-7525-db14566f2f1a",
    "first_name": "John",
    "last_name": "Dough",
    "full_name": "John Dough",
    "username": "jondough",
    "country": "UK",
    "timezone": "Europe/London",
    "timezone_utc_offset": 0,
    "verified": true,
    "official": true,
    "image": {
      "id": "75cc21be-fe47-4702-74bc-07b84beed5fb",
      "url": "https://{imagehost}/ui/users/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
      "full_screen_url": "https://{imagehost}/full/users/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
      "thumbnail_url": "https://{imagehost}/users/moments/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
      "created_at": "2017-09-04T12:26:43.403883Z",
      "updated_at": "2017-09-04T12:26:43.403883Z"
    },
    "relationships": [
      {
        "id": "4ec5c820-520d-4668-ba84-0d7bdee23af5",
        "type": "contact",
        "created_at": "2017-04-19T14:35:09.308904Z",
        "updated_at": "2017-04-19T14:35:09.308904Z"
      }
    ]
  },
  "participants": [
    {
      "id": "a0bcfb20-99fd-465d-6e23-2e19e8952420",
      "user_id": "add5c52a-0c57-4d5c-7525-db14566f2f1a",
      "transaction_id": "875ef796-88a1-4c7f-8755-d4cb066b9a3e",
      "status": "paid",
      "amount": 20.00,
      "currency": "EUR",
      "full_name": "Arnar Participant",
      "username": "arnarpart",
      "created_at": "2017-09-04T12:26:43.398646Z",
      "updated_at": "2017-09-04T12:26:43.398646Z"
    }
  ],
  "participation": {
    "count": {
      "invited": 0,
      "paid": 2,
      "pending": 2,
      "rejected": 1,
      "total": 5
    },
    "first_names": ["Arnar", "Oskar", "Jon"]
  },
  "start_at": "0001-01-01T00:00:00Z",
  "end_at": "0001-01-01T00:00:00Z",
  "created_at": "2017-09-04T12:26:43.35539Z",
  "updated_at": "2017-09-04T12:26:43.48788Z"
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| id | uuid | The unique identifier for the money pool. |
| status | string | The money pool status.<br>`open`<br>`closed` |
| amount | float | The total amount collected. |
| currency | string | Three-letter [ISO currency code](https://www.iso.org/iso-4217-currency-codes.html). |
| description | string | The money pool title. |
| detailed_description | string | The money pool description. |
| is_public | boolean | Whether the money pool is publicly available. |
| has_unique_participants | boolean | Whether users can only contribute once to the money pool. |
| allows_optional_amount | boolean | Whether users can pay an optional amount of their choice. |
| minimum_user_amount | float | The lowest amount of a single contribution made to the money pool. |
| maximum_user_amount | float | The highest amount of a single contribution made to the money pool. |
| contribution_options | array | A list of available contribution options. |
| image | Image | An optional money pool image. |
| owner | Owner | The user which created the money pool. |
| participants | array | A list of the participants. Only the 6 most recent will be provided here. For a more detailed list of participants, see [Get money pool partipants](#get-a-list-of-money-pools). |
| participation | Participation | Participation information for the money pool. |
| start_at | time | The time at which the money pool became or will become available. |
| end_at | time | The time at which the money pool became or will become unavailable. |
| created_at | time | The time when the money pool was created. |
| updated_at | time | The time when the money pool was updated. |

## The contribution option object

> Example Response

```shell
{
  "id": "a0bcfb20-99fd-465d-6e23-2e19e8952420",
  "title": "Option A",
  "amount": 50.00
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| id | uuid | The unique identifier for the contribution option. |
| title | string | The option description. |
| amount | float | The payable amount. |

## Get a list of money pools

> Example Request

```shell
curl "https://api.mementopayments.com/v1/pools" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

Get a list of all pools created by the user and pools available to the user but which the user did not create, including public pools and pools the user is invited to or has participated in. This can be specified by using the `owner` filter.

### HTTP Request

`GET` `/v1/pools`

### URL Parameters

| Name | Type | Description |
| ---- | ---- | ----------- |
| page | int | Item pagination. |
| limit | int | Number of items to return per page. |
| sort | string | Sort the results by `created_at`, `updated_at`. |
| filter | string | Filter the results. |
| search | string | Search money pools by description and detailed description. |

### Filtering

| Attribute | Type | Operators | Values |
| --------- | ---- | --------- | ------ |
| owner | boolean | eq | true, false |
| status | string | eq, in | open, closed, all `default: all` |

## Get a money pool

> Example Request

```shell
curl "https://api.mementopayments.com/v1/pools/{id}" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

Get a single money pool by ID.

### HTTP Request

`GET` `/v1/pools/{id}`

## Create a money pool

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/pools" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "description": "Money Pool #1",
  "detailed_description": "This is a more detailed, multiple line decription.",
  "funding_source_id": "d4097613-3b63-4dbb-befe-2211b9dc821a",
  "hashtag": "moneypool1",
  "contribution_options": [
    {
      "title": "Payment title",
      "amount": 10.0
    }
  ],
  "currency": "EUR",
  "invites": [
    "556b6fc6-e8dd-4bfa-89e0-9fbd286c96c3",
    "1d27d1c8-5e58-4d6e-87f7-b6890672294e"
  ],
  "image": {
    "url": "https://upload.wikimedia.org/wikipedia/en/a/a9/Example.jpg"
  },
  "is_public": true,
  "only_owner_sees_participants": true,
  "has_unique_participants": true,
  "allows_optional_amount": true,
  "minimum_user_amount": 50.0,
  "maximum_user_amount": 150.0,
  "start_at": "2017-12-20 16:00:00",
  "end_at": "2017-12-28 23:00:00"
}
```

Create a new money pool.

### HTTP Request

`POST` `/v1/pools`

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| description | string | The money pool title. `required` |
| detailed_description | string | Any description for the money pool. |
| funding_source_id | uuid | The unique identifier of the funding source receiving payment. `required` |
| hashtag | string | An optional hashtag for the money pool. |
| contribution_options | array | A list of contribution options. |
| currency | string | Three-letter [ISO currency code](https://www.iso.org/iso-4217-currency-codes.html). Must be a supported currency. `required` |
| invites | array | A list of users that will be invited to participate in the money pool. |
| image | Image | An optional image object. This can also be performed after creating the money pool. |
| is_public | boolean | Whether everyone can open the money pool or invited users only. Default: `false`. |
| only_owner_sees_participants | boolean | Whether the owner is the only one who can see the list of participants. Default: `false`. |
| has_unique_participants | boolean | Whether users can only contribute once to the money pool. Default: `false` |
| allows_optional_amount | boolean | Whether users can pay an optional amount of their choice. Default: `false` |
| minimum_user_amount | float | The lowest amount of a single contribution made to the money pool. |
| maximum_user_amount | float | The highest amount of a single contribution made to the money pool. |
| start_at | time | The time at which the money pool will become available. |
| end_at | time | The time at which the money pool will become unavailable. |

## Update a money pool

> Example Request

```shell
curl -X PUT "https://api.mementopayments.com/v1/pools/{id}" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "description": "New Title"
}'
```

Update an existing money pool. Anything defined will be updated, otherwise current values will stay unchanged. To remove all contribution options, define contribution_options as an empty array. To leave contribution options unchanged, simply do not define contribution_options in the JSON. Currency can not be changed.

### HTTP Request

`PUT` `/v1/pools/{id}`

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| description | string | The money pool title. |
| detailed_description | string | Any description for the money pool. |
| hashtag | string | An optional hashtag for the money pool. |
| contribution_options | array | A list of contribution options. |
| invites | array | A list of users that will be invited to participate in the money pool. |
| image | Image | An optional image object. This can also be performed after creating the money pool. |
| is_public | boolean | Whether everyone can open the money pool or invited users only. |
| only_owner_sees_participants | boolean | Whether the owner is the only one who can see the list of participants. |
| has_unique_participants | boolean | Whether users can only contribute once to the money pool. |
| allows_optional_amount | boolean | Whether users can pay an optional amount of their choice. |
| minimum_user_amount | float | The lowest amount of a single contribution made to the money pool. |
| maximum_user_amount | float | The highest amount of a single contribution made to the money pool. |
| start_at | time | The time at which the money pool will become available. |
| end_at | time | The time at which the money pool will become unavailable. |

## Close money pool

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/pools/{id}/close" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

Closes a money pool so users cannot contribute anymore. The pool's fulfillment status will become `fulfilled` and its status `closed`.

### HTTP Request

`POST` `/v1/pools/{id}/close`

## Invite users to participate

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/pools/{id}/invite" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "user_ids": [
    "556b6fc6-e8dd-4bfa-89e0-9fbd286c96c3",
    "1d27d1c8-5e58-4d6e-87f7-b6890672294e"
  ],
}'
```

Adds users as participants marked as `invited`.

### HTTP Request

`POST` `/v1/pools/{id}/invite`

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| user_ids | array | An array of unique identifiers of users being invited. `required` |

## Get money pool participants

> Example Request

```shell
curl "https://api.mementopayments.com/v1/pools/{id}/participants" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

> Example Response

```shell
[
  {
    "id": "a0bcfb20-99fd-465d-6e23-2e19e8952420",
    "user_id": "add5c52a-0c57-4d5c-7525-db14566f2f1a",
    "transaction_id": "875ef796-88a1-4c7f-8755-d4cb066b9a3e",
    "status": "paid",
    "amount": 20.00,
    "currency": "EUR",
    "full_name": "John Dough",
    "username": "jondough",
    "created_at": "2017-09-04T12:26:43.398646Z",
    "updated_at": "2017-09-04T12:26:43.398646Z"
  }
]
```

Get a list of participants in a money pool.

### HTTP Request

`GET` `/v1/pools/{id}/participants`

### URL Parameters

|Name|Type|Description|
|----|----|-----------|
|page|int|Item pagination.|
|limit|int|Number of items to return per page.|
|sort|string|Sort the results by `created_at`, `updated_at`.|
|filter|string|Filter the results.|

### Filtering

|Attribute|Type|Operators|Values|
|---------|----|---------|------|
|status|string|eq, in|invited, requested, paid, rejected, all `default: all`|

## Export list of participants

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/pools/{id}/export" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "email": "johndough@example.com",
  "format": "excel"
}'
```

Request a list of participants to be sent to a specific email address.

### HTTP Request

`POST` `/v1/pools/{id}/participants/export`

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| email | string | The email which the exported file should be sent to. `required` |
| format | string | The file format of the exported list. Options: `csv`, `excel`. Default: `csv` |

## Contribute to a money pool

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/pools/{id}/participants" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "amount": 50.0,
  "contribution_option_id": "4dbc5121-a7fa-4cd0-9759-9209ea1ef6b0",
  "funding_source_id": "d4097613-3b63-4dbb-befe-2211b9dc821a",
  "pin": "1234"
}'
```

> Example Response

```shell
{
  "id": "a0bcfb20-99fd-465d-6e23-2e19e8952420",
  "user_id": "add5c52a-0c57-4d5c-7525-db14566f2f1a",
  "transaction_id": "875ef796-88a1-4c7f-8755-d4cb066b9a3e",
  "status": "paid",
  "amount": 50.00,
  "currency": "EUR",
  "full_name": "John Dough",
  "username": "jondough",
  "created_at": "2017-09-04T12:26:43.398646Z",
  "updated_at": "2017-09-04T12:26:43.398646Z"
}
```

The user contributes to the money pool by making a payment. Payment source and PIN is required for payments.

### HTTP Request

`POST` `/v1/pools/{id}/participants`

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| amount | float | The amount being paid. `required` |
| contribution_option_id | uuid | The unique identifier for the contribution option, if selected by the user. |
| funding_source_id | uuid | The unique identifier for the funding source which will be withdrawn from. `required`|
| pin | string | The current user's PIN. `required` |

# Notifications

## The notification object

> Example Response

```shell
{
  "id": "68b206f0-ccea-45df-535f-ed16a97e5530",
  "actor_id": "92e7370f-8ea0-4b84-b412-776c4129a7c7",
  "actor": "Arnar",
  "notification_type": "payment",
  "notification_key": "request",
  "object_id": "74301781-a1eb-41a6-763c-70f235a636b2",
  "object_data_number": 20.00,
  "object_data_string": "",
  "description": "Arnar accepted your friend request",
  "image": {
    "id": "75cc21be-fe47-4702-74bc-07b84beed5fb",
    "url": "https://{imagehost}/ui/payments/c5d8701e-05cf-4b15-52bf-1cf76c3d84f2.jpg",
    "full_screen_url": "https://{imagehost}/full/payments/c5d8701e-05cf-4b15-52bf-1cf76c3d84f2.jpg",
    "thumbnail_url": "https://{imagehost}/thumbnail/payments/c5d8701e-05cf-4b15-52bf-1cf76c3d84f2.jpg",
    "created_at": "2017-09-04T12:26:43.403883Z",
    "updated_at": "2017-09-04T12:26:43.403883Z"
  },
  "created_at": "2015-02-17T23:45:06.872651Z",
  "updated_at": "2015-02-17T23:45:06.872651Z"
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| id | uuid | The unique identifier for the notification. |
| actor_id | uuid | The unique identifier for the acting user. |
| actor | string | The name of the acting user. |
| notification_type | string | The notification type.<br>`payment`<br>`pool`<br>`request` |
| notification_key | string | An optional key which explains the action of the notification type, such as `invited` if the user is invited to a money pool. |
| object_id | uuid | The unique identifier for the object of the specified notification type. |
| object_data_number | float | An optional number related to the notification's object, such as amount for a payment. |
| object_data_string | string | An optional message related to the notifications's object. |
| description | string | The notification message body. |
| image | Image | An optional notification image. |
| created_at | time | The time when the notification was created. |
| updated_at | time | The time when the notification was updated. |

## Get a list of notifications

> Example Request

```shell
curl "https://api.mementopayments.com/v1/notifications" \
  -H "Content-Type: application/json" 
```

Get a list of all notifications.

### HTTP Request

`GET` `/v1/notifications`

### URL Parameters

|Name|Type|Description|
|----|----|-----------|
|page|int|Item pagination.|
|limit|int|Number of items to return per page.|
|sort|string|Sort the results by `created_at`, `updated_at`.|
|filter|string|Filter the results.|
|since|string|Show notifications after a certain date and time. Format is YYYY-MM-DD HH:MM:SS.|

# Participation

Participation information for a moment, request or money pool.

## The participant object

> Example Response

```shell
{
  "id": "a0bcfb20-99fd-465d-6e23-2e19e8952420",
  "user_id": "add5c52a-0c57-4d5c-7525-db14566f2f1a",
  "parent_user_id": "79d94752-f94a-46ab-8793-7f6434025cf7",
  "transaction_id": "875ef796-88a1-4c7f-8755-d4cb066b9a3e",
  "status": "paid",
  "amount": 20.00,
  "currency": "EUR",
  "full_name": "John Dough",
  "username": "jondough",
  "created_at": "2017-09-04T12:26:43.398646Z",
  "updated_at": "2017-09-04T12:26:43.398646Z"
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| id | uuid | The unique identifier for the participant. |
| user_id | uuid | The unique identifier for the participant user. |
| parent_user_id | uuid | The unique identifier for the participant parent user. |
| transaction_id | uuid | The unique identifier for the transaction if a payment has been made. |
| status | string | The status of the participant.<br>`pending`<br>`paid`<br>`settled`<br>`rejected`<br>`cancelled`<br>`invited` |
| amount | float | The amount being paid by or requested of the participant. |
| currency | string | Three-letter [ISO currency code](https://www.iso.org/iso-4217-currency-codes.html). |
| full_name | string | The full name of the participant. |
| username | string | The username of the participant. |
| created_at | time | The time when the participant was created. |
| updated_at | time | The time when the participant was updated. |

## The participation object

> Example Response

```shell
{
  "count": {
    "invited": 0,
    "paid": 2,
    "pending": 2,
    "rejected": 1,
    "total": 5
  },
  "first_names": ["Arnar", "Oskar", "Jon"]
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| count | ParticipationCount | Number of participants. |
| first_names | array | First names of the first 6 participants across all statuses. |

### ParticipationCount

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| invited | integer | The number of participants who have been invited. |
| paid | integer | The number of participants who have paid. |
| pending | integer | The number of participants who have not responded. |
| rejected | integer | The number of participants who have rejected the payment request. |
| total | integer | The total number of participants regardless of status. |

# Payments

## The payment object

> Example Response

```shell
{
  "id": "c5d8701e-05cf-4b15-52bf-1cf76c3d84f2",
  "status": "open",
  "fulfillment_status": "unfulfilled",
  "amount": 20.00,
  "currency": "EUR",
  "description": "This is a payment description",
  "image": {
    "id": "75cc21be-fe47-4702-74bc-07b84beed5fb",
    "url": "https://{imagehost}/ui/payments/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
    "full_screen_url": "https://{imagehost}/full/payments/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
    "thumbnail_url": "https://{imagehost}/thumbnail/payments/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
    "created_at": "2017-09-04T12:26:43.403883Z",
    "updated_at": "2017-09-04T12:26:43.403883Z"
	},
  "owner": {
    "id": "add5c52a-0c57-4d5c-7525-db14566f2f1a",
    "first_name": "John",
    "last_name": "Dough",
    "full_name": "John Dough",
    "username": "jondough",
    "country": "UK",
    "timezone": "Europe/London",
    "timezone_utc_offset": 0,
    "verified": true,
    "official": true,
    "image": {
      "id": "75cc21be-fe47-4702-74bc-07b84beed5fb",
      "url": "https://{imagehost}/ui/users/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
      "full_screen_url": "https://{imagehost}/full/users/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
      "thumbnail_url": "https://{imagehost}/users/moments/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
      "created_at": "2017-09-04T12:26:43.403883Z",
      "updated_at": "2017-09-04T12:26:43.403883Z"
    },
  },
  "recipient": {
    "id": "9c2da03a-5526-457d-b7a4-0e250c46b433",
    "user_id": "baec7eb0-bb93-4ff4-94b0-feb27ad6c2e6",
    "transaction_id": "0baa166e-3130-4420-b30f-99a25829fd99",
    "status": "pending",
    "amount": 20,
    "currency": "EUR",
    "messages": null,
    "username": "",
    "created_at": "2018-08-13T11:52:07.810308Z",
    "updated_at": "2018-08-13T11:52:07.810308Z",
  },
  "expires_at": "2017-09-14T12:26:43.35539Z",
  "created_at": "2017-09-04T12:26:43.35539Z",
  "updated_at": "2017-09-04T12:26:43.48788Z"
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| id | uuid | The unique identifier for the payment. |
| status | string | The payment status.<br>`open`<br>`closed` |
| fulfillment_status | string | The payment fulfillment status.<br>`unfulfilled`<br>`partial`<br>`fulfilled` |
| amount | float | The total amount of the payment. |
| currency | string | Three-letter [ISO currency code](https://www.iso.org/iso-4217-currency-codes.html). |
| description | string | The title of the payment, visible to the owner and recipient. |
| image | Image | An optional request image or a split photo of the owner and recipient. |
| owner | Owner | The User which created the payment. |
| recipient | Participant | The payment recipient. |
| expires_at | time | An unclaimed payment needs to be claimed before this time. |
| created_at | time | The time when the payment was created. |
| updated_at | time | The time when the payment was updated. |

## Get a list of payments

> Example Request

```shell
curl "https://api.mementopayments.com/v1/payments" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

Get a list of all payments created by the user and payments where the user is the recipient. This can be specified by using the `owner` filter.

### HTTP Request

`GET` `/v1/payments`

### URL Parameters

|Name|Type|Description|
|----|----|-----------|
|page|int|Item pagination.|
|limit|int|Number of items to return per page.|
|sort|string|Sort the results by `created_at`, `updated_at`.|
|filter|string|Filter the results.|

### Filtering

| Attribute | Type | Operators | Values |
| --------- | ---- | --------- | ------ |
| owner | boolean | eq | true, false |
| status | string | eq, in | open, closed, all `default: all` |

## Get a payment

> Example Request

```shell
curl "https://api.mementopayments.com/v1/payments/{id}" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

Get a single payment by ID. Transactions are not accessible from this endpoint (see Transactions).

### HTTP Request

`GET` `/v1/payments/{id}`

## Create a payment

> Example Request (send money to another user)

```shell
curl -X POST "https://api.mementopayments.com/v1/payments" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "currency": "EUR",
  "description": "This is a payment description",
  "recipient": {
    "amount": 20.0,
    "user_id": "3fb6e878-58d6-47f6-ba3c-a5089d6e039a"
  },
  "image": {
    "url": "https://upload.wikimedia.org/wikipedia/en/a/a9/Example.jpg"
  },
  "funding_source_id": "f36525d5-39f4-48a9-a547-1887cc69b5cf",
  "pin": "1234"
}
```

> Example Request (send money by phone number)

```shell
curl -X POST "https://api.mementopayments.com/v1/payments" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "currency": "EUR",
  "description": "This is a payment description",
  "recipient": {
    "amount": 20.0,
    "phone": "+44 111 2222 3333",
    "name": "John Dough"
  },
  "image": {
    "url": "https://upload.wikimedia.org/wikipedia/en/a/a9/Example.jpg"
  },
  "funding_source_id": "f36525d5-39f4-48a9-a547-1887cc69b5cf",
  "pin": "1234"
}
```

Create a new payment and send money. Recipient can be based on a user ID or phone number, in which case an optional name can also be sent. Payment source and PIN is required for payments.

### HTTP Request

`POST` `/v1/payments`

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| amount | float | The amount being paid. `required` |
| currency | string | Three-letter [ISO currency code](https://www.iso.org/iso-4217-currency-codes.html). Must be a supported currency. `required` |
| description | string | The payment message. |
| recipient | Participant | The recipient of the funds. `required` |
| image | Image | An optional payment image. |
| funding_source_id | uuid | The unique identifier for the funding source which will be withdrawn from. `required`|
| pin | string | The current user's PIN. `required` |

## Get a receipt

> Example Request

```shell
curl "https://api.mementopayments.com/v1/payments/{id}/receipt" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

> Example Response

```shell
{
  "description": "This is a payment description",
  "reference_code": "00002MC",
  "time": "2017-05-23T12:41:15.813817Z",
  "payer": {
    "id": "add5c52a-0c57-4d5c-7525-db14566f2f1a",
    "title": "John Dough (@johndough)",
    "details": null
  },
  "recipient": {
    "id": "90bcfb20-99fd-465d-6e23-2e19e8952420",
    "title": "Merchant Name",
    "details": [
      "Memento ehf. (700114-0580)",
      "Bolholti 4, 105 Reykjavík"
    ]
  },
  "payment_method": {
    "id": "79d94752-f94a-46ab-8793-7f6434025cf7",
    "title": "Credit Card",
    "details": [
      "MasterCard 1111",
      "Transaction #71574 - AuthCode #18902877"
    ]
  },
  "amounts": {
    "total": "€20.50",
    "paid": "€20.00",
    "cost": "€0.50",
    "currency": "EUR"
  }
}
```

Get a receipt for the payment, if it has been fully processed. The sender and recipient will get different versions of the receipt; the sender will see the payment method while the recipient will not.

### HTTP Request

`GET` `/v1/payments/{id}/receipt`

# Receipt

## The receipt object

> Example Response

```shell
{
  "description": "This is a payment description",
  "reference_code": "00002MC",
  "time": "2017-05-23T12:41:15.813817Z",
  "payer": {
    "id": "add5c52a-0c57-4d5c-7525-db14566f2f1a",
    "title": "John Dough (@johndough)",
    "details": null
  },
  "recipient": {
    "id": "90bcfb20-99fd-465d-6e23-2e19e8952420",
    "title": "Merchant Name",
    "details": [
      "Memento ehf. (700114-0580)",
      "Bolholti 4, 105 Reykjavík"
    ]
  },
  "payment_method": {
    "id": "79d94752-f94a-46ab-8793-7f6434025cf7",
    "title": "Credit Card",
    "details": [
      "MasterCard 1111",
      "Transaction #71574 - AuthCode #18902877"
    ]
  },
  "amounts": {
    "total": "€20.50",
    "paid": "€20.00",
    "cost": "€0.50",
    "currency": "EUR"
  }
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| description | string | The payment title. |
| reference_code | string | A generated reference code. |
| time | time | The time when the payment occurred. |
| payer.id | uuid | The unique identifier for the user sending the payment. |
| payer.title | string | The name of the user sending the payment. |
| payer.details | array | An optional list of strings with details about the user sending the payment, e.g. address. |
| recipient.id | uuid | The unique identifier for the user receiving the payment. |
| recipient.title | string | The name of the user receiving the payment. |
| recipient.details | array | An optional list of strings with details about the user receiving the payment, e.g. address. |
| payment_method.id | uuid | The unique identifier for the payment method. |
| payment_method.title | string | The payment method title. |
| payment_method.details | array | An optional list of strings with details about the payment method, e.g. card information. |
| amount.total | string | The formatted total amount of the payment in the currency in which the payment was made. |
| amount.paid | string | The formatted amount paid in the currency in which the payment was made. |
| amount.cost | string | The formatted cost amount paid in the currency in which the payment was made. |
| amount.currency | string | Three-letter [ISO currency code](https://www.iso.org/iso-4217-currency-codes.html). Must be a supported currency. |

# Requests

## The request object

> Example Response

```shell
{
  "id": "c5d8701e-05cf-4b15-52bf-1cf76c3d84f2",
  "status": "open",
  "fulfillment_status": "partial",
  "amount": 20.00,
  "amount_paid": 10.00,
  "currency": "EUR",
  "description": "Payment request",
  "image": {
    "id": "75cc21be-fe47-4702-74bc-07b84beed5fb",
    "url": "https://{imagehost}/ui/requests/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
    "full_screen_url": "https://{imagehost}/full/requests/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
    "thumbnail_url": "https://{imagehost}/thumbnail/requests/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
    "created_at": "2017-09-04T12:26:43.403883Z",
    "updated_at": "2017-09-04T12:26:43.403883Z"
	},
  "owner": {
    "id": "add5c52a-0c57-4d5c-7525-db14566f2f1a",
    "first_name": "John",
    "last_name": "Dough",
    "full_name": "John Dough",
    "username": "jondough",
    "country": "UK",
    "timezone": "Europe/London",
    "timezone_utc_offset": 0,
    "verified": true,
    "official": true,
    "image": {
      "id": "75cc21be-fe47-4702-74bc-07b84beed5fb",
      "url": "https://{imagehost}/ui/users/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
      "full_screen_url": "https://{imagehost}/full/users/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
      "thumbnail_url": "https://{imagehost}/users/moments/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
      "created_at": "2017-09-04T12:26:43.403883Z",
      "updated_at": "2017-09-04T12:26:43.403883Z"
    },
    "relationships": [
      {
        "id": "4ec5c820-520d-4668-ba84-0d7bdee23af5",
        "type": "contact",
        "created_at": "2017-04-19T14:35:09.308904Z",
        "updated_at": "2017-04-19T14:35:09.308904Z"
      }
    ]
  },
  "participants": [
    {
      "id": "a0bcfb20-99fd-465d-6e23-2e19e8952420",
      "user_id": "add5c52a-0c57-4d5c-7525-db14566f2f1a",
      "transaction_id": "875ef796-88a1-4c7f-8755-d4cb066b9a3e",
      "status": "paid",
      "amount": 10.00,
      "currency": "EUR",
      "full_name": "Arnar Participant",
      "username": "arnarpart",
      "created_at": "2017-09-04T12:26:43.398646Z",
      "updated_at": "2017-09-04T12:26:43.398646Z"
    }
  ],
  "participation": {
    "count": {
      "invited": 0,
      "paid": 2,
      "pending": 2,
      "rejected": 1,
      "total": 5
    },
  },
  "created_at": "2017-09-04T12:26:43.35539Z",
  "updated_at": "2017-09-04T12:26:43.48788Z"
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| id | uuid | The unique identifier for the request. |
| status | string | The request status.<br>`open`<br>`closed` |
| fulfillment_status | string | The request fulfillment status.<br>`unfulfilled`<br>`partial`<br>`fulfilled` |
| amount | float | The total amount of the request. |
| amount_paid | float | The amount that has already been paid. |
| currency | string | Three-letter [ISO currency code](https://www.iso.org/iso-4217-currency-codes.html). |
| description | string | The title of the request, visible to the owner and participants. |
| image | Image | An optional request image or a group photo of the participants. |
| owner | Owner | The User which created the request. |
| participants | array | A list of all participants in the request |
| participation | Participation | Participation information for the request. |
| created_at | time | The time when the request was created. |
| updated_at | time | The time when the request was updated. |

## Get a list of requests

> Example Request

```shell
curl "https://api.mementopayments.com/v1/requests" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

Get a list of all requests created by the user and requests where the user is the recipient. This can be specified by using the `owner` filter.

### HTTP Request

`GET` `/v1/requests`

### URL Parameters

|Name|Type|Description|
|----|----|-----------|
|page|int|Item pagination.|
|limit|int|Number of items to return per page.|
|sort|string|Sort the results by `created_at`, `updated_at`.|
|filter|string|Filter the results.|

### Filtering

| Attribute | Type | Operators | Values |
| --------- | ---- | --------- | ------ |
| owner | boolean | eq | true, false |
| status | string | eq, in | open, closed, all `default: all` |

## Get a request

Get a single request by ID.

## Create a request

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/requests" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "amount": 20.0,
  "currency": "EUR",
  "description": "Payment request",
  "funding_source_id": "d4097613-3b63-4dbb-befe-2211b9dc821a",
  "image": {
    "url": "https://upload.wikimedia.org/wikipedia/en/a/a9/Example.jpg"
  },
  "only_owner_sees_participants": false,
  "participants": [
    {
      "user_id": "3fb6e878-58d6-47f6-ba3c-a5089d6e039a",
      "amount": 10.0
    },
    {
      "phone": "+44 111 2222 3333",
      "name": "John Dough",
      "amount": 10.0
    }
  ]
}'
```

Create a new request.

### HTTP Request

`POST` `/v1/requests`

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| amount | float | The amount being paid. `required` |
| currency | string | Three-letter [ISO currency code](https://www.iso.org/iso-4217-currency-codes.html). Must be a supported currency. `required` |
| description | string | The title of the request, visible to the owner and participants. |
| funding_source_id | uuid | The unique identifier of the funding source receiving payment. `required` |
| image | Image | An optional request image. |
| only_owner_sees_participants | boolean | Whether the owner is the only one who can see the list of participants. Default: `false`. |
| participants | array | A list of participants in the request. `required` |

## Update a request

> Example Request

```shell
curl -X PUT "https://api.mementopayments.com/v1/requests/{id}" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "description": "My Updated Description",
  "image": {
    "url": "https://upload.wikimedia.org/wikipedia/en/a/a9/Example.jpg"
  },
  "only_owner_sees_participants": true
}
```

Update an existing request. Can only update description and image.

### HTTP Request

`PUT` `/v1/requests/{id}`

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| description | string | The title of the request, visible to the owner and participants. |
| image | Image | An optional request image. |

## Delete a request

> Example Request

```shell
curl -X DELETE "https://api.mementopayments.com/v1/requests/{id}" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

Delete an existing request. Can only be performed if none of the participants have responded.

### HTTP Request

`DELETE` `/v1/requests/{id}`

## Remind a participant to pay

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/requests/{id}/participants/{id}/remind" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
}
```

Send a reminder to a request participant in form of a push notification. Note: There is a limit to how many times a participant can be reminded. Exceeding this limit will return in an error message.

### HTTP Request

`POST` `/v1/requests/{id}/participants/{id}/remind`

## Settle a participant

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/requests/{id}/participants/{id}/settle" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
}
```

Mark a participant as paid.

### HTTP Request

`POST` `/v1/requests/{id}/participants/{id}/remind`

## Cancel a participant

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/requests/{id}/participants/{id}/cancel" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
}
```

Cancel the request for a participant.

### HTTP Request

`POST` `/v1/requests/{id}/participants/{id}/cancel`

## Pay a request

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/requests/{id}/pay" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "funding_source_id": "d4097613-3b63-4dbb-befe-2211b9dc821a",
  "pin": "1234"
}'
```

> Example Response

```shell
{
  "id": "a0bcfb20-99fd-465d-6e23-2e19e8952420",
  "user_id": "add5c52a-0c57-4d5c-7525-db14566f2f1a",
  "transaction_id": "875ef796-88a1-4c7f-8755-d4cb066b9a3e",
  "status": "paid",
  "amount": 20.00,
  "currency": "EUR",
  "full_name": "John Dough",
  "username": "jondough",
  "created_at": "2017-09-04T12:26:43.398646Z",
  "updated_at": "2017-09-04T12:26:43.398646Z"
}
```

Pay an existing request as a participant. Payment source and PIN is required for payments.

### HTTP Request

`POST` `/v1/requests/{id}/pay`

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| funding_source_id | uuid | The unique identifier for the funding source which will be withdrawn from. `required`|
| pin | string | The current user's PIN. `required` |

### HTTP Request

`POST` `/v1/requests/{id}/pay`

## Reject a request

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/requests/{id}/reject" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
}
```

Reject an existing request as a participant.

### HTTP Request

`POST` `/v1/requests/{id}/reject`

## Get a receipt

> Example Request

```shell
curl "https://api.mementopayments.com/v1/requests/{id}/receipt" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
}
```

> Example Response

```shell
{
  "description": "This is a payment description",
  "reference_code": "00002MC",
  "time": "2017-05-23T12:41:15.813817Z",
  "payer": {
    "id": "add5c52a-0c57-4d5c-7525-db14566f2f1a",
    "title": "John Dough (@johndough)",
    "details": null
  },
  "recipient": {
    "id": "90bcfb20-99fd-465d-6e23-2e19e8952420",
    "title": "Merchant Name",
    "details": [
      "Memento ehf. (700114-0580)",
      "Bolholti 4, 105 Reykjavík"
    ]
  },
  "payment_method": {
    "id": "79d94752-f94a-46ab-8793-7f6434025cf7",
    "title": "Credit Card",
    "details": [
      "MasterCard 1111",
      "Transaction #71574 - AuthCode #18902877"
    ]
  },
  "amounts": {
    "total": "€20.50",
    "paid": "€20.00",
    "cost": "€0.50",
    "currency": "EUR"
  }
}
```

Get a payment receipt as a participant. The request must be paid, otherwise no receipt will be returned.

### HTTP Request

`GET` `/v1/requests/{id}/receipt`

## Get a receipt for a participant

> Example Request

```shell
curl "https://api.mementopayments.com/v1/requests/{id}/participants/{id}/receipt" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
}
```

> Example Response

```shell
{
  "description": "This is a payment description",
  "reference_code": "00002MC",
  "time": "2017-05-23T12:41:15.813817Z",
  "payer": {
    "id": "add5c52a-0c57-4d5c-7525-db14566f2f1a",
    "title": "John Dough (@johndough)",
    "details": null
  },
  "recipient": {
    "id": "90bcfb20-99fd-465d-6e23-2e19e8952420",
    "title": "Merchant Name",
    "details": [
      "Memento ehf. (700114-0580)",
      "Bolholti 4, 105 Reykjavík"
    ]
  },
  "payment_method": {
    "id": "79d94752-f94a-46ab-8793-7f6434025cf7",
    "title": "Credit Card",
    "details": [
      "MasterCard 1111",
      "Transaction #71574 - AuthCode #18902877"
    ]
  },
  "amounts": {
    "total": "€20.50",
    "paid": "€20.00",
    "cost": "€0.50",
    "currency": "EUR"
  }
}
```

Get a payment receipt for a specific participant in the request. The participant must have paid, otherwise no receipt will be returned. The request owner will not see the payment method.

### HTTP Request

`GET` `/v1/requests/{id}/participants/{id}/receipt`

# Transactions

## The transaction object

> Example Response

```shell
{
  "id": "add5c52a-0c57-4d5c-7525-db14566f2f1a",
  "out_user_id": "dd72ebb8-db1f-4442-b203-095ac9ded974",
  "in_user_id": "1c478b12-288a-4ea0-831d-1e36639300da",
  "out_funding_source_id": "d4097613-3b63-4dbb-befe-2211b9dc821a",
  "in_funding_source_id": "b1f6a7de-7a8b-4c3f-a908-a02e16f8e529",
  "payment_id": "745ad357-c7dc-478d-a46b-a97ebd9de4c7",
  "status": "approved",
  "amount": 50.0,
  "currency": "EUR",
  "tracking_code": "DEF456",
  "error": false,
  "gateway_response": {
    "id": "79d90419-dc82-4093-6afc-65f8b206fea0",
    "amount": 50.0,
    "currency": "EUR",
    "authorization_code": "1234",
    "reference_code": "ABC123",
    "tracking_code": "DEF456",
    "processor_datetime": "2017-09-04T12:25:53.35114Z",
    "created_at": "2017-09-04T12:25:53.35114Z",
    "updated_at": "2017-09-04T12:25:53.35114Z"
  },
  "expires_at": "2017-10-04T12:25:48.827724Z",
  "created_at": "2017-09-04T12:25:48.827724Z",
  "updated_at": "2017-09-04T12:25:48.827724Z"
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| id | uuid | The unique identifier for the moment. |
| out_user_id | uuid | The unique identifier for the user who made the transaction. |
| in_user_id | uuid | The unique identifier for the user who received the transaction. |
| out_funding_source_id | uuid | The unique identifier for the funding source which was withdrawn from. |
| in_funding_source_id | uuid | The unique identifier for the funding source which was deposited to. |
| status | string | The transactions status.<br>`active`<br>`approved`<br>`rejected`<br>`cancelled`<br>`failed` |
| amount | float | The transaction amount. |
| currency | string | Three-letter [ISO currency code](https://www.iso.org/iso-4217-currency-codes.html). |
| tracking_code | string | An optional tracking number which can be used as a reference for other systems. |
| error | boolean | Whether the transaction resulted in an error. |
| gateway_response | GatewayResponse | If the transaction was processed by the gateway, this is the response object. |
| expires_at | time | The time when the transaction expires, if set. |
| created_at | time | The time when the transaction was created. |
| updated_at | time | The time when the transaction was updated. |

## The gateway response object

> Example Response

```shell
{
  "id": "79d90419-dc82-4093-6afc-65f8b206fea0",
  "amount": 50.0,
  "currency": "EUR",
  "authorization_code": "1234",
  "reference_code": "ABC123",
  "tracking_code": "DEF456",
  "processor_datetime": "2017-09-04T12:25:53.35114Z",
  "created_at": "2017-09-04T12:25:53.35114Z",
  "updated_at": "2017-09-04T12:25:53.35114Z"
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| id | uuid | The unique identifier for the gateway response. |
| amount | float | The amount that was processed. |
| currency | string | Three-letter [ISO currency code](https://www.iso.org/iso-4217-currency-codes.html). |
| authorization_code | string | An optional authorization code received from the gateway. |
| reference_code | string | An optional reference code received from the gateway. |
| tracking_code | string | An optional tracking number received from the gateway. |
| processor_datetime | time | The time when the gateway processed the transaction. |
| created_at | time | The time when the transaction was created. |
| updated_at | time | The time when the transaction was updated. |

## Get a list of transactions

> Example Request

```shell
curl "https://api.mementopayments.com/v1/transactions" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

Get a list of all transactions, in and out, for all of the funding sources belonging to the user. A transaction has a GatewayResponse object if the transaction was processed by a gateway.

### HTTP Request

`GET` `/v1/transactions`

### URL Parameters

|Name|Type|Description|
|----|----|-----------|
|page|int|Item pagination.|
|limit|int|Number of items to return per page.|
|sort|string|Sort the results by `created_at`, `updated_at`.|
|filter|string|Filter the results.|

### Filtering

| Attribute | Type | Operators | Values |
| --------- | ---- | --------- | ------ |
| funding\_source_id | uuid | eq, in | Funding source ID(s). |

## Get a transaction

> Example Request

```shell
curl "https://api.mementopayments.com/v1/transactions/{id}" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

Get a single transaction by ID.

### HTTP Request

`GET` `/v1/transactions/{id}`

# Users

## The user object

> Example Response

```shell
{
  "id": "add5c52a-0c57-4d5c-7525-db14566f2f1a",
  "first_name": "John",
  "last_name": "Dough",
  "full_name": "John Dough",
  "username": "jondough",
  "country": "UK",
  "timezone": "Europe/London",
  "timezone_utc_offset": 0,
  "verified": true,
  "official": true,
  "preferences": "{}",
  "image": {
    "id": "75cc21be-fe47-4702-74bc-07b84beed5fb",
    "url": "https://{imagehost}/ui/users/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
    "full_screen_url": "https://{imagehost}/full/users/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
    "thumbnail_url": "https://{imagehost}/users/moments/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
    "created_at": "2017-09-04T12:26:43.403883Z",
    "updated_at": "2017-09-04T12:26:43.403883Z"
  },
  "relationships": [
    {
      "id": "4ec5c820-520d-4668-ba84-0d7bdee23af5",
      "type": "contact",
      "created_at": "2017-04-19T14:35:09.308904Z",
      "updated_at": "2017-04-19T14:35:09.308904Z"
    }
  ]
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| id | uuid | The unique identifier for the user. |
| first_name | string | The first name of the user. |
| last_name | string | The last name of the user. |
| full_name | string | The full name of the user. |
| username | string | The username of the user. |
| country | string | Two letter ISO 3166-1 alpha-2 country code representing the country the user is located in. |
| timezone | string | The name of timezone where the user is located. |
| timezone_utc_offset | integer | The hours to or from UTC of the timezone where the user is located. |
| verified | boolean | Whether the user has a verified account. |
| official | boolean | Whether the user's account has been marked as an official one. |
| preferences | string | A JSON object which stores the user's preferences. |
| image | Image | An optional user image. |
| relationships | array | An array of Relationship objects describing the relationship between the user and the current user. |

## The current user object

> Example Response

```shell
{
  "id": "add5c52a-0c57-4d5c-7525-db14566f2f1a",
  "status": "active",
  "first_name": "John",
  "last_name": "Dough",
  "full_name": "John Dough",
  "username": "jondough",
  "country": "UK",
  "timezone": "Europe/London",
  "timezone_utc_offset": 0,
  "locale": "en-UK",
  "verified": true,
  "official": true,
  "email": "info@mementopayments.com",
  "date_of_birth": "1985-09-04T12:25:48.288511Z",
  "phone": "+44 123 1234 1234",
  "token": {
    "id": "8d3f94b0-87d0-497f-810c-9b150d42ed05",
    "status": "approved",
    "token": "wxKj3JV6ET1dXVou77675tMqC...",
    "error_code": "",
    "error_message": "",
    "expires_at": "2017-09-04T12:25:48.827724Z"
  }
  "image": {
    "id": "75cc21be-fe47-4702-74bc-07b84beed5fb",
    "url": "https://{imagehost}/ui/users/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
    "full_screen_url": "https://{imagehost}/full/users/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
    "thumbnail_url": "https://{imagehost}/users/moments/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
    "created_at": "2017-09-04T12:26:43.403883Z",
    "updated_at": "2017-09-04T12:26:43.403883Z"
  },
  "created_at": "2017-09-04T12:25:48.827724Z",
  "updated_at": "2017-09-04T12:25:48.827724Z"
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| id | uuid | The unique identifier for the user. |
| status | string | The status of the user.<br>`pending`<br>`active`<br>`locked`<br>`rejected` |
| first_name | string | The first name of the user. |
| last_name | string | The last name of the user. |
| full_name | string | The full name of the user. |
| username | string | The username of the user. |
| country | string | Two letter ISO 3166-1 alpha-2 country code representing the country the user is located in. |
| timezone | string | The name of timezone where the user is located. |
| timezone_utc_offset | integer | The hours to or from UTC of the timezone where the user is located. |
| locale | string | The preferred locale selected by the user. |
| verified | boolean | Whether the user has a verified account. |
| official | boolean | Whether the user's account has been marked as an official one. |
| email | string | An optional email address for the user. |
| date_of_birth | time | An optional date of birth timestamp for the user. |
| phone | string | The user's phone number. |
| token | string | The authentication token of the user. Only sent after creating the user at signup. |
| image | Image | An optional user image. 
| created_at | time | The time when the user was created. |
| updated_at | time | The time when the user was updated. |

## The relationship object

> Example Response

```shell
{
  "id": "4ec5c820-520d-4668-ba84-0d7bdee23af5",
  "type": "contact",
  "created_at": "2017-04-19T14:35:09.308904Z",
  "updated_at": "2017-04-19T14:35:09.308904Z"
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| id | uuid | The unique identifier for the relationship. |
| type | string | The type of the relationship.<br>`blocked`<br>`contact`<br>`following`<br>`friend`<br>`guardian` |
| created_at | time | The time when the relationship was created. |
| updated_at | time | The time when the relationship was updated. |

<!--
### Statuses

| ID | Definition |
| -- | ---------- |
| 1  | active |
| 2  | blocked |
-->

## Get the current user

> Example Request

```shell
curl "https://api.mementopayments.com/v1/users/current" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

Get the currently logged in user. Returns a CurrentUser object.

### HTTP Request

`GET` `/v1/users/current`

## Get a user

> Example Request

```shell
curl "https://api.mementopayments.com/v1/users/{id}" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

Get the public information for a specific user. Returns a User object.

<aside class="notice">
If the current has a relationship to this user, the User object will embed a Relationship object.
</aside>

### HTTP Request

`GET` `/v1/users/{id}`

## Block a user

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/users/{id}/block" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

Block a specific user.

### HTTP Request

`POST` `/v1/users/{id}/block`

## Unblock a user

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/users/{id}/unblock" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

Unblock a specific user.

### HTTP Request

`POST` `/v1/users/{id}/unblock`

## Activity summary

Get a summary of events that occurred after the time defined by the `start_at` filter.

> Example Request

```shell
curl "https://api.mementopayments.com/v1/users/current/summary" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

> Example Response

```shell
{
  "current": {
    "unpaid_requests": 2,
  },
  "since_last_time": {
    "payments_received": 3,
    "requests_received": 2,
  }
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| current.unpaid_requests | integer | The number of current unpaid payment requests. |
| since_last_time.payments_received | integer | The number of payments received during the defined period. |
| since_last_time.requests_received | integer | The number of payment requests received during the defined period. |

### Filtering

| Attribute | Type | Operators | Values |
| --------- | ---- | --------- | ------ |
| start_at | time | gt | YYYY-MM-DD HH:MM:SS |

## Search for users

> Example Request (single value)

```shell
curl -X POST "https://api.mementopayments.com/v1/users/search" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "username": ["johndough"],
  "show_current_friends": true
}'
```

> Example Request (multiple values)

```shell
curl -X POST "https://api.mementopayments.com/v1/users/search" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "name": ["John Dough", "John Doe"],
  "username": ["johndough", "johndoe"],
  "email": ["john@example.com"],
  "phone": ["+44 111 2222 3333"],
  "show_current_friends": true
}'
```

> Example Response

```shell
[
  {
    "id": "add5c52a-0c57-4d5c-7525-db14566f2f1a",
    "first_name": "John",
    "last_name": "Dough",
    "full_name": "John Dough",
    "username": "jondough",
    "country": "UK",
    "timezone": "Europe/London",
    "timezone_utc_offset": 0,
    "verified": true,
    "official": true,
    "image": {
      "id": "75cc21be-fe47-4702-74bc-07b84beed5fb",
      "url": "https://{imagehost}/ui/users/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
      "full_screen_url": "https://{imagehost}/full/users/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
      "thumbnail_url": "https://{imagehost}/users/moments/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
      "created_at": "2017-09-04T12:26:43.403883Z",
      "updated_at": "2017-09-04T12:26:43.403883Z"
    },
    "relationships": [
      {
        "id": "4ec5c820-520d-4668-ba84-0d7bdee23af5",
        "type": "contact",
        "created_at": "2017-04-19T14:35:09.308904Z",
        "updated_at": "2017-04-19T14:35:09.308904Z"
      }
    ]
  }
]
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| name | array | A list of names. |
| username | array | A list of usernames. |
| email | array | A list of emails. |
| phone | array | A list of phone numbers. |
| show_current_friends | string | Show users that are already in the current user's contact list. `default: false`|

Search for users based on one or all of the following: name, username, email, phone number.

### HTTP Request

`POST` `/v1/users/search`

## Create a user – signup

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/users" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "first_name": "John",
  "last_name": "Dough",
  "username": "johndough",
  "email": "john@example.com",
  "phone": "+44 123 1234 1234",
  "pin": "1234",
  "verifications": [
    {
      "field": "email",
      "id": "fa42f949-349a-4d1b-829c-93e6d2daeac4"
    },
    {
      "field": "phone",
      "id": "c5c18397-2ed7-43c1-b481-a5f3d3a96ef1"
    }
  ],
  "device": {
    "id": "582a5abb-1335-4794-4855-11e067b8c55e",
    "make": "iPhone",
    "model": "iPhone6,2",
    "os_name": "iOS",
    "os_version": "8.0"
  },
  "image": {
    "url": "https://upload.wikimedia.org/wikipedia/en/a/a9/Example.jpg"
  }
}'
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| first_name | string | The first name of the user. `required` |
| last_name | string | The last name of the user. `required` |
| username | string | The unique username of the user. `required` |
| email | string | The user's email address. |
| phone | string | The user's phone number. |
| pin | string | The user's PIN. `required` |
| verifications | array | An array of verification IDs for specific user fields. |
| device | Device | The user device information. `required` |
| image | Image | The height of the device's screen. |

Create a new user.

### HTTP Request

`POST` `/v1/users`

## Update the current user

> Example Request

```shell
curl -X PUT "https://api.mementopayments.com/v1/users" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "first_name": "John",
  "last_name": "Dough",
  "username": "johndough",
  "email": "john@example.com",
  "phone": "+44 123 1234 1234",
  "pin": "1234",
  "device": {
    "id": "582a5abb-1335-4794-4855-11e067b8c55e",
    "make": "iPhone",
    "model": "iPhone6,2",
    "os_name": "iOS",
    "os_version": "8.0"
  },
  "image": {
    "url": "https://upload.wikimedia.org/wikipedia/en/a/a9/Example.jpg"
  }
}'
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| first_name | string | The first name of the user. |
| last_name | string | The last name of the user. |
| username | string | The unique username of the user. |
| email | string | The name of the OS running on the device. |
| phone | string | The version of the OS running on the device. |
| pin | string | The user's PIN. |
| device | Device | The user device information. |
| image | Image | The height of the device's screen. |

Update the current user.

### HTTP Request

`PUT` `/v1/users/{id}`

# Verifications

## The verification object

> Example Response

```shell
{
  "id": "f346ced3-f80c-45d9-a9f5-a2b0288cb126",
  "type": "sms",
  "status": "pending",
  "attempts": 1,
  "error_code": "",
  "error_message": "",
  "expires_at": "2017-04-19T14:35:09.308904Z"
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| id | uuid | The unique identifier for the verification. |
| type | string | The type of verification.<br>`sms`<br>`drivers_license`<br>`passport` |
| status | string | The status of verification.<br>`pending`<br>`approved`<br>`rejected`<br>`cancelled`<br>`failed` |
| attempts | integer | The number of verification attemps. Initial value is 1. The maximum number of attempts depends on the verification type and processor. |
| error_code | string | The error key, in case of an error. The value depends on the verification type and processor. | 
| error_message | string | The error message, in case of an error. The value depends on the verification type and processor. | 
| expires_at | time | The time when the verification expires. |

## Single-step verification

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/verifications" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "type": "passport",
  "data": "01234567890123456789",
  "event": "user_kyc"
}'
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| type | string | The type of verification.`required`<br>`drivers_license`<br>`passport` |
| data | string | The data which the verification process needs for processing, e.g. passport ID. `required` |
| event | string | The name of the verification event, which describes the purpose of the verification. This value can be passed along to custom or 3rd party processors. `required` |

Single step verification is when the user provides data of a specific type, such as passport or driver's liences information, and the provider processes it afterwards.

### HTTP Request

`PUT` `/v1/verifications`

## Two-step verification – Step 1: Request data

> Example Request (request verification code)

```shell
curl -X POST "https://api.mementopayments.com/v1/verifications" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "type": "sms",
  "recipient": "+44 123 1234 1234",
  "event": "add_new_device"
}'
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| type | string | The type of verification.`required`<br>`call`<br>`email`<br>`sms` |
| recipient | string | The recipient of the verification code, e.g. phone number. `required` |
| event | string | The name of the verification event, which describes the purpose of the verification. This value can be passed along to custom or 3rd party processors. `required` |

Two-step verification is used when the user initiates the verification process and then provides the required data. For example, request an SMS code for a specific phone number and then provide the code which was sent via SMS.

### HTTP Request

`POST` `/v1/verifications`

## Two-step verification – Step 2: Provide data

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/verifications/{id}/data" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "data": "123456"
}'
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| data | string | The data which the verification process needs for processing, e.g. code sent by SMS. `required` |

Provide the request code in a two-step verification process.

### HTTP Request

`POST` `/v1/verifications/{id}/data`

## Get a verification object

> Example Request

```shell
curl -X GET "https://api.mementopayments.com/v1/verifications/{id}" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

Get a single verification object by ID.

### HTTP Request

`GET` `/v1/verifications/{id}`