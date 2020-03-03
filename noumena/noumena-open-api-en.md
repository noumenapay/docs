# Noumena-OpenAPI Interface

- [API Specifications](#api-specifications)
- [1.Customers API](#1-customers)
- [2.Debit Cards API](#2-debit-cards)
- [3.Transactions API](#3-transactions)
- [4.Public API](#4-public-api)
- [5.Bank Account API](#5-bank-account-api)
- [6.Error Codes](#6-error-codes)

## API Specifications

- API requests use `HMAC` authentication.

- **Pagination** - Query record lists are all divided into pages, Pagination parameters: `page_num` represents the page number, `page_size` represents the size of each page. API `DTO` uniformly returns `total`, `records`.

- **Country** - Two digit country codes, refer to `ISO 3166-1 alpha-2` standards.

- Time management - API requests and responses return a `UNIX` timestamp, unit being **seconds**, in order to avoid issues due to regional time differences.

- Amount management - All API requests and responses are of the `string` data type in order to avoid precision loss.

- All the requests that have a `body` but don't explicitly define a format are of `JSON` type, `Content-Type: application/json`

- The query interval of all query interfaces must be **less than one month**

- API response format standard-

  | Parameter  |  Type  |                               Description                               |
  | :----: | :----: | :---------------------------------------------------------------------: |
  |  code  |  int   |               Error code. `0`: Normal, non-`0`: Abnormal                |
  |  msg   | string | `SUCCESS` indicates success, error code indicates and describes failure |
  | result | object |                                 Result                                  |

### HMAC Authentication

The institution first needs to apply for the API `key` and API `secret` that will be used when accessing the API.

| Term                   | Description                                                                                                                                                                              |
| ---------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| User ID                | User ID is used to indicate the developer account, is used as the user ID                                                                                                                |
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
  "ont_id": "did:ont:Ae9ujqUnAtH9yRiepRvLUE3t9R2NbCTZPG",
  "amount": 190,
  "to_address": "AUol16ghiT9AtxRDtNeq3ovhWJ5iaY6iyd"
}
```

The payload is converted to-

```text
amount=190&ont_id=did:ont:Ae9ujqUnAtH9yRiepRvLUE3t9R2NbCTZPG&to_address=AUol16ghiT9AtxRDtNeq3ovhWJ5iaY6iyd
```

#### Code Implementation and Examples:

For illustrative sample code, please refer to [https://github.com/noumenapay/noumena-sdk-java](https://github.com/noumenapay/noumena-sdk-java)

## 1. Customers

This API contains methods that can be used by an institution to carry out user-related operations such as user creation and KYC, along with fetching relevant KYC details for users, etc.

### 1.1 Submitting user's KYC data

Email verification feature is optional. The verification code status is updated to used when verification is successfully carried out.

- Request:

```text
url：/api/v1/customers/accounts
method：POST
```
> How to fill Card_type_id (bank card type ID) ? Please use the 4.5 interface to query.


|       Parameter        |  Type  | Whether Required |                                          Description                                          |
| :--------------------: | :----: | :--------------: | :-------------------------------------------------------------------------------------------: |
|        acct_no         | String |     Required     |         Account serial no. (Unique within the institution), Max. character length: 64         |
|       acct_name        | String |     Required     |                      Institution account name, Max. character length: 64                      |
|       first_name       | String |     Required     |                          Legal first name, Max. character length: 50                          |
|       last_name        | String |     Required     |                          Legal last name, Max. character length: 50                           |
|         gender         | String |     Required     |             male: Male，female: Female，unknown: other, Max. character length: 6              |
|        birthday        | String |     Required     |                                    Birth date (YYYY-MM-DD)                                    |
|          city          | String |     Required     |                               City, Max. character length: 100                                |
|         state          | String |     Required     |                               State, Max. character length: 100                               |
|        country         | String |     Required     |                              Country, Max. character length: 50                               |
|      nationality       | String |     Required     |                           Nationality，, Max. character length: 255                           |
|         doc_no         | String |     Required     |                                        Document number                                        |
|        doc_type        | String |     Required     | Document type. passport：Passport，idcard：National ID card |
|       front_doc        | String |     Required     |                              Front face picture. Base64 encoding. File size should be less than 2M  |
|        back_doc        | String |     Required     |                              Back face picture. Base64 encoding. File size should be less than 2M              |
|        mix_doc         | String |     Required     |                            Photo with certificate in hand. Base64 encoding. File size should be less than 2M |
|      country_code      | String |     Required     | International country code，for example "+86". Max. character length: 5 |
|         mobile         | String |     Required     |                           Mobile number, Max. character length: 32                            |
|          mail          | String |     Required     |                           Email address,don't support 163.com email host. Max. character length: 64                            |
|        address         | String |     Required     |                          Postal address, the bank card will send tothis address. Max. character length: 256                           |
|        zipcode         | String |     Required     |                              Zip code, Max. character length: 20                              |
|      maiden_name       | String |     Required      |                         Legal maiden name, Max. character length: 255                         |
| card_type_id |String |Required | Bank card type id, for example: 10010001|   
|        kyc_info        | String |     Optional     |                                     Other KYC information                                     |
| mail_verification_code | String |     Optional     |                                    Email verification code                                    |
|       mail_token       | String |     Optional     |                        Token returned upon sending verification Email                         |
| cust_tx_id            | String | Optional         | customer transaction id|

- Response:

```json
{
  "code": 0,
  "msg": "string",
  "result": true
}
```

> How to get mail_token and mail_verification_code? Please see "4.1. Sending Email verification code". mail_verification_code is filled by user.


### 1.2 Query all KYC records

- Request:

```text
url：/api/v1/customers/accounts
method：GET
```

|  Parameter  | Type  | Whether Required |                        Description                         |
| :---------: | :---: | :--------------: | :--------------------------------------------------------: |
|  page_num   |  int  |     Optional     |                        Page number                         |
|  page_size  |  int  |     Optional     |                         Page size                          |
| former_time | long  |     Optional     | Time period upper limit, `UNIX` timestamp, Unit: `seconds` |
| latter_time | long  |     Optional     | Time period lower limit, `UNIX` timestamp, Unit: `seconds` |
| time_sort | String | Optional| sort time. asc: time in ascending order，desc: time in descending order    |

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
        "card_type_id": "10010001",
        "status": 1,
        "reason": "",
        "create_time": 1546300800000
      }
    ]
  }
}
```

|    Parameter    |  Type   |                                                    Description                                                     |
| :---------: | :----:   | :----------------------------------------------------------------------------------------------------------------: |
|   acct_no   | String  |                         Institution account name (Unique within scope of the institution)                          |
| card_type_id |String | Card type id|   
|   status    |  int    | Status code : 0 - Submitted successfully, 1 - Verification successful (Account activated), 2 - Verification failed, 3 - Verifying, 4 - Submission in progress|
| reason | String |      Reason for verification failure. Blank for status other than failure       |
| create_time |  long   |                                                   Creation time                                                    |

### 1.3 Query a specific user's KYC records

- Request:

```text
url：/api/v1/customers/accounts
method：GET
```

|  Parameter  |  Type  | Whether Required | Description                                                |
| :---------: | :----: | :------------:  | :--------------------------------------------------------- |
|   acct_no   | String | Required | User's unique institutional ID                             |
| former_time |  long  | Optional | Time period upper limit, `UNIX` timestamp, Unit: `seconds` |
| latter_time |  long  | Optional | Time period lower limit, `UNIX` timestamp, Unit: `seconds` |

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
        "card_type_id": "10010001",
        "status": 1,
        "reason": "",
        "create_time": 1546300800000
      }
    ]
  }
}
```

|    Parameter    |  Type   |                                                    Description                                                     |
| :---------: | :----:   | :----------------------------------------------------------------------------------------------------------------: |
|   acct_no   | String  |                         Institution account name (Unique within scope of the institution)                          |
| card_type_id |String  | Card type id|   
|   status    |  int    | Status code : 0 - Submitted successfully, 1 - Verification successful (Account activated), 2 - Verification failed, 3 - Verifying, 4 - Submission in progress |
| reason | String |      Reason for verification failure. Blank for status other than failure       |
| create_time |  long   |                                                   Creation time                                                    |


## 2. Debit Cards

This API contains methods related to

### 2.1. Apply a card

- Request:

```text
url：/api/v1/debit-cards
method：POST
```

| Parameter |  Type  | Whether Required |                            Description                            |
| :-------: | :----: | :--------------: | :---------------------------------------------------------------: |
|  acct_no  | String |     Required     | Institution account name (Unique within scope of the institution) |
| card_type_id |String |Required | Card type id|   

- Response:

```json
{
  "code": 0,
  "msg": "string",
  "result": {
    "card_no": "xxxx",
    "card_number": "430021******1144"
  }
}
```

|  Parameter  |  Type  |     Description     |
| :-----: | :----: | :-----------------: |
| card_no | String | To prevent real card information from being exposed, use 'card_no' parameter to query  |
| card_number | String | Actual card number, Expose only the first 6 and last 4 |

### 2.2. User activating bank card

```text
url：/api/v1/debit-cards/status
method：PUT
```

- Request:

| Parameter |  Type  | Whether Required |                            Description                            |
| :-------: | :----: | :--------------: | :---------------------------------------------------------------: |
|  acct_no  | String |     Required     | Institution account name (Unique within scope of the institution) |
|  card_no  | String |     Required     |                           Bank card no.                           |

- Response:

```json
{
  "code": 0,
  "msg": "string",
  "result": true
}
```

### 2.3. User triggers a card withdrawal password reset Email (Currently not supported)

An institution invokes the Noumena API triggering the action that sends the bank card withdrawal password reset Email to the user's Email account.

- Request:

```text
url：/api/v1/debit-cards/deposit-pwd-emails?acct_no={acct_no}
method：POST
```

| Parameter |  Type  | Whether Required |                            Description                            |
| :-------: | :----: | :--------------: | :---------------------------------------------------------------: |
|  acct_no  | String |     Required     | Institution account name (Unique within scope of the institution) |
|  card_no  | String |     Required     |                           Bank card no.                           |

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

|    Parameter    | Type  | Whether Required |                        Description                         |
| :---------: | :---: | :--------------: | :--------------------------------------------------------: |
|  page_num   |  int  |     Required     |                        Page number                         |
|  page_size  |  int  |     Required     |                         Page size                          |
| former_time | long  |     Required     | Time period upper limit, `UNIX` timestamp, Unit: `seconds` |
| latter_time | long  |     Required     | Time period lower limit, `UNIX` timestamp, Unit: `seconds` |
| time_sort | String | Optional| sort time. asc: time in ascending order，desc: time in descending order    |

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

|    Parameter    |  Type  |                             Description                              |
| :---------: | :----: | :------------------------------------------------------------------: |
|   acct_no   | String |  Institution account name (Unique within scope of the institution)   |
|   card_no   |  int   |                             Card ID                              |
|   status    |  int   | Status code : 0 - Frozen, 1 - Activated successfully, 2 - Not active |
| create_time |  long  |                            Creation time                             |

### 2.5 Query a specific user's card activation records

- Request:

```text
url：/api/v1/debit-cards?acct_no={acct_no}
method：GET
```

|  Parameter  |  Type  | Whether Required | Description                                                |
| :---------: | :----: | :--------------: | :--------------------------------------------------------- |
|   acct_no   | String |     Required     | User's unique institutional ID                             |
|  page_num   |  int   |     Optional     | Page number                                                |
|  page_size  |  int   |     Optional     | Page size                                                  |
| former_time |  long  |     Optional     | Time period upper limit, `UNIX` timestamp, Unit: `seconds` |
| latter_time |  long  |     Optional     | Time period lower limit, `UNIX` timestamp, Unit: `seconds` |
| time_sort | String | Optional| sort time. asc: time in ascending order，desc: time in descending order    |

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

|    Parameter    |  Type  |                             Description                              |
| :---------: | :----: | :------------------------------------------------------------------: |
|   acct_no   | String |  Institution account name (Unique within scope of the institution)   |
|   card_no   |  int   |                             Card ID                              |
|   status    |  int   | Status code : 0 - Frozen, 1 - Activated successfully, 2 - Not active |
| create_time |  long  |                            Creation time                             |

## 3. Transactions

### 3.1. User deposit

- Request:

```text
url：/api/v1/deposit-transactions
method：POST
```

| Parameter  |  Type  | Whether Required |                            Description                            |
| :--------: | :----: | :--------------: | :---------------------------------------------------------------: |
|  card_no   | String |     Required     |                           Bank card no.                           |
|  acct_no   | String |     Required     | Institution account name (Unique within scope of the institution) |
|   amount   | String |     Required     |             Deposit amount in corresponding currency              |
| coin_type  | String |     Required     |             Only USDT supported in version v1.0.0              |
| cust_tx_id | String |     Required     |                    Institution transaction ID                     |
|  remarks   | String |     Optional     |                        Transaction remarks                        |

- Response:

```json
{
    "code": 0,
    "msg": "SUCCESS",
    "result": {
        "tx_id": "2020022511324811001637548",
        "currency_type": "USD",
        "deposit_usdt": "0.9188",
        "currency_amount": "0.92",
        "exchange_rate": "1.00239251357",
        "exchange_fee_rate": "0",
        "exchange_fee": "0",
        "loading_fee": "0.0812"
    }
}
```

| Parameter |  Type    | Description |
| :------------: | :----------: |:---------- |
|     tx_id      | String | Noumena transaction ID  |
|     currency_amount      | String | received currency amount  |
|     currency_type      | String | received currency type  |
|     exchange_rate      | String |  exchange rate of USDT/Fiat currency  |
|     loading_fee      | String | Deposit fee，Unit: USDT   |
|     exchange_fee      | String | Fee for exchanging digital coin to USDT, Unit: USDT  |
|     exchange_fee_rate      | String | Fee rate for exchanging digital coin to USDT   |
|     deposit_usdt      | String | The amount of USDT deposited for the user after charging loading_fee and exchange_fee, Unit: USDT   |

> USDT amount charged from customer = exchange_fee + loading_fee + deposit_usdt.


### 3.2. Query a deposit transaction status

```text
url：/api/v1/deposit-transactions/{tx_id}/status
method：GET
```

- Request：

| Parameter |  Type  | Whether Required | Description    |
| :-------: | :----: | :--------------: | :------------- |
|   tx_id   | String |     Required     | Noumena Transaction ID |

- Response：

```json
{
  "code": 0,
  "msg": "string",
  "result": {
  	"tx_status": 0
  }
}
```

| Parameter |  Type    | Description |
| :------------: | :----------: |:---------- |
|     tx_status      | int | 0 and 3:process pending，1: deposit successful，5：deposit failed  |

### 3.3 Query all the deposit records

```text
url：/api/v1/deposit-transactions
method：GET
```

|    Parameter    | Type  | Whether Required |                        Description                         |
| :---------: | :---: | :--------------: | :--------------------------------------------------------: |
|  page_num   |  int  |     Optional     |                        Page number                         |
|  page_size  |  int  |     Optional     |                         Page size                          |
| former_time | long  |     Optional     | Time period upper limit, `UNIX` timestamp, Unit: `seconds` |
| latter_time | long  |     Optional     | Time period lower limit, `UNIX` timestamp, Unit: `seconds` |
| time_sort | String | Optional| sort time. asc: time in ascending order，desc: time in descending order    |

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
        "currency_amount": "10",
        "currency_type": "USD",
        "exchange_rate": "1",
        "bank_tx_id": "",
        "loading_fee": "1",
        "cust_tx_time": null
      }
    ]
  }
}
```

|     Parameter     |  Type  |                            Description                            |
| :-----------: | :----: | :---------------------------------------------------------------: |
|    acct_no    | String | Institution account name (Unique within scope of the institution) |
|    card_no    |  int   |                            Card ID                            |
|   tx_amount   | String |                        Transaction amount                         |
|     currency_amount      | String | received currency amount  |
|     currency_type      | String | received currency type  |
| exchange_rate | String |                           Exchange rate                           |
|  bank_tx_id   | String |                        Bank transaction ID                        |
|      loading_fee      | String |                          Transaction fee                          |
| cust_tx_time  |  long  |                           Creation time                           |

### 3.4 Query a particular user's deposit records

```text
url：/api/v1/deposit-transactions?acct_no={acct_no}
method：GET
```

- Request:

|  Parameter  |  Type  | Whether Required | Description                                                |
| :---------: | :----: | :--------------: | :--------------------------------------------------------- |
|   acct_no   | String |     Required     | User's unique institutional ID                             |
|  page_num   |  int   |     Optional     | Page number                                                |
|  page_size  |  int   |     Optional     | Page size                                                  |
| former_time |  long  |     Optional     | Time period upper limit, `UNIX` timestamp, Unit: `seconds` |
| latter_time |  long  |     Optional     | Time period lower limit, `UNIX` timestamp, Unit: `seconds` |
| time_sort | String | Optional| sort time. asc: time in ascending order，desc: time in descending order    |

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
        "coin_type": "USDT",
        "tx_amount": "1",
        "currency_amount": "10",
        "currency_type": "USD",
        "exchange_rate": "1",
        "cust_tx_id": "1",
        "bank_tx_id": "",
        "loading_fee": "1",
        "cust_tx_time": null
      }
    ]
  }
}
```

|     Parameter     |  Type  |                            Description                            |
| :-----------: | :----: | :---------------------------------------------------------------: |
|    acct_no    | String | Institution account name (Unique within scope of the institution) |
|    card_no    |  int   |                            Card ID                            |
|   tx_amount   | String |                        Transaction amount                         |
|     currency_amount      | String | received currency amount  |
|     currency_type      | String | received currency type  |
| exchange_rate | String |                           Exchange rate                           |
|  bank_tx_id   | String |                     Bank transaction records                      |
|      loading_fee      | String |                          Transaction fee                          |
| cust_tx_time  |  long  |                           Creation time                           |

## 4. Public API

### 4.1. Sending Email verification code

- Request:

```text
url：/api/v1/emails/{email}/verification-codes
method：POST
```

| Parameter |  Type  | Whether Required |  Description  |
| :-------: | :----: | :--------------: | :-----------: |
|   email   | String |     Required     | Email address |

- Response:

```json
{
  "code": 0,
  "msg": "string",
  "result": {
  	"mail_token":"xxxxxx"
  }
}
```

|   Parameter    |  Type  | Description                  |
| :--------: | :----: | :--------------------------- |
| mail_token | String | Allocated verification token |


### 4.2. Email verification code validation

```text
url：/api/v1/emails/{email}/verification-codes?code={code}&mail_token={mail_token}
method：PUT
```

- Request:

| Parameter  |  Type  | Whether Required |        Description        |
| :--------: | :----: | :--------------: | :-----------------------: |
|   email    | String |     Required     |          E-mail           |
|    code    | String |     Required     |           Code            |
| mail_token | String |     Required     | Allocated verification ID |

- Response:

```json
{
  "code": 0,
  "msg": "string",
  "result": true
}
```

### 4.3. Sending SMS verification code (Currently not supported)

- Request:

```text
url：/api/v1/mobiles/{mobile}/verification-codes
method：POST
```

| Parameter |  Type  | Whether Required |        Description        |
| :-------: | :----: | :--------------: | :-----------------------: |
|  mobile   | String |     Required     | Area code + Mobile number |

- Response:

```json
{
  "code": 0,
  "msg": "string",
  "result": {
    "mobile_token": "xxxxxx"
  }
}
```

|    Parameter     |  Type  |         Description          |
| :----------: | :----: | :--------------------------: |
| mobile_token | String | Allocated verification token |

### 4.4. SMS verification code validation (Currently not supported)

```text
url：/api/v1/mobiles/{mobile}/verification-codes?code={code}&mobile_token={mobile_token}
method：PUT
```

- Request:

|  Parameter   |  Type  | Whether Required |        Description        |
| :----------: | :----: | :--------------: | :-----------------------: |
|    mobile    | String |     Required     | Area code + Mobile number |
|     code     | String |     Required     |           Code            |
| mobile_token | String |     Required     | Allocated verification ID |

- Response:

```json
{
  "code": 0,
  "msg": "string",
  "result": true
}
```




### 4.5 Querying Card type

```text
url：/api/v1/card/type
method：GET
```

- Response:

```json
{
    "code": 0,
    "msg": "SUCCESS",
    "result": {
        "total": 2,
        "records": [
            {
                "card_type_id": "50000001",
                "currency_type": "USD",               
                "bank_id": "5000"
            },
            {
                "card_type_id": "50000002",
                "currency_type": "USD",               
                "bank_id": "5000"
            }
        ]
    }
}
```

| Parameter |  Type  |          Description          |
| :--------: | :----: | :------------------------------ |
|   bank_id   | String |        Bank ID              |
|   currency_type   | String |    Currency type of the card             |
| card_type_id |String | Card type id, for example 50000001|


### 4.6 Querying rate

```text
url：/api/v1/rates?card_type_id={card_type_id}
method：GET
```

- Request：

| Parameter |  Type  |   Requirement  | Description   |
| :------------: | :----: | :----------: |:---------- |
| card_type_id | String |Required| Card type id, for example 50000001|

- Response：

```json
{
  "code": 0,
  "msg": "string",
  "result": {
    "open_card_fee_usdt":"20",
    "exchange_rate":"1.12",
    "loading_rate": "0.005",
    "bank_transaction_rate":"0.0012",
    "bank_atm_rate":"0.005"
  }
}
```

| Parameter |  Type  |          Description          |
| :--------: | :----: | :------------------------------ |
|   open_card_fee_usdt   | String |           Open card fee in USDT          |
|   exchange_rate   | String |   exchange rate of  USDT to fiat currency         |
|   loading_rate   | String |           Loading rate for deposit to user        |
|   bank_transaction_rate   | String |          Bank transaction rate for consumption          |
|   bank_atm_rate   | String |          ATM withdraw rate           |

## 5. Bank account API

This API provides details such as transaction records and other account related information.

### 5.1 Querying card status

```text
url：/api/v1/bank/account-status
method：POST
```

- Request:

| Parameter |  Type  | Whether Required | Description |
| :-------: | :----: | :--------------: | :---------- |
|  card_no  | String |     Required     | Card ID |

- Response: 

```json
{
  "code": 0,
  "msg": "string",
  "result": true
}
```


### 5.2 Check account balance

```text
url：/api/v1/bank/balance
method：POST
```

- Request:

| Parameter |  Type  | Whether Required | Description |
| :-------: | :----: | :--------------: | :---------- |
|  card_no  | String |     Required     | Card ID |

- Response:

```json
{
    "code": 0,
    "msg": "SUCCESS",
    "result": {
        "card_number": "438521******2001",
        "card_type": "EGEN BLUE",
        "current_balance": "148.05",
        "available_balance": "138.05"
    }
}
```

|       Parameter       |  Type  | Description     |
| :---------------: | :----: | :-------------- |
|      card_number      | String | Actual card number     |
|     card_type     | String | Card type       |
|  current_balance  | String | Current balance（USD） |
| available_balance | String | Usable balance（USD）  |

### 5.3 Check transaction records 

```text
url：/api/v1/bank/transaction-record
method：POST
```

- Request:

|     Parameter     |  Type  | Whether Required | Description                                 |
| :---------------: | :----: | :--------------: | :------------------------------------------ |
|      card_no      | String |     Required     | Card ID                                 |
| former_month_year | String |     Required     | Period upper limit month (Format: `012020`) |
| latter_month_year | String |     Required     | Period lower limit month (Format: `012020`) |

- Response:

```json
{
    "code": 0,
    "msg": "SUCCESS",
    "result": [
      {
      	  "month_year":"022020",
          "statement_cycle_date": "28/11/2019",
          "opening_balance": "0.00",
          "closing_balance": "150.55",
          "available_balance": "N/A",
          "bank_tx_list": [
              {
                  "transaction_date": "20/11/2019",
                  "posting_date": "20/11/2019",
                  "description": "MONTHLY FEE",
                  "debit": "2.50",
                  "credit": "",
                  "type": 1
              },
              {
                  "transaction_date": "28/11/2019",
                  "posting_date": "28/11/2019",
                  "description": "MONTHLY FEE",
                  "debit": "2.50",
                  "credit": "",
                  "type": 1
              }
          ]
      }
    ]
 }   
