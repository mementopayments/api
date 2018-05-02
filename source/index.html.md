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

If you are using the Memento Payments Cloud you need to either provide your project's ID in the header of each request or send requests to your custom host name (CNAME).

All examples in this reference are based on the Memento Payments Cloud endpoint.

# Authentication

To successfully connect to the platform the user needs an authentication token and a session token. The authentication token is used to create a new session token (initially and when the current one expires) and the session token is used for all operations.

All requests to the API need to be accompanied by an Authorization header:

`Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC...`

## The authentication object

> Example Response

```shell
{
  "uuid": "8d3f94b0-87d0-497f-810c-9b150d42ed05",
  "status_id": 2,
  "token": "wxKj3JV6ET1dXVou77675tMqC..."
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| uuid | uuid | The unique identifier for the authentication. |
| status_id | integer | The status of the authentication.<br>`1 = pending`<br>`2 = approved`<br>`3 = rejected`|
| token | string | The authentication token. |

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
| uuid | uuid | The unique identifier for the session. |
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
  "status_id": 1,
  "token": "wxKj3JV6ET1dXVou77675tMqC..."
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| identity.type | string | The name of the identity. Can be `phone` or `username`. `required` |
| identity.value | string | The value which to look up the user by, e.g. a username. `required` |
| authenticator | string | The name of the authenticator. Can be `password`, `sms` or a custom authenticator. `required` |
| secret | string | The secret required for the authenticator. `required` |
| device | Device | The user device information. `required` |

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
}'
```

> Example Response

