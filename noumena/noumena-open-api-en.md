# Noumena-OpenAPI Interface

- [API Specifications](#api-specifications)
- [1.Customers API](#1-customers)
- [2.Debit Cards API](#2-debit-cards)
- [3.Transactions API](#3-transactions)
- [4.Public API](#4-public-api)
- [5.Bank Account API](#5-bank-account-api)

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
|      country_code      | String |     Required     | International country code, for example "+86". Max. character length: 5 |
|         mobile         | String |     Required     |                           Mobile number, Max. character length: 32                            |
|          mail          | String |     Required     |                           Email address, Max. character length: 64                            |
|        address         | String |     Required     |                          Postal address, Max. character length: 256                           |
|        zipcode         | String |     Required     |                              Zip code, Max. character length: 20                              |
|      maiden_name       | String |     Required      |                         Legal maiden name, Max. character length: 255                         |
|        bank_id         | String |     Required     |                            ID of the bank where the account exists                            |
|        kyc_info        | String |     Optional     |                                     Other KYC information                                     |
| mail_verification_code | String |     Optional     |                                    Email verification code                                    |
|       mail_token       | String |     Optional     |                        Token returned upon sending verification Email                         |

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
        "bank_id": "1001",
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
|bank_id|String| bank id|
|   status    |  int    | Status code : 0 - Submitted successfully, 1 - Verification successful (Account activated), 2 - Verification failed 3 Verifying|
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
        "bank_id": "1001",
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
|bank_id|String| bank id|
|   status    |  int    | Status code : 0 - Submitted successfully, 1 - Verification successful (Account activated), 2 - Verification failed |
| reason | String |      Reason for verification failure. Blank for status other than failure       |
| create_time |  long   |                                                   Creation time                                                    |


## 2. Debit Cards

This API contains methods related to

### 2.1. Submitting user's card information

- Request:

```text
url：/api/v1/debit-cards
method：POST
```

| Parameter |  Type  | Whether Required |                            Description                            |
| :-------: | :----: | :--------------: | :---------------------------------------------------------------: |
|  acct_no  | String |     Required     | Institution account name (Unique within scope of the institution) |
|  bank_id  | String |     Required     |              ID of the bank where the account exists              |

- Response:

```json
{
  "code": 0,
  "msg": "string",
  "result": {
    "card_no": "xxxx",
    "card_number":"xxxx"
  }
}
```

|  Parameter  |  Type  |     Description     |
| :-----: | :----: | :-----------------: |
| card_no | String | To prevent real card information from being exposed, use 'card_no' parameter to query  |
| card_number | String | Actual card number |

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
|   card_no   |  int   |                             Card number                              |
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
|   card_no   |  int   |                             Card number                              |
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
| coin_type  | String |     Required     |              Only USDT supported in version v1.0.0              |
| cust_tx_id | String |     Required     |                    Institution transaction ID                     |
|  remarks   | String |     Optional     |                        Transaction remarks                        |

- Response:

```json
{
  "code": 0,
  "msg": "string",
  "result": {
    "tx_id": "2020011910590413101433814",
    "usd_amount": "10",
    "exchange_rate": "1",
    "fee": "1"
  }
}
```

| Parameter |  Type    | Description |
| :------------: | :----------: |:---------- |
|     tx_id      | String | Noumena transaction ID  |
|     usd_amount      | String | Noumena transaction ID  |
|     exchange_rate      | String | Exchange rate |
|     fee      | String | Deposit fee |

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
|     tx_status      | int | 0:process pending，1: deposit successful，5：cancelled  |

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
        "usd_amount": "1",
        "exchange_rate": "1",
        "bank_tx_id": "",
        "fee": "1",
        "cust_tx_time": null
      }
    ]
  }
}
```

|     Parameter     |  Type  |                            Description                            |
| :-----------: | :----: | :---------------------------------------------------------------: |
|    acct_no    | String | Institution account name (Unique within scope of the institution) |
|    card_no    |  int   |                            Card number                            |
|   tx_amount   | String |                        Transaction amount                         |
|  usd_amount   | String |                     Transaction amount in USD                     |
| exchange_rate | String |                           Exchange rate                           |
|  bank_tx_id   | String |                        Bank transaction ID                        |
|      fee      | String |                          Transaction fee                          |
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
        "usd_amount": "1",
        "exchange_rate": "1",
        "cust_tx_id": "1",
        "bank_tx_id": "",
        "fee": "1",
        "cust_tx_time": null
      }
    ]
  }
}
```

|     Parameter     |  Type  |                            Description                            |
| :-----------: | :----: | :---------------------------------------------------------------: |
|    acct_no    | String | Institution account name (Unique within scope of the institution) |
|    card_no    |  int   |                            Card number                            |
|   tx_amount   | String |                        Transaction amount                         |
|  usd_amount   | String |                     Transaction amount in USD                     |
| exchange_rate | String |                           Exchange rate                           |
|  bank_tx_id   | String |                     Bank transaction records                      |
|      fee      | String |                          Transaction fee                          |
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
|  card_no  | String |     Required     | Card number |

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
|  card_no  | String |     Required     | Card number |

- Response:

```json
{
    "code": 0,
    "msg": "SUCCESS",
    "result": {
        "card_number": "4385211206642001",
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
|  current_balance  | String | Current balance |
| available_balance | String | Usable balance  |


### 5.3 Check transaction records 

```text
url：/api/v1/bank/transaction-record
method：POST
```

- Request:

|     Parameter     |  Type  | Whether Required | Description                                 |
| :---------------: | :----: | :--------------: | :------------------------------------------ |
|      card_no      | String |     Required     | Card number                                 |
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
|       month_year                 | String  | query Date,format:DDyyyy                                      |
|       statement_cycle_date       | String | Date for generating statement                                      |
|         opening_balance          | String | Opening balance                                                    |
|         closing_balance          | String | Closing balance                                                    |
|        available_balance         | String | Usable balance                                                     |
|         bank_tx_list[n]          | Object | Transaction list                                                   |
| bank_tx_list[0].transaction_date | String | Transaction date                                                   |
|   bank_tx_list[0].posting_date   | String | Transaction record submission date                                 |
|   bank_tx_list[0].description    | String | Description                                                        |
|      bank_tx_list[0].debit       | String | Debit amount                                                       |
|      bank_tx_list[0].credit      | String | Credit amount                                                      |
|       bank_tx_list[0].type       |  int   | Transaction type, 1. Debit, 2. Deposit, 3. Withdrawal, 4. Transfer |