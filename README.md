# API domain: `https://approveme.cc`
**Table of contents**
- [Method **create_task**](#post-api10create_taskapikeyapikey)
- [Method **get_status**](#get-api10get_statusidcreate_taskid)
- [Method **get_info**](#get-api10get_infoapikeyapikey)
- [Common Errors](#common-errors)

# `POST` /api/1.0/create_task?apikey=`<apikey>`
**Content-type: `multipart/form-data`**

Create a task to generate document. Some parameters are not required. If the parameter is not passed, then its value will be randomly selected. You will receive the passed or selected value in the response.

## Request
```
{
    "country": <string>,
    "mode": <string>,
    "sex": <string>,
    "name": <string>,
    "surname": <string>,
    "patronymic": <string>,
    "birth": <string>,
    "remove_bg": <boolean>,
    "file": <binary>
}
```

**Body params description**
| Name | Type | Required | Description |
| ---- | ---- | -------- | ----------- |
| `country` | string | true | See list of countries. |
| `mode` | string | true | `id`, `passport`. See list of countries. |
| `sex` | string | true | `male` or `female`. |
| `name` | string | false \* | First name (only letters). |
| `surname` | string | false \* | Last name (only letters). |
| `birth` | string | false \* | Birthday (DD.MM.YYYY). |
| `remove_bg` \*\* | boolean | false \* | `true` - remove background of photo, `false` - not remove. Default `true`. |
| `file` | binary | false \* | Photo in bytes. You should send a photo larger then minimum sizes and saving original ratio. See photo sizes. |

**Extra params available for some countries (see list of countries)**
| Name | Type | Required | Description |
| ---- | ---- | -------- | ----------- |
| `patronymic` | string | false \* | Second name (only letters). |

\* Not required params can be `null` or you can choose not to pass them.

\*\* You should set `remove_bg` to `false` always if you don't need to remove background. If the photo has a transparent background, then `true` can lead to the deletion of part of the original image!

**List of countries**
| Country | Country Description | Mode | Extra Params |
| ------- | ------------------- | ----- | ---------------- |
| `ru` | Russia | `passport` | `patronymic` |
| `ua` | Ukraine | `id` | `patronymic` |

**Photo sizes**
| Country | Mode | Minimum Sizes | Ratio |
| ------- | ---- | ----- | ----- |
| `ru` | `passport` | 191x191 | 1x1 |
| `ua` | `id` | 192x256 | 35x45 |

## Response
### `201` Created
```
{
    "message": {
        "id": <string>,
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

**Errors description**
| Reason | Description |
| ------ | ----------- |
| EMPTY_COUNTRY | Country is empty |
| BAD_COUNTRY | Country not found |
| EMPTY_MODE | Mode is empty. You should pass "id" or "passport" |
| BAD_MODE | Mode not found. You should pass one of the following params available for country "\<country>": \<mode1>, \<mode2>... |
| BAD_BIRTH | Date format must be: DD.MM.YYYY |
| EMPTY_SEX | Sex must not be empty |
| BAD_SEX | Sex must be "male" or "female" |
| BAD_IMAGE | File has not recognized as image |
| BAD_SIZE | Width and height of the image must be \<width>x\<height> or more |
| BAD_BALANCE | Not enough money on the account |

---

# `GET` /api/1.0/get_status?id=`<create_task:id>`
Get status of created task.

**Attention**: you have 20 minutes to get the result of the task. After that, the task will be deleted.

## Request
**Url params**
| Name | Type | Required | Description |
| ---- | ---- | -------- | ----------- |
| `id` | string | true | `id` from response of the `create_task` method |

## Response
### `200` OK
```
{
    "message": {
        {
            "id": <string>,
            "status": true,
            "code": 200,
            "done": <boolean>,
	    "data": <JSON object>,
            "file": <base64>
        }
    }
}
```
**Response params description**
| Name | Type | Description |
| ---- | ---- | ----------- |
| `id` | integer | `id` from response of the `create_task` method |
| `done` | boolean | `true` if document is ready; `false` if not ready yet |
| `data` | JSON | The object contains the data to generate, which was passed in the `create_task` method. If the parameter was `null` or not passed, the value selected during generation will be returned. This object return `true` if `done` is `true`, else return `null`. |
| `file` | base64 or null | Return base64 encoded image if `done` is `true`; else return `null` |

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
**Errors description**
| Reason | Description |
| ------ | ----------- |
| EMPTY_ID | ID is empty |

---

# `GET` /api/1.0/get_info?apikey=`<apikey>`
Get account's balance, member status, discount (percent) and prices.

## Response
### `200` OK
```
{
    "message": {
        "status": true,
        "code": 200,
        "balance": {
	    "main": <float>,
	    "referral": <float>
	},
        "member": <string>,
        "discount": <integer>,
        "prices": {
            "id": {
                "ua": <float>,
                -------------
            },
            "passport": {
                "ru": <float>,
                -------------
            }
        }
    }
}
```

---

# Common Errors
### `401` Unauthorized Error
Methods: `create_task`, `get_info`.
```
{
    "message": {
        "status": false,
        "code": 500,
        "reason": "BAD_APIKEY",
        "description": "API key is wrong or empty"
    }
}
```

### `500` Internal Server Error
```
{
    "message": {
        "status": false,
        "code": 500,
        "reason": "UNKNOWN_ERROR",
        "description": "Unknown error"
    }
}
```

### `429` Too Many Requests
```
{
    "message": {
	"status": false,
	"code": 429,
	"reason": "REQUESTS_LIMIT",
	"description": "Number of allowed requests exceeded"
    }
}
```
All API requests are limited to `2 per second` and `120 per minute`. If you see this status code, you need to increase delays between requests. You can get more info in following response headers: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset` and `Retry-After`.

### Unexpected Errors
```
{
    "message": {
    	"status": false,
	"code": <integer>
    }
}
```
