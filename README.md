# Api domain: `https://approveme.cc`
---
# `POST` /api/1.0/create_task?apikey=`<apikey>`
**Content-type: `multipart/form-data`**
Create a task to generate document. Some parameters are not required. If the parameter is not passed, then its value will be randomly selected. You will receive the passed or selected value in the response.
## Request
```
{
    "country": <string>,
    "sex": <string>,
    "name": <string>,
    "surname": <string>,
    "birth": <string>,
    "file": <binary>
}
```

**Body params description**
| Name | Type | Required | Description |
| ---- | ---- | -------- | ----------- |
| `country` | string | true | 2-letters country code. See list of countries. |
| `sex` | string | true | `male` or `female`. |
| `name` | string | false * | First name (only letters). |
| `surname` | string | false * | Last name (only letters). |
| `file` | binary | false * | Photo in bytes. See table of available sizes. |

\* Not required params can be `null` or you can choose not to pass them.

**List of countries:**
* `ru` - Russia;
* `ua` - Ukraine.

**Photo sizes:**
| Country code | Minimum size |
| ------------ | ------------ |
| `ru` | 192x192 |
| `ua` | 192x192 |

## Response
### `201` Created
```
{
    "message": {
        "id": <integer>,
        "status": true,
        "code": 201,
        "data": {
            "sex": <string>,
            "name": <string>,
            "surname": <string>,
            "birth": <string>,
            "file": <boolean>
        }
    }
}
```

### `400` Bad Request
```
{
    "message": {
        "status": false,
        "code": 400,
        "reason": <string>,
        "description": <string>
    }
}
```

**Errors description:**
| Reason | Description |
| ------ | ----------- |
| EMPTY_COUNTRY | Country is empty |
| BAD_COUNTRY | Country not found |
| BAD_BIRTH | Date format must be: DD.MM.YYYY |
| EMPTY_SEX | Sex must not be empty |
| BAD_SEX | Sex must be "male" or "female" |
| BAD_IMAGE | File has not recognized as image |
| BAD_SIZE | Width and height of the image must be <width>x<height> or more |
| BAD_BALANCE | Not enough money on the account |

---

# `GET` /api/1.0/get_status?id=`<id>`
Get status of created task.

## Request
**Url params:**
| Name | Type | Required | Description |
| ---- | ---- | -------- | ----------- |
| `id` | integer | true | `id` from response of the `create_task` method |

## Response
### `200` OK
```
{
    "message": {
        {
            "id": <integer>,
            "status": true,
            "code": 200,
            "done": <boolean>,
            "file": <base64>
        }
    }
}
```
**Response params description:**
| Name | Type | Description |
| ---- | ---- | ----------- |
| `id` | integer | `id` from response of the `create_task` method |
| `done` | boolean | `true` if document is ready; `false` if not ready yet |
| `file` | base64 or null | Return base64 encoded image if `done` is `true`; else return `null`

### `400` Bad Request
```
{
    "message": {
        "status": false,
        "code": 400,
        "reason": <string>,
        "description": <string>
    }
}
```
**Errors description:**
| Reason | Description |
| ------ | ----------- |
| EMPTY_ID | ID is empty |

# `GET` /api/1.0/get_info?apikey=`<apikey>`
Get your balance, member status and prices.

## Response
### `200` OK
```
{
    "message": {
        "status": true,
        "code": 200,
        "balance": <float>,
        "member": <integer>,
        "prices": {
            "ru": <float>,
            "ua": <float>,
            -------------
        }
    }
}
```

# Common Errors
### `400` Bad Request
| Reason | Description |
| ------ | ----------- |
| BAD_APIKEY | API key is wrong or empty |

### `500` Internal Server Error
```
{
    'message': {
        "status": false,
        "code": 500,
        "reason": "UNKNOWN_ERROR",
        "description": "Unknown error"
    }
}
```

### `429` Too Many Requests
All API requests are limited to `2 per second` and `120 per minute`. If you see this status code, you need to increase delays between requests. You can get more info in following response headers: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset` and `Retry-After`.