```shell
{
  "uuid": "8d3f94b0-87d0-497f-810c-9b150d42ed05",
  "status_id": 1,
  "token": "wxKj3JV6ET1dXVou77675tMqC..."
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| secret | string | The secret required to authenticate. `required` |
| pin | string | The PIN for the user wanting to authenticate. `required` |

Post the secret (e.g. verification code) and PIN (depends on the authenticator type).

### HTTP Request

`POST` `/v1/tokens/{uuid}/secret`

## Get authentication token status

> Example Request

```shell
curl "https://api.mementopayments.com/v1/tokens/{uuid}" \
  -H "Content-Type: application/json"
}
```

> Example Response

```shell
{
  "uuid": "8d3f94b0-87d0-497f-810c-9b150d42ed05",
  "status_id": 1
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
| type_id | integer | The type of announcement.<br>`1 = general`|
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

### HTTP Request

`PUT` `/v1/contacts/{uuid}`

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| name | string | The full name of the contact. |
| email | string | The contact's email address. |
| phone | string | The contact's full international phone number. |

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

## The device object

> Example Response

```shell
{
  "uuid": "582a5abb-1335-4794-4855-11e067b8c55e",
  "make": "iPhone",
  "model": "iPhone6,2",
  "os_name": "iOS",
  "os_version": "8.0",
  "screen_width": 480,
  "screen_height": 640,
  "sdk_version": "17"
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| uuid | uuid | The unique identifier for the device. |
| make | string | The device make. |
| model | string | The device model. |
| os_name | string | The name of the OS running on the device. |
| os_version | string | The version of the OS running on the device. |
| sdk_version | string | The SDK version of the OS running on the device (Android only). |
| screen_width | integer | The width of the device's screen. |
| screen_height | integer | The height of the device's screen. |

## Get the current device

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/devices/current" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "uuid": "582a5abb-1335-4794-4855-11e067b8c55e",
  "make": "iPhone",
  "model": "iPhone6,2",
  "os_name": "iOS",
  "os_version": "8.0",
  "screen_width": 480,
  "screen_height": 640,
  "sdk_version": "17",
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
| uuid | uuid | The unique identifier for the device. `required` |
| make | string | The device make. `required` |
| model | string | The device model. `required` |
| os_name | string | The name of the OS running on the device. `required` |
| os_version | string | The version of the OS running on the device. `required` |
| sdk_version | string | The SDK version of the OS running on the device (Android only). |
| screen_width | integer | The width of the device's screen. `required` |
| screen_height | integer | The height of the device's screen. `required` |
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
    "payment_source_uuid": "d4e07e85-bb7e-485d-b13a-cd6ee18ff599",
    "currency": "EUR"
  },
  "destination": {
    "payment_source_uuid": "9f5bbafc-cab6-47f4-8489-c8e007ab4288",
    "currency": "USD"
  }
}'
```

Calculate the fee when sending money from one payment source to another.

### HTTP Request

`POST` `/v1/fees/calculate`

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| amount | float | The amount being paid. `required` |
| source | FeePaymentSource | The source from which payment is made and the currency it's made in. `required` |
| destination | FeePaymentSource | The destination to which payment is made and the currency it should be received in. `required` |

### FeePaymentSource

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| payment_source_uuid | uuid | The unique identifier of the payment source handling the payment. `required` |
| currency | string | Three-letter [ISO currency code](https://www.iso.org/iso-4217-currency-codes.html). Must be a supported currency. `required` |

# Images

## The image object

> Example Response

```shell
{
  "uuid": "75cc21be-fe47-4702-74bc-07b84beed5fb",
  "url": "https://{imagehost}/ui/moments/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
  "full_screen_url": "https://{imagehost}/full/moments/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
  "thumbnail_url": "https://{imagehost}/thumbnail/moments/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
  "created_at": "2017-09-04T12:26:43.403883Z",
  "updated_at": "2017-09-04T12:26:43.403883Z"
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| uuid | uuid | The unique identifier for the image. |
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
  "uuid": "ad2636c3-82fe-4c45-af2d-d6324b2e618f",
  "status_id": 1,
  "type_id": 1,
  "object_uuid": "c5d8701e-05cf-4b15-52bf-1cf76c3d84f2",
  "title": "Payment request",
  "note": "Message from user",
  "amount": 10.0,
  "total_amount": 20.0,
  "currency": "EUR",
  "image": {
    "uuid": "75cc21be-fe47-4702-74bc-07b84beed5fb",
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
| uuid | uuid | The unique identifier for the moment. |
| status_id | integer | The moment status.<br>`1 = open`<br>`2 = closed`|
| type_id | integer | The moment type. <br>`1 = payment`<br>`2 = request`<br>`3 = pool`|
| object_uuid | uuid | The unique identifier of the object which the moment refers to. |
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

|Name|Type|Description|
|----|----|-----------|
|page|int|Item pagination.|
|limit|int|Number of items to return per page.|
|sort|string|Sort the results by `created_at`, `updated_at`.|
|filter|string|Filter the results.|

### Filtering

|Attribute|Type|Operators|Values|
|---------|----|---------|------|
|open|boolean|eq|true, false|

## Get a moment

> Example Request

```shell
curl "https://api.mementopayments.com/v1/moments/{uuid}" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

Get a single moment by UUID.

### HTTP Request

`GET` `/v1/moments/{uuid}`

# Money Pools

## The money pool object

> Example Response

```shell
{
  "uuid": "c5d8701e-05cf-4b15-52bf-1cf76c3d84f2",
  "status_id": 1,
  "amount": 60.00,
  "currency": "EUR",
  "description": "Money Pool Title",
  "detailed_description": "Money Pool Description",
  "is_public": true,
  "has_unique_recipients": true,
  "allows_optional_amount": true,
  "minimum_user_amount": 50.00,
  "maximum_user_amount": 150.00,
  "amounts": [
    {
      "uuid": "a0bcfb20-99fd-465d-6e23-2e19e8952420",
      "title": "Option A",
      "amount": 50.00
    }
  ],
  "image": {
    "uuid": "75cc21be-fe47-4702-74bc-07b84beed5fb",
    "url": "https://{imagehost}/ui/pools/c5d8701e-05cf-4b15-52bf-1cf76c3d84f2.jpg",
    "full_screen_url": "https://{imagehost}/full/pools/c5d8701e-05cf-4b15-52bf-1cf76c3d84f2.jpg",
    "thumbnail_url": "https://{imagehost}/thumbnail/pools/c5d8701e-05cf-4b15-52bf-1cf76c3d84f2.jpg",
    "created_at": "2017-09-04T12:26:43.403883Z",
    "updated_at": "2017-09-04T12:26:43.403883Z"
	},
  "owner": {
    "uuid": "add5c52a-0c57-4d5c-7525-db14566f2f1a",
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
      "uuid": "75cc21be-fe47-4702-74bc-07b84beed5fb",
      "url": "https://{imagehost}/ui/users/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
      "full_screen_url": "https://{imagehost}/full/users/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
      "thumbnail_url": "https://{imagehost}/users/moments/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
      "created_at": "2017-09-04T12:26:43.403883Z",
      "updated_at": "2017-09-04T12:26:43.403883Z"
    },
    "relationship": {
      "status_id": 1,
      "created_at": "2017-04-19T14:35:09.308904Z",
      "updated_at": "2017-04-19T14:35:09.308904Z"
    }
  },
  "participants": [
    {
      "uuid": "a0bcfb20-99fd-465d-6e23-2e19e8952420",
      "user_uuid": "add5c52a-0c57-4d5c-7525-db14566f2f1a",
      "transaction_uuid": "875ef796-88a1-4c7f-8755-d4cb066b9a3e",
      "status_id": 1,
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

<!--
| Attribute | Type | Description |
| --------- | ---- | ----------- |
| uuid | uuid | The unique identifier for the money pool. |
| status_id | integer | The money pool status.<br>`1 = open`<br>`2 = closed` |
| TODO: ... |
| currency | string | Three-letter [ISO currency code](https://www.iso.org/iso-4217-currency-codes.html). |
| image | Image | An optional money pool image. |
| participation | Participation | Participation information for the money pool. |
| created_at | time | The time when the money pool was created. |
| updated_at | time | The time when the money pool was updated. |
-->

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
|status|string|eq, in|open, closed, all `default: all`|

## Get a money pool

> Example Request

```shell
curl "https://api.mementopayments.com/v1/pools/{uuid}" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

Get a single money pool by UUID.

### HTTP Request

`GET` `/v1/pools/{uuid}`

## Create a money pool

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/pools" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "description": "Money Pool #1",
  "detailed_description": "This is a more detailed, multiple line decription.",
  "hashtag": "moneypool1",
  "amounts": [
    {
      title: "Payment title",
      amount: 10.0
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
  "only_owner_sees_recipients": true,
  "has_unique_recipients": true,
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
| hashtag | string | An optional hashtag for the money pool. |
| amounts | array | A list of payment options. |
| currency | string | Three-letter [ISO currency code](https://www.iso.org/iso-4217-currency-codes.html). Must be a supported currency. `required` |
| invites | array | A list of users that will be invited to participate in the money pool. |
| image | Image | An optional image object. This can also be performed after creating the money pool. |
| is_public | boolean | Whether everyone can open the money pool or invited users only. Default: `false`. |
| only_owner_sees_participants | boolean | Whether the owner is the only one who can see the list of participants. Default: `false`. |
| has_unique_recipients | boolean | Whether users can only contribute once to the money pool. Default: `false` |
| allows_optional_amount | boolean | Whether users can pay an optional amount of their choice. Default: `false` |
| minimum_user_amount | float | The lowest amount of a single contribution made to the money pool. |
| maximum_user_amount | float | The highest amount of a single contribution made to the money pool. |
| start_at | time | The time at which the money pool will become available. |
| end_at | time | The time at which the money pool will become unavailable. |

## Update a money pool

> Example Request

```shell
curl -X PUT "https://api.mementopayments.com/v1/pools/{uuid}" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "description": "New Title"
}'
```

Update an existing money pool. Anything defined will be updated, otherwise current values will stay unchanged. To remove all amounts, define amounts as an empty array. To leave amounts unchanged, simply do not define amounts in the JSON.

### HTTP Request

`PUT` `/v1/pools/{uuid}`

<!-- TODO: Object -->

## Close money pool

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/pools/{uuid}/close" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

Closes a money pool so users cannot contribute anymore. The pool's fulfillment status will become `fulfilled` and its status `closed`.

### HTTP Request

`POST` `/v1/pools/close`

## Invite users to participate

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/pools/{uuid}/invite" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'[
  {
    "user_uuid": "d31fabcb-dbd8-4a32-824f-23d2dad5a5cc"
  },
  {
    "user_uuid": "5e2c96cd-9701-46e9-b9d2-b4fa4e786f1a"
  }
]'
```

Adds users as participants marked as `invited`.

### HTTP Request

`POST` `/v1/pools/{uuid}/invite`

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| user_uuid | uuid | The unique identifier of the user being invited. `required` |

## Get money pool participants

> Example Request

```shell
curl "https://api.mementopayments.com/v1/pools/{uuid}/participants" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

> Example Response

```shell
[
  {
    "uuid": "a0bcfb20-99fd-465d-6e23-2e19e8952420",
    "user_uuid": "add5c52a-0c57-4d5c-7525-db14566f2f1a",
    "transaction_uuid": "875ef796-88a1-4c7f-8755-d4cb066b9a3e",
    "status_id": 1,
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

`GET` `/v1/pools/{uuid}/participants`

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
curl -X POST "https://api.mementopayments.com/v1/pools/{uuid}/export" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "email": "johndough@example.com",
  "format": "excel"
}'
```

Request a list of participants to be sent to a specific email address.

### HTTP Request

`POST` `/v1/pools/{uuid}/participants/export`

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| email | string | The email which the exported file should be sent to. `required` |
| format | string | The file format of the exported list. Options: `csv`, `excel`. Default: `csv` |

## Contribute to a money pool

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/pools/{uuid}/pay" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "amount": 50.0,
  "payment_source_uuid": "d4097613-3b63-4dbb-befe-2211b9dc821a",
  "pin": "1234"
}'
```

> Example Response

```shell
{
  "uuid": "a0bcfb20-99fd-465d-6e23-2e19e8952420",
  "user_uuid": "add5c52a-0c57-4d5c-7525-db14566f2f1a",
  "transaction_uuid": "875ef796-88a1-4c7f-8755-d4cb066b9a3e",
  "status_id": 1,
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

`POST` `/v1/pools/{uuid}/pay`

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| amount | float | The amount being paid. `required` |
| payment_source_uuid | uuid | The unique identifier for the payment source which will be withdrawn from. `required`|
| pin | string | The current user's PIN. `required` |

# Notifications

## The notification object

> Example Response

```shell
{
  "uuid": "68b206f0-ccea-45df-535f-ed16a97e5530",
  "actor_uuid": "92e7370f-8ea0-4b84-b412-776c4129a7c7",
  "actor": "Arnar",
  "notification_type": "payment",
  "notification_key": "request",
  "object_uuid": "74301781-a1eb-41a6-763c-70f235a636b2",
  "object_data_number": 20.00,
  "object_data_string": "",
  "description": "Arnar accepted your friend request",
  "image": {
    "uuid": "75cc21be-fe47-4702-74bc-07b84beed5fb",
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
| uuid | uuid | The unique identifier for the notification. |
| actor_uuid | uuid | The UUID of the acting user. |
| actor | string | The name of the acting user. |
| notification_type | string | The notification type. Can be either `payment`, `pool`, `request`. |
| notification_key | string | An optional key which explains the action of the notification type, such as `invited` if the user is invited to a money pool. |
| object_uuid | uuid | The unique identifier for the object of the specified notification type. |
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
  "uuid": "a0bcfb20-99fd-465d-6e23-2e19e8952420",
  "user_uuid": "add5c52a-0c57-4d5c-7525-db14566f2f1a",
  "parent_user_uuid": "79d94752-f94a-46ab-8793-7f6434025cf7",
  "transaction_uuid": "875ef796-88a1-4c7f-8755-d4cb066b9a3e",
  "status_id": 1,
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
| uuid | uuid | The unique identifier for the participant. |
| user_uuid | uuid | The unique identifier for the participant user. |
| parent_user_uuid | uuid | The unique identifier for the participant parent user. |
| transaction_uuid | uuid | The unique identifier for the transaction if a payment has been made. |
| status_id | integer | The status of the participant.<br>`1 = pending`<br>`2 = paid`<br>`3 = settled`<br>`4 = rejected`<br>`5 = cancelled`<br>`6 = invited` |
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

# Payment Sources

## Get a list of payment sources

> Example Request

```shell
curl "https://api.mementopayments.com/v1/payment_sources" \
  -H "Content-Type: application/json" 
```

Get a list of all payment sources.

### HTTP Request

`GET` `/v1/payment_sources`

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

## Get a payment source

> Example Request

```shell
curl "https://api.mementopayments.com/v1/payment_sources/{uuid}" \
  -H "Content-Type: application/json" 
```

Get a single payment source by UUID.

### HTTP Request

`GET` `/v1/payment_sources/{uuid}`

## Create a payment source

> Example Request (with BankAccount as type)

```shell
curl -X POST "https://api.mementopayments.com/v1/payment_sources" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "description": "My Bank Account",
  "type": "bank_account",
  "gateway": "bank_of_london",
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
curl -X POST "https://api.mementopayments.com/v1/payment_sources" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "description": "My Default Card",
  "type": "card",
  "gateway": "valitor",
  "card": {
    "expiration_month": 11,
    "expiration_year": 2020,
    "token": "9724017303484431"
  }
}
```

Create a new payment source.

### HTTP Request

`POST` `/v1/payment_sources`

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| description | string | The title of the payment source, only visible to the user. `required` |
| type | string | The type of payment source, can be `bank_account` or `card`. `required` |
| gateway | string | The name of the gateway being used for the payment source type. `required` |
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
| expiration_month | integer | Two digit number representing the card's expiration month. `required` |
| expiration_year | integer | Four digit number representing the card's expiration year. `required` |
| token | string | The tokenized cardholder data used by the card processor gateway. `required` |

## Update a payment source

> Example Request (with BankAccount as type)

```shell
curl -X PUT "https://api.mementopayments.com/v1/payment_sources/{uuid}" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "description": "My Updated Bank Account"
}
```

> Example Request (with Card as type)

```shell
curl -X PUT "https://api.mementopayments.com/v1/payment_sources/{uuid}" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "description": "My Updated Card",
  "card": {
    "expiration_month": 11,
    "expiration_year": 2025
  }
}
```

Update an existing payment source. If the source is a bank account, only its description can be updated. If the source is a card, its description and expiration date can be updated.

### HTTP Request

`PUT` `/v1/payment_sources/{uuid}`

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| description | string | The title of the payment source, only visible to the user. `required` |

### Card

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| expiration_month | integer | Two digit number representing the card's expiration month. `required` |
| expiration_year | integer | Four digit number representing the card's expiration year. `required` |

## Delete a payment source

> Example Request

```shell
curl -X DELETE "https://api.mementopayments.com/v1/payment_sources/{uuid}" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

Delete an existing payment source.

### HTTP Request

`DELETE` `/v1/payment_sources/{uuid}`

## Verify payment source

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/payment_sources/{uuid}/verify" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "code": "abc123"
}
```

Verify a payment source using a specific code, which can, for example, be sent to the user's card statement.

### HTTP Request

`POST` `/v1/payment_sources/{uuid}/verify`

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| code | string | The verification code. `required` |

# Payments

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

|Attribute|Type|Operators|Values|
|---------|----|---------|------|
|owner|boolean|eq|true, false|
|status|string|eq, in|open, closed, all `default: all`|

## Get a payment

> Example Request

```shell
curl "https://api.mementopayments.com/v1/payments/{uuid}" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

