---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell

# toc_footers:
#   - <a href='https://www.mementopayments.com'>Memento Payments</a>

includes:
  - errors

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

If you are using the Memento Payments Cloud you need to either provide your project's ID in the header of each request or send requests to your custom host name (CNAME).

All examples in this reference are based on the Memento Payments Cloud endpoint.

# Authentication

To successfully connect to the platform the user needs an authentication token and a session token. The authentication token is used to create a new session token (initially and when the current one expires) and the session token is used for all operations.

All requests to the API need to be accompanied by an Authorization header:

`Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC...`

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
      "uuid": "582a5abb-1335-4794-4855-11e067b8c55e",
      "make": "iPhone",
      "model": "iPhone6,2",
      "os_name": "iOS",
      "os_version": "8.0",
      "screen_width": 480,
      "screen_height": 640,
      "sdk_version": "17"
  }
}'
```

> Example Response

```shell
{
    "uuid": "8d3f94b0-87d0-497f-810c-9b150d42ed05",
    "status_id": 1, // pending
    "token": "wxKj3JV6ET1dXVou77675tMqC..."
}
```

Post identity type + value (e.g. phone number), type of authentication (e.g. "sms") and device. The response will include a UUID and status for lookup.

### HTTP Request

`POST` `/v1/tokens`

## Get authentication token – Step 2

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/tokens" \
  -H "Content-Type: application/json" \
  -d $'{
    "secret": "111111",
    "pin": "1234"
}
```

> Example Response

```shell
{
    "uuid": "8d3f94b0-87d0-497f-810c-9b150d42ed05",
    "status_id": 2, // approved
    "token": "wxKj3JV6ET1dXVou77675tMqC..."
}
```

Post the secret (e.g. verification code) and PIN (depends on the authenticator type).

### HTTP Request

`POST` `/v1/tokens/{uuid}/secret`

## Get authentication token status

```shell
{
    "uuid": "8d3f94b0-87d0-497f-810c-9b150d42ed05",
    "status_id": 2, // approved
}
```

### HTTP Request

`GET` `/v1/tokens/{uuid}`

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
    "uuid": "3f35ae0a-e421-9ba3-531a-b51492d2a5ac",
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

Verify that a specific session token is valid by sending the token as an Authorization header.

### HTTP Request

`POST` `/v1/sessions/verify`

# Response Codes & Errors

|Code|Definition|Returns|Remarks|
|----|----------|-------|-------|
|200 |OK|The requested data.||
|201 |Created|The newly created object.||
|400 |Bad Request|An Error object.|General error within the API.|
|401 |Unauthorized|Empty body.||
|404 |Not Found|Empty body.|Wrong URL or current user does not own specific object being manipulated.|
|422 |Unprocessable Entity|A ValidationError object.|Validation errors, missing parameters or incorrect data.|

# Filtering

Filtering, pagination and sorting is done with query string parameters.

|Parameter|Description|Format|
|---------|-----------|------|
|page|Which page to get.|page=1|
|limit|How many items to get per page.|limit=100|
|filter|Result filtering.|filter=status:eq:open,name:like:john|
|sort|Which field and direction to sort the results.|sort=created_at:desc,name:asc|

## Operators

|Operator|Description|
|--------|-----------|
|eq|Equals|
|gt|Greater than|
|gte|Greater than or equal|
|lt|Less than|
|lte|Less than or equal|
|in|Any of [list]|
|like|Partial text search|

## Example

`GET` `/v1/pools?filter=status:in:[open,closed],created_at:lt:2018-01-01,name:like:john&sort=name:asc`

# Activity Updates

## The activity update object

> Example Response

```shell
{
  "has_updates": true,
  "payments_received": 3,
  "payments_due": 2
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| has_updates | boolean | Whether there are new updates or not. |
| payments_received | integer | The number of payments received. |
| payment_due | integer | The number of payments due. |

## Get activity updates

Get activity updates that occurred after the time defined by the `since` parameter.

> Example Request

```shell
curl "https://api.mementopayments.com/v1/activity/updates" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

### HTTP Request

`GET` `/v1/activity/updates`

### URL Parameters

|Name|Type|Description|
|----|----|-----------|
|since|string|Show updates after a certain date and time. Format is YYYY-MM-DD HH:MM:SS.|

# Announcements

## The announcement object

> Example Response

