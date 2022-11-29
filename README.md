# API Simpplo

One API integration allows to access all your payment options.

### Current status

Currently only API version 1 is available

## Basic usage

API requests should be in sandbox or production mode, after the main URL has the environment mode

Sandbox URL:

```shell
curl "https://api.agiltec.mx/sandbox/v1/"
```

Production URL:

```shell
curl "https://api.agiltec.mx/production/v1/"
```
The API uses JSON to serialize data. You don't need to specify `.json` at the
end of an API URL.

## Authentication

Most API requests require authentication, or will only return public data when
authentication is not provided. For
those cases where it is not required, this will be mentioned in the documentation
for each individual endpoint.

If authentication information is invalid or omitted, an error message will be
returned with status code `401`:

```json
{
  "message": "401 Unauthorized"
}
```

### How to use the Token

You can use the token that was sent to your email account to authenticate with the API by passing it in either the `Token` header.

Example of using the token in a header:

```shell
curl --header "Token: AUTH-TOKEN" https://api.agiltec.mx/production/v1/
```

## Status codes

The API is designed to return different status codes according to context and
action. This way, if a request results in an error, the caller is able to get
insight into what went wrong.

The following table gives an overview of how the API functions generally behave.

| Request type | Description |
| ------------ | ----------- |
| `GET`   | Access one or more resources and return the result as JSON. |
| `POST`  | Return `201 Created` if the resource is successfully created and return the newly created resource as JSON. |
| `GET` / `PUT` | Return `200 OK` if the resource is accessed or modified successfully. The (modified) result is returned as JSON. |
| `DELETE` | Returns `204 No Content` if the resource was deleted successfully. |

The following table shows the possible return codes for API requests.

| Return values | Description |
| ------------- | ----------- |
| `200 OK` | The `GET`, `PUT` or `DELETE` request was successful, the resource(s) itself is returned as JSON. |
| `204 No Content` | The server has successfully fulfilled the request and that there is no additional content to send in the response payload body. |
| `201 Created` | The `POST` request was successful and the resource is returned as JSON. |
| `304 Not Modified` | Indicates that the resource has not been modified since the last request. |
| `400 Bad Request` | A required attribute of the API request is missing, e.g., the title of an issue is not given. |
| `401 Unauthorized` | The user is not authenticated, a valid [user token](#authentication) is necessary. |
| `403 Forbidden` | The request is not allowed, e.g., the user is not allowed to delete a project. |
| `404 Not Found` | A resource could not be accessed, e.g., an ID for a resource could not be found. |
| `405 Method Not Allowed` | The request is not supported. |
| `409 Conflict` | A conflicting resource already exists, e.g., creating a project with a name that already exists. |
| `412` | Indicates the request was denied. May happen if the `If-Unmodified-Since` header is provided when trying to delete a resource, which was modified in between. |
| `422 Unprocessable` | The entity could not be processed. |
| `500 Server Error` | While handling the request something went wrong server-side. |

## Create a charge

```plaintext
POST payments/process
```

BODY
| Attribute | Type           | Required | Description |
| --------- | -------------- | -------- | ----------- |
| `transaction_id`      | string | yes      | The ID to identify a transaction |
| `amount`      | double | yes      | The transaction amount, example: 200.50 |
| `currency`      | string | yes      | Example: MXN or USD |
| `cc_name`      | string | yes      | Billing information: Full name |
| `cc_number`      | integer | yes      | Billing information: Card number |
| `cc_expmonth`      | integer | yes      | Billing information: Card expiration month |
| `cc_expyear`      | integer | yes      | Billing information: Card expiration year |
| `cc_method`      | integer | yes      | Billing information: Card method, Example: VISA |
| `cc_type`      | string | yes      | Billing information: Card type, Example: debit or credit |
| `user_id`      | string | yes      | The ID to identify the user card information |
| `user_name`      | string | no      | The user name |
| `user_email`      | string | no      | The user email: Example: hello@simpplo.com |
| `user_ip`      | integer | no      | The IP Address |

💡 For testing purposes you can use that cards in Sandbox:

- Transaction Successful: 4111111111111111
- Transaction Rejected: 4222222222222220
- Card expired: 4000000000000069
- Insufficient funds: 4444444444444448

Example request:

```bash
curl --request POST \
  --url https://api.agiltec.mx/sandbox/v1/payments/process \
  --header 'Token: AUTH-TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
  "tx_reference": "referencia de pago3",
  "tx_amount": "3",
  "tx_currency": "MXN",
  "cc_name": "Fernando Zapien",
  "cc_number": "4111111111111111",
  "cc_expMonth": "02",
  "cc_expYear": "25",
  "cc_cvv": "123",
  "tx_browserIP": "192.168.41.23",
  "opcion": "Santander",
  "tipoPago": "1",
  "url": "urlretorno"
}'
```

RESPONSE
| Attribute | Type           | Required | Description |
| --------- | -------------- | -------- | ----------- |
| `status`      | string | yes      | Example: Success or Error |
| `statuscode`      | integer | yes      | The status code error |
| `body`      | string | yes      | System response |
| `data`      | string | no      | Additional response information |
| `reference`      | string | yes      | ID to identify the authorization |
| `responseId`      | integer | no      | Bank folio number |

Example response:

```json
{
  "status": "Success",
  "statuscode": 200,
  "body": "<!DOCTYPE HTML PUBLIC '-//W3C//DTD HTML 4.01 Transitional//EN'><html><h..",
  "data": "",
  "reference": "3010",
  "responseId": "766892"
}
```

## Create a refund

```plaintext
POST payments/refund
```

BODY
| Attribute | Type           | Required | Description |
| --------- | -------------- | -------- | ----------- |
| `reference`      | string | yes      | ID to identify the authorization |

Example request:

```bash
curl --request POST \
  --url https://api.agiltec.mx/sandbox/v1/payments/refund \
  --header 'Token: AUTH-TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
  "reference": "3012"
}'
```

RESPONSE
| Attribute | Type           | Required | Description |
| --------- | -------------- | -------- | ----------- |
| `status`      | string | yes      | Example: Success or Error |
| `statuscode`      | integer | yes      | The status code error |
| `body`      | string | yes      | System response |

Example response:

```json
{
  "status": "Success",
  "statusCode": 200,
  "body": "Devolución exitosa"
}
```

## Get payment status

```plaintext
POST payments/verify
```

BODY
| Attribute | Type           | Required | Description |
| --------- | -------------- | -------- | ----------- |
| `reference`      | string | yes      | ID to identify the authorization |

Example request:

```bash
curl --request POST \
  --url https://api.agiltec.mx/sandbox/v1/payments/verify \
  --header 'Token: AUTH-TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
  "reference": "3012"
}'
```

RESPONSE
| Attribute | Type           | Required | Description |
| --------- | -------------- | -------- | ----------- |
| `status`      | string | yes      | Example: Success or Error |
| `statuscode`      | integer | yes      | The status code error |
| `body`      | string | yes      | System response |
| `data`      | string | no      | Additional response information |
| `reference`      | string | yes      | ID to identify the authorization |
| `responseId`      | integer | no      | Bank folio number |

Example response:

```json
{
  "status": "Success",
  "statusCode": 200,
  "body": "Autenticación exitosa",
  "data": null,
  "reference": "3012",
  "responseId": "766892"
}
```
