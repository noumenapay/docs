# Noumena-OpenAPI Interface

* [API Specifications](#api-specifications)
* [Customers API](#customers)
* [DebitCards API](#debit-cards)
* [Transactions API](#transactions)
* [Public API](#public-api)

## API Specifications

- API requests use `HMAC` authentication.

- **Pagination** - Query record lists are all divided into pages, Pagination parameters: `page_num` represents the page number, `page_size` represents the size of each page. API `DTO` uniformly returns `total`, `records`.

- **Country** - Two digit country codes, refer to `ISO 3166-1 alpha-2` standards.

- Time management - API requests and responses return a `UNIX` timestamp, unit being **seconds**, in order to avoid issues due to regional time differences.

- Amount management - All API requests and responses are of the `string` data type in order to avoid precision loss.

- All the requests that have a `body` but don't explicitly define a format are of `JSON` type, `Content-Type: application/json`

- API response format standard-

  | Field_Name |  Type  |                         Description                          |
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



## Customers


### 1.1 Submitting user's KYC data

```text
url：/api/v1/customers/accounts
method：POST
```

- Request:

|     Body_Field_Name      |  Type  |                         Description                          |
| :----------------------: | :----: | :----------------------------------------------------------: |
|         acct_no          | String |      Account serial no. (Unique within the institution)      |
|        acct_name         | String |                   Institution account name                   |
|        real_name         | String |                       Real legal name                        |
|           sex            | String |          male: Male，female: Female，unknown: other          |
|         birthday         | String |                          Birth date                          |
|         country          | String | Two digit country code，refer to `ISO 3166-1 alpha-2` standards |
|          doc_no          | String |                       Document number                        |
|         doc_type         | String | Document type. passport：Passport，idcard：National ID card，driving_license：Driving Licence |
|        front_doc         | String |             Front face picture. Base64 encoding              |
|         back_doc         | String |              Back face picture. Base64 encoding              |
|         mix_doc          | String |           Other clicked pictures. Base64 encoding            |
|        area_code         | String |                   International area code                    |
|          mobile          | String |                        Mobile number                         |
|           mail           | String |                        Email address                         |
|         address          | String |                        Postal address                        |
|         kyc_info         | String |                    Other KYC information                     |
|  mail_verification_code  | String |                   Email verification code                    |
| mobile_verification_code | String |                    SMS verification code                     |


- Response:

```json
{
  "code": 0,
  "msg": "string",
  "result": true
}
```



### 1.2. Querying KYC status
```text
url：/api/v1/customers/accounts/{acct_no}/kyc-status
method：GET
```

- Request:

| URL_Field_Name |  Type  |                         Description                          |
| :------------: | :----: | :----------------------------------------------------------: |
|    acct_no     | String | Institution account name (Unique within scope of the institution) |


- Response:

```json
{
  "code": 0,
  "msg": "string",
  "result": {
      "status":1,
      "reason":""
  }
}
```

| Field_Name |  Type  |                         Description                          |
| :--------: | :----: | :----------------------------------------------------------: |
|   reason   | String | Reason for verification failure. Blank for status other than failure |
|   status   |  int   | KYC status. 0：Under review，1：Verification successful，2：Verification failed |


### 1.3 Query all KYC records

```text
url：/api/v1/customers/accounts
method：GET
```
- Request:

| Field_Name  | Type |       Description       |
| :---------: | :--: | :---------------------: |
|  page_num   | int  |       Page number       |
|  page_size  | int  |        Page size        |
| former_time | long | Time period upper limit |
| latter_time | long | Time period lower limit |


- Response:

```json
{
    "code": 0,
    "msg": "SUCCESS",
    "result": {
        "total": 1,
        "records": [
            {
                "acct_no": "1222",
                "status": 1,
                "create_time": 1546300800000
            }
        ]
    }
}
```


| Field_Name  |  Type  |                         Description                          |
| :---------: | :----: | :----------------------------------------------------------: |
|   acct_no   | String | Institution account name (Unique within scope of the institution) |
|   status    |  int   | Status code : 0 - Submitted successfully, 1 - Verification successful (Account activated), 2 - Verification failed |
| create_time |  long  |                        Creation time                         |


### 1.4 Query a specific user's KYC records

```text
url：/api/v1/customers/accounts/{acct_no}
method：GET
```

- Request:

| URL_Field_Name |  Type  | Description                    |
| :------------: | :----: | :----------------------------- |
|    acct_no     | String | User's unique institutional ID |
|    page_num    |  int   | Page number                    |
|   page_size    |  int   | Page size                      |
|  former_time   |  long  | Time period upper limit        |
|  latter_time   |  long  | Time period lower limit        |


- Response:

```json
{
    "code": 0,
    "msg": "SUCCESS",
    "result": {
        "total": 1,
        "records": [
            {
                "acct_no": "1222",
                "status": 1,
                "create_time": 1546300800000
            }
        ]
    }
}
```


| Field_Name  |  Type  |                         Description                          |
| :---------: | :----: | :----------------------------------------------------------: |
|   acct_no   | String | Institution account name (Unique within scope of the institution) |
|   status    |  int   | Status code : 0 - Submitted successfully, 1 - Verification successful (Account activated), 2 - Verification failed |
| create_time |  long  |                        Creation time                         |



## Debit Cards

### 2.1. Submitting user's card information

```text
url：/api/v1/debit-cards
method：POST
```

- Request:

| Body_Field_Name |  Type  |                         Description                          |
| :-------------: | :----: | :----------------------------------------------------------: |
|     acct_no     | String | Institution account name (Unique within scope of the institution) |



- Response:

```json
{
  "code": 0,
  "msg": "string",
  "result": {
  	"card_no":"xxxx"
  }
}
```

| Field_Name |  Type  |     Description     |
| :--------: | :----: | :-----------------: |
|  card_no   | String | Allocated bank card |



### 2.2. User activating bank card

```text
url：/api/v1/debit-cards/status
method：PUT
```

- Request:

| Body_Field_Name |  Type  |                         Description                          |
| :-------------: | :----: | :----------------------------------------------------------: |
|     acct_no     | String | Institution account name (Unique within scope of the institution) |
|     card_no     | String |                        Bank card no.                         |



- Response:

```json
{
  "code": 0,
  "msg": "string",
  "result": true
}
```


### 2.3. User triggers a card withdrawal password reset Email

An institution invokes the API triggering the action that sends the bank card withdrawal password reset Email to the user's Email account.

```text
url：/api/v1/debit-cards/deposit-pwd-emails?acct_no={acct_no}
method：POST
```

- Request:

| Body_Field_Name |  Type  |  Description  |
| :-------------: | :----: | :-----------: |
|     card_no     | String | Bank card no. |



| URL_Field_Name |  Type  |                         Description                          |
| :------------: | :----: | :----------------------------------------------------------: |
|    acct_no     | String | Institution account name (Unique within scope of the institution) |
|    card_no     | String |                        Bank card no.                         |

- Response:

```json
{
  "code": 0,
  "msg": "string",
  "result": true
}
```

### 2.4 Query all active card records

```text
url：/api/v1/debit-cards
method：GET
```

| Field_Name  | Type |       Description       |
| :---------: | :--: | :---------------------: |
|  page_num   | int  |       Page number       |
|  page_size  | int  |        Page size        |
| former_time | long | Time period upper limit |
| latter_time | long | Time period lower limit |

- Response:

```json
{
    "code": 0,
    "msg": "SUCCESS",
    "result": {
        "total": 1,
        "records": [
            {
                "acct_no": "1",
                "card_no": "4385211202597301",
                "status": 1,
                "create_time": 1576847136000
            }
        ]
    }
}
```


| Field_Name  |  Type  |                         Description                          |
| :---------: | :----: | :----------------------------------------------------------: |
|   acct_no   | String | Institution account name (Unique within scope of the institution) |
|   card_no   |  int   |                         Card number                          |
|   status    |  int   | Status code : 0 - Frozen, 1 - Activated successfully, 2 - Not active |
| create_time |  long  |                        Creation time                         |


### 2.5 Query a specific user's card activation records

```text
url：/api/v1/debit-cards?acct_no={acct_no}
method：GET
```

- Request:

| URL_Field_Name |  Type  | Description                    |
| :------------: | :----: | :----------------------------- |
|    acct_no     | String | User's unique institutional ID |
|    page_num    |  int   | Page number                    |
|   page_size    |  int   | Page size                      |
|  former_time   |  long  | Time period upper limit        |
|  latter_time   |  long  | Time period lower limit        |


- Response:

```json
{
    "code": 0,
    "msg": "SUCCESS",
    "result": {
        "total": 1,
        "records": [
            {
                "acct_no": "1",
                "card_no": "4385211202597301",
                "status": 1,
                "create_time": 1576847136000
            }
        ]
    }
}
```

| Field_Name  |  Type  |                         Description                          |
| :---------: | :----: | :----------------------------------------------------------: |
|   acct_no   | String | Institution account name (Unique within scope of the institution) |
|   card_no   |  int   |                         Card number                          |
|   status    |  int   | Status code : 0 - Frozen, 1 - Activated successfully, 2 - Not active |
| create_time |  long  |                        Creation time                         |



## Transactions

### 3.1. User deposit

```text
url：/api/v1/deposit-transactions
method：POST
```

- Request:

| Body_Field_Name |  Type  |                         Description                          |
| :-------------: | :----: | :----------------------------------------------------------: |
|     card_no     | String |                        Bank card no.                         |
|     acct_no     | String | Institution account name (Unique within scope of the institution) |
|     amount      | String |           Deposit amount in corresponding currency           |
|    coin_type    | String |           Currency. Only USDT supported in Phase 1           |
|   cust_tx_id    | String |                  Institution transaction ID                  |
|     remarks     | String |                     Transaction remarks                      |

- Response:

```json
{
  "code": 0,
  "msg": "string",
  "result": true
}
```


### 3.2. Query a deposit transaction status

```text
url：/api/v1/deposit-transactions/{tx_id}/status
method：GET
```

- Request：

| URL_Field_Name |  Type  | Description    |
| :------------: | :----: | :------------- |
|     tx_id      | String | Transaction ID |

- Response：

```json
{
  "code": 0,
  "msg": "string",
  "result": {
  	"tx_status": //0:process pending，1: deposit successful，2: deposit failed
  }
}
```


### 3.3  Query all the deposit records

```text
url：/api/v1/deposit-transactions
method：GET
```

| Field_Name  | Type |       Description       |
| :---------: | :--: | :---------------------: |
|  page_num   | int  |       Page number       |
|  page_size  | int  |        Page size        |
| former_time | long | Time period upper limit |
| latter_time | long | Time period lower limit |

- Response:

```json
{
    "code": 0,
    "msg": "SUCCESS",
    "result": {
        "total": 1,
        "records": [
            {
                "acct_no": "1",
                "card_no": "",
                "tx_amount": "1",
                "usd_amount": "1",
                "exchange_rate": "1",
                "bank_tx_id": "",
                "fee": "1",
                "create_time": null
            }
        ]
    }
}
```

|  Field_Name   |  Type  |                         Description                          |
| :-----------: | :----: | :----------------------------------------------------------: |
|    acct_no    | String | Institution account name (Unique within scope of the institution) |
|    card_no    |  int   |                         Card number                          |
|   tx_amount   | String |                      Transaction amount                      |
|  usd_amount   | String |                  Transaction amount in USD                   |
| exchange_rate | String |                        Exchange rate                         |
|  bank_tx_id   | String |                     Bank transaction ID                      |
|      fee      | String |                       Transaction fee                        |
|  create_time  |  long  |                        Creation time                         |

### 3.4 Query a particular user's deposit records

```text
url：/api/v1/deposit-transactions?acct_no={acct_no}
method：GET
```

- Request:

| URL_Field_Name |  Type  | Description                    |
| :------------: | :----: | :----------------------------- |
|    acct_no     | String | User's unique institutional ID |
|    page_num    |  int   | Page number                    |
|   page_size    |  int   | Page size                      |
|  former_time   |  long  | Time period upper limit        |
|  latter_time   |  long  | Time period lower limit        |


- Response:

```json
{
    "code": 0,
    "msg": "SUCCESS",
    "result": {
        "total": 1,
        "records": [
            {
                "acct_no": "1",
                "card_no": "",
                "tx_amount": "1",
                "usd_amount": "1",
                "exchange_rate": "1",
                "bank_tx_id": "",
                "fee": "1",
                "create_time": null
            }
        ]
    }
}
```

|  Field_Name   |  Type  |                         Description                          |
| :-----------: | :----: | :----------------------------------------------------------: |
|    acct_no    | String | Institution account name (Unique within scope of the institution) |
|    card_no    |  int   |                         Card number                          |
|   tx_amount   | String |                      Transaction amount                      |
|  usd_amount   | String |                  Transaction amount in USD                   |
| exchange_rate | String |                        Exchange rate                         |
|  bank_tx_id   | String |                   Bank transaction records                   |
|      fee      | String |                       Transaction fee                        |
|  create_time  |  long  |                        Creation time                         |


## Public API


### 4.1. Sending Email verification code

```text
url：/api/v1/emails/{email}/verification-codes
method：POST
```

- Request:

| URL_Field_Name |  Type  |  Description  |
| :------------: | :----: | :-----------: |
|     email      | String | Email address |

- Response:

```json
{
  "code": 0,
  "msg": "string",
  "result": {
  	"verification_id":"xxxxxx"
  }
}
```

|   Field_Name    |  Type  |        Description        |
| :-------------: | :----: | :-----------------------: |
| verification_id | String | Allocated verification ID |



### 4.2. Sending SMS verification code

```text
url：/api/v1/mobiles/{mobile}/verification-codes
method：POST
```

- Request:

| URL_Field_Name |  Type  |        Description        |
| :------------: | :----: | :-----------------------: |
|     mobile     | String | Area code + Mobile number |

- Response:

```json
{
  "code": 0,
  "msg": "string",
  "result": {
  	"verification_id":"xxxxxx"
  }
}
```

|   Field_Name    |  Type  |        Description        |
| :-------------: | :----: | :-----------------------: |
| verification_id | String | Allocated verification ID |



### 4.3. Email verification code validation

```text
url：/api/v1/emails/{email}/verification-codes?code={code}&verification_id={verification_id}
method：PUT
```

- Request:

| URL_Field_Name  |  Type  |        Description        |
| :-------------: | :----: | :-----------------------: |
|      email      | String |          E-mail           |
|      code       | String |           Code            |
| verification_id | String | Allocated verification ID |

- Response:

```json
{
  "code": 0,
  "msg": "string",
  "result": true
}
```



### 4.4. SMS verification code validation

```text
url：/api/v1/mobiles/{mobile}/verification-codes?code={code}&verification_id={verification_id}
method：PUT
```

- Request:

| URL_Field_Name  |  Type  |        Description        |
| :-------------: | :----: | :-----------------------: |
|     mobile      | String | Area code + Mobile number |
|      code       | String |           Code            |
| verification_id | String | Allocated verification ID |

- Response:

```json
{
  "code": 0,
  "msg": "string",
  "result": true
}
```