Get a single payment by UUID. Transactions are not accessible from this endpoint (see Transactions).

### HTTP Request

`GET` `/v1/payments/{uuid}`

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
    "user_uuid": "3fb6e878-58d6-47f6-ba3c-a5089d6e039a"
  },
  "image": {
    "url": "https://upload.wikimedia.org/wikipedia/en/a/a9/Example.jpg"
  },
  "payment_source_uuid": "f36525d5-39f4-48a9-a547-1887cc69b5cf",
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
  "payment_source_uuid": "f36525d5-39f4-48a9-a547-1887cc69b5cf",
  "pin": "1234"
}
```

Create a new payment and send money. Recipient can be based on a user UUID or phone number, in which case an optional name can also be sent. Payment source and PIN is required for payments.

### HTTP Request

`POST` `/v1/payments`

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| amount | float | The amount being paid. `required` |
| currency | string | Three-letter [ISO currency code](https://www.iso.org/iso-4217-currency-codes.html). Must be a supported currency. `required` |
| description | string | The payment message. |
| recipient | Participant | The recipient of the funds. `required` |
| image | Image | An optional payment image. |
| payment_source_uuid | uuid | The unique identifier for the payment source which will be withdrawn from. `required`|
| pin | string | The current user's PIN. `required` |

## Get a receipt

> Example Request

```shell
curl "https://api.mementopayments.com/v1/payments/{uuid}/receipt" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

