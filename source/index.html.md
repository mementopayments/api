---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell

toc_footers:
  - <a href='https://www.mementopayments.com'>Memento Payments</a>

includes:
  - errors

search: true
---

# Introduction

> API Endpoint

```shell
https://api.mementopayments.com
```

Welcome to the Kittn API! You can use our API to access Kittn API endpoints, which can get information on various cats, kittens, and breeds in our database.

We have language bindings in Shell, Ruby, and Python! You can view code examples in the dark area to the right, and you can switch the programming language of the examples with the tabs in the top right.

This example API documentation page was created with [Slate](https://github.com/lord/slate). Feel free to edit it and use it as a base for your own API's documentation.

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

`POST https://api.mementopayments.com/v1/tokens`

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

`POST https://api.mementopayments.com/v1/tokens/{uuid}/secret`

## Get authentication token status

```shell
{
    "uuid": "8d3f94b0-87d0-497f-810c-9b150d42ed05",
    "status_id": 2, // approved
}
```

### HTTP Request

`GET https://api.mementopayments.com/v1/tokens/{uuid}`

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

`POST https://api.mementopayments.com/v1/sessions`

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

`POST https://api.mementopayments.com/v1/sessions/verify`

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

`GET https://api.mementopayments.com/v1/pools?filter=status:in:[open,closed],created_at:lt:2018-01-01,name:like:john&sort=name:asc`

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

> Example Request

```shell
curl "https://api.mementopayments.com/v1/activity/updates" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

### HTTP Request

`GET https://api.mementopayments.com/v1/activity/updates`

### URL Parameters

|Name|Type|Description|
|----|----|-----------|
|since|string|Show updates after a certain date and time. Format is YYYY-MM-DD HH:MM:SS.|

# Announcements

## The announcement object

# Contacts

# Devices

# Fees

# Images

# Moments

# Money Pools

# Notifications

# Payment Sources

# Payments

# Requests

# Transactions

# Users

# Verifications


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