```shell
{
  "id": 1,
  "type_id": 1,
  "title": "Updated Terms of Use",
  "message": "Lorem ipsum dolor sit amet, consectetur adipiscing elit.",
  "action_url": "http://www.mementopayments.com/terms",
  "action_label": "Open Terms of Use",
  "dismissible": true,
  "created_at": "2017-09-04T12:26:43.403883Z",
  "updated_at": "2017-09-04T12:26:43.403883Z"
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| uuid | uuid | The unique identifier for the announcement. |
| type_id | integer | The type of announcement. |
| title | string | The announcement title. |
| message | string | The announcement message body. |
| action_label | string | If an action is optional or required, this is the action button label. |
| action_url | string | If an action is option or required, this is the URL which the action button opens. |
| dismissible | boolean | Whether the announcement can be dismissed or not. |
| created_at | time | The time when the announcement was created. |
| updated_at | time | The time when the announcement was updated. |

## Get a list of announcements

> Example Request

```shell
curl "https://api.mementopayments.com/v1/announcements" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

Get a list of all announcements that were sent after the time defined by the `since` parameter.

### HTTP Request

`GET` `/v1/announcements`

### URL Parameters

|Name|Type|Description|
|----|----|-----------|
|since|string|Show announcements after a certain date and time. Format is YYYY-MM-DD HH:MM:SS.|

## Get an announcement

> Example Request

```shell
curl "https://api.mementopayments.com/v1/announcement/{uuid}" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

Get a single announcement by UUID.

### HTTP Request

`GET` `/v1/announcements/{uuid}`

# Contacts

## The contact object

> Example Response

```shell
{
  "uuid": "d6edcdba-f3f1-4249-96bc-bb977fde27fb",
  "user_uuid": "fbc521f9-aea8-4da7-840a-1e13ec924b28",
  "name": "John Dough",
  "username": "johndough",
  "country": "UK",
  "emails": ["john@example.com"],
  "phones": ["+44 123 4567 8901"],
  "created_at": "2017-09-04T12:25:52.43349Z",
  "updated_at": "2017-09-04T12:25:52.43349Z"
}
```

A contact can be either a reference to a User or an independent object with a person's name, emails and/or phone numbers.

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| uuid | uuid | The unique identifier for the announcement. |
| user_uuid | uuid | The User object representing the contact, if the contact is a registered user. |
| name | string | The full name of the contact. |
| username | string | The contact's username, if the contact is a registered user. |
| country | string | Two letter ISO 3166-1 alpha-2 country code representing the country the contact is located in. |
| emails | array | A list of the contact's email addresses. |
| phones | array | A list of the contact's phone numbers. |
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
curl "https://api.mementopayments.com/v1/contacts/{uuid}" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

Get a single contact by UUID.

### HTTP Request

`GET` `/v1/contacts/{uuid}`

## Create a contact

> Example Request (create contact from a user)

```shell
curl -X POST "https://api.mementopayments.com/v1/contacts" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "user_uuid": "d6edcdba-f3f1-4249-96bc-bb977fde27fb"
}'
```

> Example Request (create contact with a name and phone number)

```shell
curl -X POST "https://api.mementopayments.com/v1/contacts" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "name": "Jane Dough",
  "email": "jane@example.com",
  "phone": "+44 123 4567 8901"
}
```

Create a new contact. A contact can either be created with a user ID or a name and phone number.

### HTTP Request

`POST` `/v1/contacts`

### Create contact from a user

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| user_uuid | uuid | The unique identifier of the user. `required` |

### Create contact with a name and phone number

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| name | string | The full name of the contact. `required` |
| email | string | The contact's email address. |
| phone | string | The contact's full international phone number. `required` |

## Update a contact

> Example Request

```shell
curl -X PUT "https://api.mementopayments.com/v1/contacts/{uuid}" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "name": "Johanna Dough"
}'
```

Update an existing contact.

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| name | string | The full name of the contact. |
| email | string | The contact's email address. |
| phone | string | The contact's full international phone number. |

### HTTP Request

`PUT` `/v1/contacts/{uuid}`

## Delete a contact

> Example Request

```shell
curl -X DELETE "https://api.mementopayments.com/v1/contacts/{uuid}" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

Delete an existing contact.

### HTTP Request

`DELETE` `/v1/contacts/{uuid}`

# Devices

## Get the current device

