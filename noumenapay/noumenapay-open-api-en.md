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



## 1.1. Get institution balance

```text
url：/api/v1/npay/cust/balance
method：GET
```

- Request：


- Response：

```json
	{
	  "code": 0,
	  "msg": "string",
	  "result": {
        "total": 2,
        "records": [
            {
                "coin_type": "ONT",
                "balance": "0.992"
            },
            {
                "coin_type": "PAX",
                "balance": "2.04"
            }
        ]
	  }
	}
```
|  Field_Name   |  Type  |        Description         |
| :-----------: | :----: | :------------------------: |
|  coin_type   | String |     coin type   |
|  balance   | String |      balance   |



### 1.2.Get transaction history

Transaction history include deposit and withdraw transactions.

```text
url：/api/v1/npay/cust/transaction
method：GET
```

- Request：

|  Field_Name   |  Type  | Whether Required |       Description         |
| :-----------: | :----: | :-----------------: | :-----------: |
|  page_num   | int  |  Required |  page number     |
|  page_size  | int  |  Required | page size   |
|  acct_no  | String  |  Optional | User account number in Institution  |
|  cust_tx_id  | String  | Optional | transaction ID in Institution   |
|  tx_id  | String  | Optional | transaction ID   |

- Response：

```json
	{
	  "code": 0,
	  "msg": "SUCCESS",
	  "result": {
		"records": [
		  {
			"acct_no": "12345678",
			"acct_name": "user name",
			"cust_tx_id":"1",
			"coinType":"PAX",
			"tx_amount": "126",
			"address": "",
			"bonus": "10",
			"bonus_coin_type": "ONT",	
		    "tx_id": "2020042218173801700000002",
		    "tx_type": "deposit",
		    "tx_status": 1,
            "create_time": 1587886601000,
            "update_time": 1587894841000
		  }
		],
		"total": 1
	  }
	}
```
|  Field_Name   |  Type  |        Description         |
| :-----------: | :----: | :------------------------: |
|    acct_no    | String | User account number in Institution|
|    acct_name    | String | User account name in Institution |
|    cust_tx_id    |  String   |   transaction ID in Institution          |
|  coin_type   | String |      coin type   |
|  tx_amount   | String |      amount   |
|   address | String |   address |
| bonus | String | bonus |
| bonus_coin_type | String | bonus coin type |
| tx_id | String | transation ID |
|  tx_type   | String |      transaction type：deposit or withdraw   |
| tx_status | int | transaction status. 0 pending， 1 successful， 2 failure |
|  create_time   | long |      create time   |
|  update_time   | long |      update time   |



### 1.2.Deposit for User


```text
url：/api/v1/npay/cust/deposit
method：POST
```


- Request：

| Body_Field_Name |  Type  |  Whether Required |    Description   |
|:----------:|:------:| :-----------------:| :-----------------:|
|   acct_no | String | Required |  User account number in Institution|
|   acct_name | String | Required |  User account name in Institution|
|   cust_tx_id | String | Optional |  transaction ID in Institution |
|   coin_type | String | Required |  coin type|
|   tx_amount | String | Required |  coin amount |
|   bonus_coin_type | String | Optional |  bonus coin type|
|   bonus_tx_amount | String | Optional |  bonus amount |
|   remark | String | Optional |  remark|



- Response：

```json
{
  "code": 0,
  "msg": "SUCCESS",
  "result": {
		"tx_id": "202001120001"
    }
}
```

|  Field_Name   |  Type  |        Description         |
| :-----------: | :----: | :------------------------: |
|    tx_id    | String | transaction ID|



### 1.3.Get User deposit history

```text
url：/api/v1/npay/cust/deposit
method：GET
```

- Request：

|  Field_Name   |  Type  | Whether Required |       Description         |
| :-----------: | :----: | :-----------------: | :-----------: |
|  page_num   | int  |  Required |  page number     |
|  page_size  | int  |  Required | page size   |
|  acct_no  | String  |  Optional | User account number in Institution  |
|  cust_tx_id  | String  | Optional | transaction ID in Institution  |
|  tx_id  | String  | Optional | transaction ID   |

- Response：

