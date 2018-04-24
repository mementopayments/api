---
title: Memento Payments API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell: curl

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

### HTTP Request

`PUT` `/v1/devices/current`

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

### HTTP Request

`POST` `/v1/fees/calculate`

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
| status_id | integer | The moment status. |
| type_id | integer | The moment type. |
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
    { TODO: PaymentAmount object }
  ],
  "image": {
    "uuid": "75cc21be-fe47-4702-74bc-07b84beed5fb",
    "url": "https://{imagehost}/ui/pools/c5d8701e-05cf-4b15-52bf-1cf76c3d84f2.jpg",
    "full_screen_url": "https://{imagehost}/full/pools/c5d8701e-05cf-4b15-52bf-1cf76c3d84f2.jpg",
    "thumbnail_url": "https://{imagehost}/thumbnail/pools/c5d8701e-05cf-4b15-52bf-1cf76c3d84f2.jpg",
    "created_at": "2017-09-04T12:26:43.403883Z",
    "updated_at": "2017-09-04T12:26:43.403883Z"
	},
  "owner": { TODO: User object },
  "participants": [
    { TODO: Participant object }
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
| uuid | uuid | The unique identifier for the money pool. |
| status_id | integer | The money pool status. |
| TODO: ... |
| currency | string | Three-letter [ISO currency code](https://www.iso.org/iso-4217-currency-codes.html). |
| image | Image | An optional money pool image. |
| participation | Participation | Participation information for the money pool. |
| created_at | time | The time when the money pool was created. |
| updated_at | time | The time when the money pool was updated. |

## Get a list of money pools

> Example Request

```shell
curl "https://api.mementopayments.com/v1/pools" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

Get a list of all pools created by the user and pools available to the user but which the user did not create, including public pools and pools the user is invited to or has participated in. This can be specified by using the `owner` filter.

### HTTP Request

`GET` `/v1/pools`

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
curl -X POST "https://api.mementopayments.com/v1/contacts" \
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
  "image": { TODO: Image object },
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

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| description | string | The money pool title. `required` |
| detailed_description | string | Any description for the money pool. |
| hashtag | string | An optional hashtag for the money pool. |
| amounts | array | A list of payment options (PaymentAmount). |
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

### HTTP Request

`POST` `/v1/pools`

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

TODO: Object

### HTTP Request

`PUT` `/v1/pools/{uuid}`

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

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| user_uuid | uuid | The unique identifier of the user being invited. `required` |

### HTTP Request

`POST` `/v1/pools/{uuid}/invite`

## Get money pool participants

> Example Request

```shell
curl "https://api.mementopayments.com/v1/pools/{uuid}/participants" \
  -H "Authorization: Bearer wxKj3JV6ET1dXVou77675tMqC..."
```

> Example Response

```shell
[
  { TODO: Participant object }
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

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| email | string | The email which the exported file should be sent to. `required` |
| format | string | The file format of the exported list. Options: `csv`, `excel`. Default: `csv` |

### HTTP Request

`POST` `/v1/pools/{uuid}/participants/export`

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
{ TODO: Participant object }
```

The user contributes to the money pool by making a payment. Payment source and PIN is required for payments.

| Attribute | Type | Description |
| --------- | ---- | ----------- |
| amount | float | The amount being paid. `required` |
| payment_source_uuid | uuid | The unique identifier for the payment source which will be withdrawn from. `required`|
| pin | string | The current user's PIN. `required` |

### HTTP Request

`POST` `/v1/pools/{uuid}/pay`

# Notifications

## The notification object

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

### Filtering

TODO: Filtering

# Participation

Participation information for a moment, request or money pool.

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
    "expiration_year": 2017,
    "token": "9724017303484431"
  }
}
```

Create a new payment source.

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

### HTTP Request

`POST` `/v1/payment_sources`

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

## Get a receipt

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

## Get a receipt

Get a payment receipt as a participant. The request must be paid, otherwise no receipt will be returned.

## Get a receipt for a participant

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