Get the user's current device.

## Update the current device

Update the user's current device.

# Fees

## Calculate payment fee

Calculate the fee when sending money from one payment source to another.

# Images

## Get an image

## Upload an image

# Moments

## Get a list of moments

Get a list of all moments.

## Get a moment

Get a single moment by UUID.

# Money Pools

## Get a list of money pools

Get a list of all pools created by the user and pools available to the user but which the user did not create, including public pools and pools the user is invited to or has participated in. This can be specified by using the `owner` filter.

## Get a money pool

Get a single money pool by UUID.

## Create a money pool

Create a new money pool.

## Update a money pool

Update an existing money pool. Anything defined will be updated, otherwise current values will stay unchanged. To remove all amounts, define amounts as an empty array. To leave amounts unchanged, simply do not define amounts in the JSON.

## Close money pool

Closes a money pool so users cannot contribute anymore. The pool's fulfillment status will become `fulfilled` and its status `closed`.

## Invite users to participate

Adds users as participants marked as `invited`.

## Get money pool participants

Get a list of participants in a money pool.

## Export list of participants

Request a list of participants to be sent to a specific email address.

## Contribute to a money pool

The user contributes to the money pool by making a payment. Payment source and PIN is required for payments.

# Notifications

## The notification object

## Get a list of notifications

Get a list of all notifications.

# Payment Sources

## Get a list of payment sources

Get a list of all payment sources.

## Get a payment source

Get a single payment source by UUID.

## Create a payment source

Create a new payment source.

## Update a payment source

Update an existing payment source.

## Delete a payment source

Delete an existing payment source.

## Verify payment source

Verify a payment source using a specific code, which can, for example, be sent to the user's card statement.

# Payments

## Get a list of payments

Get a list of all payments created by the user and payments where the user is the recipient. This can be specified by using the `owner` filter.

## Get a payment

Get a single payment by UUID. Transactions are not accessible from this endpoint (see Transactions).

## Create a payment

Create a new payment. Recipient can be based on a user UUID or phone number, in which case an optional name can also be sent. Payment source and PIN is required for payments.

## Get a payment receipt

Get a receipt for the payment, if it has been fully processed. The sender and recipient will get different versions of the receipt; the sender will see the payment method while the recipient will not.

# Requests

## Get a list of requests

Get a list of all requests created by the user and requests where the user is the recipient. This can be specified by using the `owner` filter.

## Get a request

Get a single request by UUID.

## Create a request

Create a new request.

## Update a request

Update an existing request. Can only update description and image.

## Delete a request

Delete an existing request. Can only be performed if none of the participants have responded.

## Remind a participant to pay

Send a reminder to a request participant in form of a push notification. Note: There is a limit to how many times a participant can be reminded. Exceeding this limit will return in an error message.

## Settle a participant

Mark a participant as paid.

## Cancel a participant

Cancel the request for a participant.

## Pay a request

Pay an existing request as a participant. Payment source and PIN is required for payments.

## Reject a request

Reject an existing request as a participant.

## Get a payment receipt

Get a payment receipt as a participant. The request must be paid, otherwise no receipt will be returned.

## Get a payment receipt for a participant

Get a payment receipt for a specific participant in the request. The participant must have paid, otherwise no receipt will be returned. The request owner will not see the payment method.

# Transactions

## Get a list of transactions

Get a list of all transactions, in and out, for all of the payment sources belonging to the user. A transaction has a GatewayResponse object if the transaction was processed by a gateway.

## Get a transaction

Get a transaction by UUID.

# Users

## Get the current user

Get the currently logged in user.

## Get a user

Get the public information for a specific user.

## Block a user

Block a specific user.

## Unblock a user

Unblock a specific user.

## Search for users

## Create a user – signup

## Update the current user

# Verifications

## Single step verification

## Two step verification

## Get verification status

# Kittens

## Get All Kittens

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get()
```

```shell
curl "http://example.com/api/kittens"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let kittens = api.kittens.get();
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

This endpoint retrieves all kittens.

### HTTP Request

`GET http://example.com/api/kittens`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
include_cats | false | If set to true, the result will also include cats.
available | true | If set to false, the result will include kittens that have already been adopted.

<aside class="success">
Remember — a happy kitten is an authenticated kitten!
</aside>

## Get a Specific Kitten

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

## Delete a Specific Kitten

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