> Example Response

```shell
{
  "description": "This is a payment description",
  "reference_code": "00002MC",
  "time": "2017-05-23T12:41:15.813817Z",
  "payer": {
    "uuid": "add5c52a-0c57-4d5c-7525-db14566f2f1a",
    "title": "John Dough (@johndough)",
    "details": null
  },
  "recipient": {
    "uuid": "90bcfb20-99fd-465d-6e23-2e19e8952420",
    "title": "Merchant Name",
    "details": [
      "Memento ehf. (700114-0580)",
      "Bolholti 4, 105 Reykjavík"
    ]
  },
  "payment_method": {
    "uuid": "79d94752-f94a-46ab-8793-7f6434025cf7",
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

`GET` `/v1/payments/{uuid}/receipt`

# Receipt

## The receipt object

> Example Response

```shell
{
  "description": "This is a payment description",
  "reference_code": "00002MC",
  "time": "2017-05-23T12:41:15.813817Z",
  "payer": {
    "uuid": "add5c52a-0c57-4d5c-7525-db14566f2f1a",
    "title": "John Dough (@johndough)",
    "details": null
  },
  "recipient": {
    "uuid": "90bcfb20-99fd-465d-6e23-2e19e8952420",
    "title": "Merchant Name",
    "details": [
      "Memento ehf. (700114-0580)",
      "Bolholti 4, 105 Reykjavík"
    ]
  },
  "payment_method": {
    "uuid": "79d94752-f94a-46ab-8793-7f6434025cf7",
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

<!-- TODO: Receipt object -->

# Requests

## The request object

> Example Response

```shell
{
  "uuid": "c5d8701e-05cf-4b15-52bf-1cf76c3d84f2",
  "status_id": 1,
  "fulfillment_status_id": 2,
  "amount": 20.00,
  "amount_paid": 10.00,
  "currency": "EUR",
  "description": "Payment request",
  "image": {
    "uuid": "75cc21be-fe47-4702-74bc-07b84beed5fb",
    "url": "https://{imagehost}/ui/requests/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
    "full_screen_url": "https://{imagehost}/full/requests/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
    "thumbnail_url": "https://{imagehost}/thumbnail/requests/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
    "created_at": "2017-09-04T12:26:43.403883Z",
    "updated_at": "2017-09-04T12:26:43.403883Z"
	},
  "owner": {
    "uuid": "add5c52a-0c57-4d5c-7525-db14566f2f1a",
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
      "uuid": "75cc21be-fe47-4702-74bc-07b84beed5fb",
      "url": "https://{imagehost}/ui/users/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
      "full_screen_url": "https://{imagehost}/full/users/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
      "thumbnail_url": "https://{imagehost}/users/moments/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
      "created_at": "2017-09-04T12:26:43.403883Z",
      "updated_at": "2017-09-04T12:26:43.403883Z"
    },
    "relationship": {
      "status_id": 1,
      "created_at": "2017-04-19T14:35:09.308904Z",
      "updated_at": "2017-04-19T14:35:09.308904Z"
    }
  },
  "participants": [
    {
      "uuid": "a0bcfb20-99fd-465d-6e23-2e19e8952420",
      "user_uuid": "add5c52a-0c57-4d5c-7525-db14566f2f1a",
      "transaction_uuid": "875ef796-88a1-4c7f-8755-d4cb066b9a3e",
      "status_id": 1,
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
| uuid | uuid | The unique identifier for the moment. |
| status_id | integer | The request status.<br>`1 = open`<br>`2 = closed` |
| fulfillment_status_id | integer | The request fulfillment status.<br>`1 = unfulfilled`<br>`2 = partial`<br>`3 = fulfilled` |
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

`GET` `/v1/reqests`

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
|status|string|eq, in|open, closed, all `default: all`|

## Get a request

Get a single request by UUID.

## Create a request

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/requests" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "amount": 20.0,
  "currency": "EUR",
  "description": "Payment request",
  "participants": [
    {
      "user_uuid": "3fb6e878-58d6-47f6-ba3c-a5089d6e039a",
      "amount": 10.0
    },
    {
      "phone": "+44 111 2222 3333",
      "name": "John Dough",
      "amount": 10.0
    }
  ],
  "image": {
    "url": "https://upload.wikimedia.org/wikipedia/en/a/a9/Example.jpg"
  }
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
| participants | array | A list of participants in the request. `required` |
| image | Image | An optional request image. |

## Update a request

> Example Request

```shell
curl -X PUT "https://api.mementopayments.com/v1/requests/{uuid}" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "description": "My Updated Bank Account"
}
```

Update an existing request. Can only update description and image.

### HTTP Request

`PUT` `/v1/requests/{uuid}`

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| description | string | The title of the request, visible to the owner and participants. |
| image | Image | An optional request image. |

## Delete a request

> Example Request

```shell
curl -X DELETE "https://api.mementopayments.com/v1/requests/{uuid}" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

Delete an existing request. Can only be performed if none of the participants have responded.

### HTTP Request

`DELETE` `/v1/requests/{uuid}`

## Remind a participant to pay

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/requests/{uuid}/participants/{uuid}/remind" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
}
```

Send a reminder to a request participant in form of a push notification. Note: There is a limit to how many times a participant can be reminded. Exceeding this limit will return in an error message.

### HTTP Request

`POST` `/v1/requests/{uuid}/participants/{uuid}/remind`

## Settle a participant

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/requests/{uuid}/participants/{uuid}/settle" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
}
```