```

|              Parameter               |  Type  | Description                                                        |
| :------------------------------: | :----: | :----------------------------------------------------------------- |
|       month_year                 | String  | query Date,format:MMyyyy                                      |
|       statement_cycle_date       | String | Date for generating statement                                      |
|         opening_balance          | String | Opening balance(USD)                                                    |
|         closing_balance          | String | Closing balance(USD)                                                    |
|        available_balance         | String | Usable balance(USD)                                                     |
|         bank_tx_list[n]          | Object | Transaction list                                                   |
| bank_tx_list[0].transaction_date | String | Transaction date                                                   |
|   bank_tx_list[0].posting_date   | String | Transaction record submission date                                 |
|   bank_tx_list[0].description    | String | Description                                                        |
|      bank_tx_list[0].debit       | String | Debit amount(USD)                                                       |
|      bank_tx_list[0].credit      | String | Credit amount(USD)                                                      |
|       bank_tx_list[0].type       |  int   | Transaction type, 1. Debit, 2. Deposit, 3. Withdrawal, 4. Transfer |

## 6. Error Codes

### 6.1. Business Logic Error Codes

| Status Code | Description                                                 |
| :---------: | ----------------------------------------------------------- |
|      0      | Succesful                                                   |
|   111001    | Request parameter error                                     |
|   111002    | KYC status abnormal                                         |
|   111003    | KYC failure                                                 |
|   111004    | Duplicate KYC                                               |
|   111005    | KYC record not found                                        |
|   111006    | Specified institution not found                             |
|   111007    | Abnormal institution status                                 |
|   111008    | Institution asset configuration not found                   |
|   111009    | Institution asset status abnormal                           |
|   111010    | Institution configuration not found                         |
|   111011    | Insufficient institution balance                            |
|   111012    | Account not found                                           |
|   111013    | Account status abnormal                                     |
|   111014    | Insufficient accounts of specified account type             |
|   111015    | Account frozen                                              |
|   111016    | ID Account type ID not found                                |
|   111017    | Account already exists                                      |
|   111018    | Internet banking not active                                 |
|   111019    | Account ID already exists                                   |
|   111020    | Only one card can be registered for one particular type     |
|   111021    | No account of the specified type exists for the institution |
|   111022    | Transaction not found                                       |
|   111023    | Unauthorized to query this transaction                      |
|   111024    | Invalid Email format                                        |
|   111025    | Invalid Email verification code                             |
|   111026    | Email already exists                                        |
|   111027    | Email provider not supported                                |
|   111028    | Contact number already exists                               |
|   111029    | Invalid mobile verification code                            |
|   111030    | Unable to fetch bank details                                |
|   111031    | Specified bank not found                                    |
|   111032    | Noumena configuration not found                             |
|   111033    | Birthdate field cannot be left empty                        |
|   111034    | Picture could not be uploaded                               |
|   111035    | Max. query time limited to one month                        |
|   111036    | Invalid time format                                         |
|   111037    | Max. bank statement period limited to six months            |

### 6.2 Identity Authentication Error Codes

| Status Code | Description                 |
| :---------: | --------------------------- |
|   112001    | Request timed out           |
|   112002    | Illegal access privileges   |
|   112003    | Invalid IP address          |
|   112004    | Invalid timestamp           |
|   112005    | Verification failure        |
|   112006    | Invalid verification format |
|   112007    | Invalid signature           |
|   112008    | Specified app key not found |
|   112009    | Invalid app key secret      |
|   112010    | Invalid request header      |

### 6.3 Abnormal Status Error Codes

| Status Code | Description           |
| :---------: | --------------------- |
|   119001    | Service unusable      |
|   119002    | Communication error   |
|   119003    | Data encryption error |
|   119004    | Data decryption error |
|   119005    | Too many API requests |
|   119006    | Unauthorized API      |

