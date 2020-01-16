# Noumena pay OpenAPI

* [API Specifications](#1-api-specifications)
* [1.Noumena Pay API](#2-noumena-pay-api)

## API Specifications

- API requests use `HMAC` authentication.

- **Pagination** - Query record lists are all divided into pages, Pagination parameters: `page_num` represents the page number, `page_size` represents the size of each page. API `DTO` uniformly returns `total`, `records`.

- **Country** - Two digit country codes, refer to `ISO 3166-1 alpha-2` standards.

- Time management - API requests and responses return a `UNIX` timestamp, unit being **seconds**, in order to avoid issues due to regional time differences.

- Amount management - All API requests and responses are of the `string` data type in order to avoid precision loss.

- All the requests that have a `body` but don't explicitly define a format are of `JSON` type, `Content-Type: application/json`

- API response format standard-

  | Parameter |  Type  |                         Description                          |
  | :--------: | :----: | :----------------------------------------------------------: |
  |    code    |  int   |          Error code. `0`: Normal, non-`0`: Abnormal          |
  |    msg     | string | `SUCCESS` indicates success, error code indicates and describes failure |
  |   result   | object |                        Result                                |

### HMAC Authentication

The institution first needs to apply for the API `key` and API `secret` that will be used when accessing the API.

| Term                   | Description                                                  |
| ---------------------- | ------------------------------------------------------------ |
| User ID                | User ID is used to indicate the developer account, is used as the user ID |
| API key and API secret | Multiple API key + API secret maintained under a User ID, API key is linked with an application, multiple applications are allowed, each application can apply for API access privileges |

#### Client side implementation process:

1. Compose the data that needs to be signed, including-
   - UNIX timestamp, unit being `milliseconds`: the `request` time stamp 
   - Request method: `HTTP` method
   - Request API key： API Key
   - Complete request path, including the `URL` parameters: request URI
   - If there is a request `body`, the post conversion `string` of the `body` also needs to be added: string representation of the request payload
2. Client side generates the signature using `HMAC_SHA256` based on the data and API secret
3. Set the Authorization header based on the fixed sequence, i.e. the key is `Authorization`, and the value is: Noumena:ApiKey:request time stamp:signature (linked using colon) 
4. If the server side sets a password when creating the API key and secret, then an Access-Passphrase header needs to be set, i.e., the `key` is `Access-Passphrase`, and the `value` is the password.
5. Client side sends the data, Authorization header, and the Access-Passphrase header (in case there is a fourth step) to the server side, i.e., the final http header sent is as follows:
   - Authorization：Noumena:ApiKey:request timestamp:signature
   - Access-Passphrase：Your API Secret passphrase


#### How to build the request body string to be signed:

The parameter names of the request body need to be based on the respective `ASCII` values, pair key and value using `=`, and connect multiple key-value pairs using `&` to form a string.

Here is an example `body`-

```json
{
	"ont_id":"did:ont:Ae9ujqUnAtH9yRiepRvLUE3t9R2NbCTZPG",
	"amount":190,
	"to_address":"AUol16ghiT9AtxRDtNeq3ovhWJ5iaY6iyd"
}
```

The payload is converted to-

```java
amount=190&ont_id=did:ont:Ae9ujqUnAtH9yRiepRvLUE3t9R2NbCTZPG&to_address=AUol16ghiT9AtxRDtNeq3ovhWJ5iaY6iyd
```


#### Code Implementation and Examples:

[https://github.com/noumenapay/noumena-sdk-java](https://github.com/noumenapay/noumena-sdk-java)


## 1. Noumena Pay API

### 1.1. Deposit funds for a user


```text
url：/api/v1/npay/cust/transaction
method：POST
```


- Request:

| Body_Field_Name |  Type  |   Description   |
|:----------:|:------:|:---------------------------------------------------------------------:|
|   acct_no | String |User account number|
|   cust_user_no | String |User institution level ID (unique)|
|   cust_tx_id | String |Institution level transaction ID (can be blank)|
|   coin_type | String |Deposit currency type|
|   tx_amount | String |Deposit amount|
|   bonus_coin_type | String |Bonus currency type (can be blank)|
|   bonus_tx_amount | String |Bonus amount (can be blank if `bonus_coin_type` is left empty)|
|   remark | String |Remarks (optional)|



- Response:

```json
{
  "code": 0,
  "msg": "SUCCESS",
  "result": {
		"tx_id": "202001120001",
		"bonus_txid": "202001120002"
    }
}
```

|  Parameter   |  Type  |        Description         |
| :-----------: | :----: | :------------------------: |
|    tx_id    | String | Funds transaction ID|
|    bonus_txid    | String | Bonus transaction ID |






### 1.2. Fetch user's transaction records

```text
url：/api/v1/npay/cust/transaction
method：GET
```

- Request:

|  Parameter   |  Type  |        Description         |
| :-----------: | :----: | :------------------------: |
|  page_num   | int  |    Page number     |
|  page_size  | int  |  Page size   |
|  acct_no  | String  |  ONTO account number (optional)   |
|  cust_user_no  | String  | Linked institution ID (optional)    |
|  cust_tx_id  | String  | Linked transaction ID (optional)   |

- Response:

```json
	{
	  "code": 0,
	  "msg": "SUCCESS",
	  "result": {
		"records": [
            {
                "acct_no": "ont:did:AYUkLqCtozedCQrzLMXZiXq1wjr6Qm6Cj5",
                "cust_user_no": "mid-zx",
                "cust_tx_id": "12346",
                "cust_id": 13,
                "coin_type": "PAX",
                "tx_amount": "1.01",
                "bonus": "1.001",
                "bonus_coin_type": "ONT",
                "create_time": 1578889620000
            }
		],
		"total": 0
	  }
	}
```

|  Parameter   |  Type  |        Description         |
| :-----------: | :----: | :------------------------: |
|    acct_no    | String | User institution level ID (unique)|
| bonus | String | Bonus |
| bonus_coin_type | String | Bonus currency type |
|  create_time   | long |      Creation time   |
|    cust_user_no    |  String   |   Linked institution ID          |
|    cust_tx_id    |  String   |   Linked transaction ID          |
|  coin_type   | String |      Currency type  |
|  tx_amount   | String |      Transaction amount   |