```json
	{
	  "code": 0,
	  "msg": "SUCCESS",
	  "result": {
		"records": [
		  {
			"acct_no": "12345678",
			"acct_name": "user name",
			"cust_tx_id":"1",
			"coinType":"PAX",
			"tx_amount": "126",
			"address": "",
			"bonus": "10",
			"bonus_coin_type": "ONT",	
		    "tx_id": "2020042218173801700000002",
		    "tx_type": "deposit",
		    "tx_status": 1,
            "create_time": 1587886601000,
            "update_time": 1587894841000
		  }
		],
		"total": 1
	  }
	}
```
|  Field_Name   |  Type  |        Description         |
| :-----------: | :----: | :------------------------: |
|    acct_no    | String | User account number in Institution |
|    acct_name    | String | User account name in Institution |
|    cust_tx_id    |  String   |   transaction ID in Institution          |
|  coin_type   | String |      coin type   |
|  tx_amount   | String |      coin amount   |
|   address | String |   address |
| bonus | String | bonus |
| bonus_coin_type | String | bonus coin type |
| tx_id | String | transaction ID |
|  tx_type   | String |      transaction type：deposit or withdraw   |
| tx_status | int | transaction status. 0 pending， 1 successful， 2 failure |
|  create_time   | long |      create time   |
|  update_time   | long |      update time   |


### 1.4.User withdraw


```text
url：/api/v1/npay/cust/withdrawal
method：POST
```


- Request：

| Body_Field_Name |  Type  |  Whether Required |  Description   |
|:----------:|:------:| :------:| :--------------:|
|   acct_no | String |  Required |  User account number in Institution|
|   cust_tx_id | String |  Optional |  transaction ID in Institution |
|   coin_type | String | Required |  coin type |
|   address | String |  Required |  User coin_type address |
|   tx_amount | String |  Required |  amount |
|   remark | String | Optional |  remark|



- Response：

```json
{
  "code": 0,
  "msg": "SUCCESS",
  "result": {
			"tx_id": "202001120001"
    }
}
```
|  Field_Name   |  Type  |        Description         |
| :-----------: | :----: | :------------------------: |
|    tx_id    | String | transaction ID |



### 1.5.User withdraw history

```text
url：/api/v1/npay/cust/withdrawal
method：GET
```

- Request：

|  Field_Name   |  Type  |  Whether Required |        Description         |
| :-----------: | :----: | :------------------------: | :------------------------: |
|  page_num   | int  |  Required |  page number     |
|  page_size  | int  | Required | page size   |
|  acct_no  | String  | Optional | User account number in Institution  |
|  cust_tx_id  | String  | Optional | transaction ID in Institution  |
|  tx_id  | String  | Optional | transaction ID   |

- Response：

```json
	{
	  "code": 0,
	  "msg": "SUCCESS",
	  "result": {
        "total": 1,
        "records": [
            {
                "acct_no": "ont:did:test2",
                "acct_name": "",
                "tx_type": "withdraw",
                "cust_tx_id": "id-123456789023",
                "tx_id": "2020042218071501700000001",
                "coin_type": "ONT",
                "tx_amount": "0.1",
                "address": "",
                "bonus": "0",
                "bonus_coin_type": "",
                "tx_status": 1,
                "create_time": 1587886601000,
                "update_time": 1587894841000
            }
        ]
	  }
	}
```
|  Field_Name   |  Type  |        Description         |
| :-----------: | :----: | :------------------------: |
|    acct_no    | String | User account number in Institution |
|    acct_name    | String | empty |
|    cust_tx_id    |  String   |   transaction ID in Institution          |
|  coin_type   | String |      coin type   |
|  tx_amount   | String |      coin amount   |
| bonus | String | "0" |
| bonus_coin_type | String | empty |
| tx_id | String | transaction ID |
|  tx_type   | String |      transaction type: deposit or withdraw   |
| tx_status | int | transaction status. 0 pending， 1 successful， 2 failure |
|   address | String |   coin_type address |
|  create_time   | long |      create time   |
|  update_time   | long |      update time   |



### 1.6.Get user balance

```text
url：/api/v1/npay/cust/user/balance?acct_no=123&coin_type=USDT
method：GET
```

- Request：

|  Field_Name   |  Type  | Whether Required |        Description         |
| :-----------: | :----: | :------------------------: | :------------------------: |
|  acct_no  | String  | Required | User account number in Institution  |
|  coin_type  | String  | Optional | Coin type    |

- Response：

```json
	{
	  "code": 0,
	  "msg": "string",
	  "result": {
        "total": 2,
        "records": [
            {
                "coin_type": "ONT",
                "balance": "0.791",
                "deposit": "1.312",
                "withdraw": "2.301"
            },
            {
                "coin_type": "USDT",
                "balance": "2",
                "deposit": "1",
                "withdraw": "0"
            }
        ]
	  }
	}
```
|  Field_Name   |  Type  |        Description         |
| :-----------: | :----: | :------------------------: |
|  coin_type   | String |     Coin 类型   |
|  balance   | String |      balance   |
|  deposit   | String |      total deposit   |
|  withdraw   | String |     total withdraw   |