Mark a participant as paid.

### HTTP Request

`POST` `/v1/requests/{uuid}/participants/{uuid}/remind`

## Cancel a participant

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/requests/{uuid}/participants/{uuid}/cancel" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
}
```

Cancel the request for a participant.

### HTTP Request

`POST` `/v1/requests/{uuid}/participants/{uuid}/cancel`

## Pay a request

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/requests/{uuid}/pay" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..." \
  -d $'{
  "payment_source_uuid": "d4097613-3b63-4dbb-befe-2211b9dc821a",
  "pin": "1234"
}'
```

> Example Response

```shell
{
  "uuid": "a0bcfb20-99fd-465d-6e23-2e19e8952420",
  "user_uuid": "add5c52a-0c57-4d5c-7525-db14566f2f1a",
  "transaction_uuid": "875ef796-88a1-4c7f-8755-d4cb066b9a3e",
  "status_id": 1,
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

`POST` `/v1/pools/{uuid}/pay`

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| payment_source_uuid | uuid | The unique identifier for the payment source which will be withdrawn from. `required`|
| pin | string | The current user's PIN. `required` |

### HTTP Request

`POST` `/v1/requests/{uuid}/pay`

## Reject a request

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/requests/{uuid}/reject" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
}
```

Reject an existing request as a participant.

### HTTP Request

`POST` `/v1/requests/{uuid}/reject`

## Get a receipt

> Example Request

```shell
curl "https://api.mementopayments.com/v1/requests/{uuid}/receipt" \
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
    "uuid": "add5c52a-0c57-4d5c-7525-db14566f2f1a",
    "title": "John Dough (@johndough)",
    "details": null
  },
  "recipient": {
    "uuid": "90bcfb20-99fd-465d-6e23-2e19e8952420",
    "title": "Merchant Name",
    "details": [
      "Memento ehf. (700114-0580)",
      "Bolholti 4, 105 Reykjavík"
    ]
  },
  "payment_method": {
    "uuid": "79d94752-f94a-46ab-8793-7f6434025cf7",
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

`GET` `/v1/requests/{uuid}/receipt`

## Get a receipt for a participant

> Example Request

```shell
curl "https://api.mementopayments.com/v1/requests/{uuid}/participants/{uuid}/receipt" \
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
    "uuid": "add5c52a-0c57-4d5c-7525-db14566f2f1a",
    "title": "John Dough (@johndough)",
    "details": null
  },
  "recipient": {
    "uuid": "90bcfb20-99fd-465d-6e23-2e19e8952420",
    "title": "Merchant Name",
    "details": [
      "Memento ehf. (700114-0580)",
      "Bolholti 4, 105 Reykjavík"
    ]
  },
  "payment_method": {
    "uuid": "79d94752-f94a-46ab-8793-7f6434025cf7",
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

`GET` `/v1/requests/{uuid}/participants/{uuid}/receipt`

# Transactions

## The transaction object

> Example Response

```shell
{
  "uuid": "add5c52a-0c57-4d5c-7525-db14566f2f1a",
  "out_user_uuid": "dd72ebb8-db1f-4442-b203-095ac9ded974",
  "in_user_uuid": "1c478b12-288a-4ea0-831d-1e36639300da",
  "out_payment_source_uuid": "d4097613-3b63-4dbb-befe-2211b9dc821a",
  "in_payment_source_uuid": "b1f6a7de-7a8b-4c3f-a908-a02e16f8e529",
  "payment_uuid": "745ad357-c7dc-478d-a46b-a97ebd9de4c7",
  "status_id": 2,
  "amount": 50.0,
  "currency": "EUR",
  "tracking_code": "DEF456",
  "error": false,
  "gateway_response": {
    "uuid": "79d90419-dc82-4093-6afc-65f8b206fea0",
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
| uuid | uuid | The unique identifier for the moment. |
| out_user_uuid | uuid | The unique identifier for the user who made the transaction. |
| in_user_uuid | uuid | The unique identifier for the user who received the transaction. |
| out_payment_source_uuid | uuid | The unique identifier for the payment source which was withdrawn from. |
| in_payment_source_uuid | uuid | The unique identifier for the payment source which was deposited to. |
| status_id | integer | The transactions status.<br>`1 = active`<br>`2 = approved`<br>`3 = rejected`<br>`4 = cancelled`<br>`5 = failed` |
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
  "uuid": "79d90419-dc82-4093-6afc-65f8b206fea0",
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
| uuid | uuid | The unique identifier for the gateway response. |
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

Get a list of all transactions, in and out, for all of the payment sources belonging to the user. A transaction has a GatewayResponse object if the transaction was processed by a gateway.

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

|Attribute|Type|Operators|Values|
|---------|----|---------|------|
|payment\_source_uuid|uuid|eq, in|Payment source UUID(s).|

## Get a transaction

> Example Request

```shell
curl "https://api.mementopayments.com/v1/transactions/{uuid}" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

Get a single transaction by UUID.

### HTTP Request

`GET` `/v1/transactions/{uuid}`

# Users

## The user object

> Example Response

```shell
{
  "uuid": "add5c52a-0c57-4d5c-7525-db14566f2f1a",
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
    "uuid": "75cc21be-fe47-4702-74bc-07b84beed5fb",
    "url": "https://{imagehost}/ui/users/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
    "full_screen_url": "https://{imagehost}/full/users/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
    "thumbnail_url": "https://{imagehost}/users/moments/ad2636c3-82fe-4c45-af2d-d6324b2e618f.jpg",
    "created_at": "2017-09-04T12:26:43.403883Z",
    "updated_at": "2017-09-04T12:26:43.403883Z"
  },
  "relationship": {
    "status_id": 1,
    "created_at": "2017-04-19T14:35:09.308904Z",
    "updated_at": "2017-04-19T14:35:09.308904Z"
  }
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| uuid | uuid | The unique identifier for the user. |
| first_name | string | The first name of the user. |
| last_name | string | The last name of the user. |
| full_name | string | The full name of the user. |
| username | string | The username of the user. |
| country | string | Two letter ISO 3166-1 alpha-2 country code representing the country the user is located in. |
| timezone | string | The name of timezone where the user is located. |
| timezone_utc_offset | integer | The hours to or from UTC of the timezone where the user is located. |
| verified | boolean | Whether the user has a verified account. |
| official | boolean | Whether the user's account has been marked as an official one. |
| image | Image | An optional user image. |
| relationship| Relationship | The relationship between the user and the current user. |

## The current user object

> Example Response

```shell
{
  "uuid": "add5c52a-0c57-4d5c-7525-db14566f2f1a",
  "status_id": 1,
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
  "token": "jLIeXTajtW1j4bfC2opJI...",
  "image": {
    "uuid": "75cc21be-fe47-4702-74bc-07b84beed5fb",
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
| uuid | uuid | The unique identifier for the user. |
| status_id | integer | The status of the user.<br>`1 = pending`<br>`2 = active`<br>`3 = locked`<br>`4 = rejected` |
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
| image | Image | An optional user image. |
| created_at | time | The time when the user was created. |
| updated_at | time | The time when the user was updated. |

## The relationship object

> Example Response

```shell
{
  "status_id": 1,
  "created_at": "2017-04-19T14:35:09.308904Z",
  "updated_at": "2017-04-19T14:35:09.308904Z"
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| status_id | integer | The status of the relationship.<br>`1 = active`<br>`2 = blocked` |
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
curl "https://api.mementopayments.com/v1/users/{uuid}" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

Get the public information for a specific user. Returns a User object.

<aside class="notice">
If the current has a relationship to this user, the User object will embed a Relationship object.
</aside>

### HTTP Request

`GET` `/v1/users/{uuid}`

## Block a user

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/users/{uuid}/block" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

Block a specific user.

### HTTP Request

`POST` `/v1/users/{uuid}/block`

## Unblock a user

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/users/{uuid}/unblock" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

Unblock a specific user.

### HTTP Request

`POST` `/v1/users/{uuid}/unblock`

<!-- ## Search for users

TODO: Search -->

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
  "device": {
    "uuid": "582a5abb-1335-4794-4855-11e067b8c55e",
    "make": "iPhone",
    "model": "iPhone6,2",
    "os_name": "iOS",
    "os_version": "8.0",
    "screen_width": 480,
    "screen_height": 640,
    "sdk_version": "17"
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
| email | string | The name of the OS running on the device. |
| phone | string | The version of the OS running on the device. `required` |
| pin | string | The user's PIN. `required` |
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
    "uuid": "582a5abb-1335-4794-4855-11e067b8c55e",
    "make": "iPhone",
    "model": "iPhone6,2",
    "os_name": "iOS",
    "os_version": "8.0",
    "screen_width": 480,
    "screen_height": 640,
    "sdk_version": "17"
  },
  "image": {
    "url": "https://upload.wikimedia.org/wikipedia/en/a/a9/Example.jpg"
  }
}'
```

<!-- TODO: Which attributes? -->

Update the current user.

### HTTP Request

`PUT` `/v1/users/{uuid}`

# Verifications

## The verification object

> Example Response

```shell
{
  "uuid": "f346ced3-f80c-45d9-a9f5-a2b0288cb126",
  "type_id": 1,
  "status_id": 1,
  "attempts": 1,
  "error_code": "",
  "error_message": "",
  "expires_at": "2017-04-19T14:35:09.308904Z"
}
```

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| uuid | uuid | The unique identifier for the announcement. |
| type_id | integer | The type of verification.<br>`1 = sms`<br>`2 = driver's license`<br>`3 = passport` |
| status_id | integer | The status of verification.<br>`1 = pending`<br>`2 = approved`<br>`3 = rejected`<br>`4 = cancelled`<br>`5 = failed` |
| attempts | integer | The number of verification attemps. Initial value is 1. The maximum number of attempts depends on the verification type and processor. |
| error_code | string | The error code, in case of an error. The value depends on the verification type and processor. | 
| error_message | string | The error message, in case of an error. The value depends on the verification type and processor. | 
| dismissible | boolean | Whether the announcement can be dismissed or not. |
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
| type | string | The type of verification. Can be `driver's license` or `passport`. `required` |
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
| type | string | The type of verification. Can be `call`, `email` or `sms`. `required` |
| recipient | string | The recipient of the verification code, e.g. phone number. `required` |
| event | string | The name of the verification event, which describes the purpose of the verification. This value can be passed along to custom or 3rd party processors. `required` |

Two-step verification is used when the user initiates the verification process and then provides the required data. For example, request an SMS code for a specific phone number and then provide the code which was sent via SMS.

### HTTP Request

`POST` `/v1/verifications`

## Two-step verification – Step 2: Provide data

> Example Request

```shell
curl -X POST "https://api.mementopayments.com/v1/verifications/{uuid}/data" \
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

`POST` `/v1/verifications/{uuid}/data`

## Get a verification object

> Example Request

```shell
curl -X GET "https://api.mementopayments.com/v1/verifications/{uuid}" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

Get a single verification object by UUID.

### HTTP Request

`GET` `/v1/verifications/{uuid}`