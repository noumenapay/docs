# noumena-OpenAPI 接口

* [接口规范](#接口规范)
* [1.Customers接口](#1-Customers)
* [2.Debit Cards接口](#2-DebitCards)
* [3.Transactions接口](#3-Transactions)
* [4.公共接口](#4-公共接口)
* [5.银行卡接口](#5-银行卡接口)
* [6.错误码](#6-错误码)

## 接口规范

- Open API 请求都使用 `HMAC` 认证，以保证请求的完整性，真实性，同时做身份认证和鉴权。

- **分页**。查询记录列表都有分页，分页参数：`page_num` 表示页数，`page_size` 表示每页大小。接口 `DTO` 统一返回 `total`，`records`。

- **国家**。两位国家代码，参照 `ISO 3166-1 alpha-2` 编码标准

- 时间处理。API 请求和返回时间都是 `UNIX` 时间戳，**毫秒为单位**，避免因为时区导致误差

- 金额处理。API 请求和返回都是 `String` 字符串类型，避免精度丢失

- 所有带 `body` 的请求没有特殊说明body都是 `JSON`格式，`Content-Type：application/json`

- 所有查询接口查询时间间隔必须**小于一个月**

- 接口返回格式统一：

  | Parameter |  Type  |           Description            |
  | :--------: | :----: | :------------------------------ |
  |    code    |  int   |  错误码。`0`：正常，非`0`：异常  |
  |    msg     | String | 成功为 `SUCCESS`，失败为错误描述 |
  |   result   | Object |             返回信息             |

### HMAC认证

首先机构需要申请 API `Key` 和 API `Secret`，访问 API 时会用到。

| 名词 | 解释 |
| --- | --- |
| User ID | User ID 是用来标记你的开发者账号的， 是你的用户 ID|
| API Key & API Secret| User ID 下面管理了多个 API Key + API Secret， API Key 对应你的应用，你可以有多个应用，每个应用可以申请不同的  API 权限|

#### 客户端实现流程：

1. 构造需要签名的 data ，包括
   - UNIX 时间戳，`毫秒为单位`：`request` time stamp
   - 请求方法：`HTTP` method
   - 请求 API Key： Api Key
   - 完整的请求路径，包括 `URL` 问号后的参数：request URI
   - 如果有请求 `body`，再加上请求 `body` 转换后的`字符串`：string representation of the request payload
2. 客户端根据签名 data 和 API Secret 使用 `HMAC_SHA256` 算法生成签名 signature。
3. 按照指定顺序设置 Authorization header，即 key 是：`Authorization`， value 是：Noumena:ApiKey:request time stamp:signature（以冒号拼接）。
4. 如果在服务端创建 API Key，API Secret 时使用了密码，则需要设置 Access-Passphrase header，即 `key` 是：`Access-Passphrase`，`value` 是：当时设置的密码。
5. 客户端发送数据和 Authorization header，以及 Access-Passphrase header（如果有第四步的话）到服务端。即最终发送的 http header 为：
   - Authorization:Noumena:ApiKey:request timestamp:signature
   - Access-Passphrase:Your API Secret passphrase


#### 如何构造待签名的请求body string：

请求 body 需要按照 `ASCII` 码的顺序对参数名进行排序，以  `=` 拼接 key 和 value，并以 `&` 分割多个 key-value，转换成字符串。

例如请求 `body` 为：

```javascript
{
	"ont_id":"did:ont:Ae9ujqUnAtH9yRiepRvLUE3t9R2NbCTZPG",
	"amount":190,
	"to_address":"AUol16ghiT9AtxRDtNeq3ovhWJ5iaY6iyd"
}
```

转换后为：

```text
amount=190&ont_id=did:ont:Ae9ujqUnAtH9yRiepRvLUE3t9R2NbCTZPG&to_address=AUol16ghiT9AtxRDtNeq3ovhWJ5iaY6iyd
```


#### 实现代码示例：

请参考：[https://github.com/noumenapay/noumena-sdk-java](https://github.com/noumenapay/noumena-sdk-java)

## 1.Customers

提供机构相关接口，如用户创建并KYC、查询KYC等。

### 1.1.提交用户 KYC 数据

可选邮件和短信验证码校验，校验通过更新验证码状态为已使用。

```text
url：/api/v1/customers/accounts
method：POST
```
> 目前已支持的 card_type_id （银行卡类型 ID ） 请用4.5的接口查询

- 请求：

| Parameter |  Type  |    Requirement  |Description   |
| :------------: | :----: | :----------: |:---------- |
|   acct_no | String | 必填 |机构端用户编号(机构端唯一)，字符长度最大64位|
|   acct_name | String |必填 |机构端用户名，字符长度最大64位|
|   first_name | String |必填 |真实用户名，字符长度最大50位|
|   last_name | String |必填 |真实用户姓，字符长度最大50位|
|   gender | String |必填 |male:男，female:女，unknown:其他，字符长度最大6位|
|   birthday | String |必填 |生日（生日格式为"1990-01-01"）|
|   city | String |必填 |城市，字符长度最大100位|
|   state | String |必填 |省份，字符长度最大100位|
|   country | String |必填 |国家，字符长度最大50位|
|   nationality | String |必填 |出生国，字符长度最大255位|
| doc_no | String |必填 |证件号码，字符长度最大128位|
| doc_type | String |必填 |证件类型: passport: 护照，idcard：身份证，字符长度最大8位|
| front_doc | String |必填 |正面照。base64编码, 照片文件大小应小于2M|
| back_doc | String |必填 |反面照。base64编码，照片文件大小应小于2M|
| mix_doc | String |必填 |手持证件照。base64编码，照片文件大小应小于2M|
|   country_code | String |必填 |手机国际区号，如“+86”。字符长度最大5位|
|   mobile | String |必填 |手机号，字符长度最大32位|
|  mail | String |必填 |邮箱，不支持163.com的邮箱。字符长度最大64位|
|   address | String |必填 |通讯地址，银行卡会寄到该地址。字符长度最大256位|
|   zipcode | String |必填 |邮编，字符长度最大20位|
|   maiden_name | String |必填 |妈妈的名字，字符长度最大255位|
| card_type_id |String |必填 |银行卡种类对应的id,比如 10010001|
|   kyc_info | text |选填 |KYC 其他信息|
| mail_verification_code | String |选填 |邮箱验证码|
| mail_token | String |选填|发送邮件后返回的token|
| cust_tx_id | String | 选填| KYC流水号|
| poa_doc | String |选填 |地址证明照片。base64编码，照片文件大小应小于2M|

- 响应：

```json
{
  "code": 0,
  "msg": "string",
  "result": true
}
```

> 如何获取 mail_token and mail_verification_code? 请看 "4.1.发送邮箱验证码". mail_verification_code 由用户自己填写.

### 1.2 查询所有用户kyc记录

```text
url：/api/v1/customers/accounts
method：GET
```

| Parameter  | Type |Requirement  | Description |
| :------------: | :----: | :----------: |:---------- |
|  page_num   | int  |    选填|页数     |
|  page_size  | int  |  选填|页的大小   |
| former_time | long |  选填|前置时间, UNIX 时间戳，`秒为单位`   |
| latter_time | long | 选填|后置时间, UNIX 时间戳，`秒为单位`   |
| time_sort | String | 选填|时间排序, asc为升序，desc为降序   |



- 响应：

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


| Parameter  |  Type  |                     Description                     |
| :--------: | :----: | :------------------------------ |
|   acct_no   | String |             机构端用户编号(机构端唯一)              |
| card_type_id |String |卡种对应的id|
|   status    |  int   | 状态码: 0 已提交， 1 认证通过(开卡成功)， 2 认证未通过， 3 认证中， 4 提交信息处理中 |
|   reason   | String | 认证失败原因。其他情况为空字符串 |
| create_time |  long  |                      创建时间                       |


### 1.3 查询指定用户kyc记录

```text
url：/api/v1/customers/accounts
method：GET
```

- 请求：

| Parameter  | Type |Requirement  | Description |
| :------------: | :----: | :----------: |:---------- |
|    acct_no     | String | 必填|机构用户的唯一id号 |
|  page_num   | int  |    选填|页数     |
|  page_size  | int  |  选填|页的大小   |
| former_time | long |  选填|前置时间, UNIX 时间戳，`秒为单位`   |
| latter_time | long | 选填|后置时间, UNIX 时间戳，`秒为单位`   |
| time_sort | String | 选填|时间排序, asc为升序，desc为降序   |

- 响应：

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


| Parameter  |  Type  |                     Description                     |
| :--------: | :----: | :------------------------------ |
|   acct_no   | String |             机构端用户编号(机构端唯一)              |
| card_type_id |String  |卡种对应的id|
|   status    |  int   | 状态码: 0 已提交， 1 认证通过(开卡成功)， 2 认证未通过， 3 认证中， 4 提交信息处理中  |
|   reason   | String | 认证失败原因。其他情况为空字符串 |
| create_time |  long  |                      创建时间                       |





## 2. Debit Cards

提供机构给用户开卡，记录查询等接口。

### 2.1 提交用户开卡申请

```text
url：/api/v1/debit-cards
method：POST
```

- 请求

| Parameter |  Type  |   Requirement  |Description   |
| :------------: | :----: | :----------: |:---------- |
| acct_no | String | 必填|机构端用户编号(机构端唯一)|
| card_type_id |String |必填 |卡种对应的id|


- 响应：

```json
{
  "code": 0,
  "msg": "string",
  "result": {
 	"card_no": "xxxx",
	"card_number": "430021******1144",
	"status": 2
  }
}
```

| Parameter |  Type  |          Description          |
| :--------: | :----: | :------------------------------ |
|   card_no   | String |           分配的银行卡ID，查询时用card_no，避免真是卡号信息泄露。生成规则：机构id+5位随机数 +卡种id +卡最后四位           |
|   card_number   | String |           分配的真实银行卡号, 只显示前6位和后4位           |
|   status   | int |   状态：2. 开卡申请成功， 5. 申请失败，卡片正在制作中           |


### 2.2 提交激活卡需要的附件

- Request:

```text
url：/api/v1/debit-cards/attachment
method：POST
```

|  Parameter  | Type  | Whether Required |                        Description                         |
| :---------: | :---: | :--------------: | :-------------------------------------------------------- |
|  card_no  |  String  |    必填     | 卡号    |
|  poa_doc  |  String  |    选填     |   地址证明照。base64编码，照片文件大小应小于2M  |
|  active_doc  |  String  |    选填     | 手持护照和银行卡照。base64编码，照片文件大小应小于2M  |

- Response:

```
{
    "code": 0,
    "msg": "SUCCESS",
    "result": true
}
```

### 2.3 用户激活卡片

```text
url：/api/v1/debit-cards/status
method：PUT
```

- 请求

| Parameter |  Type  |   Requirement  |     Description         |
| :------------: | :----: | :----------: |:---------- |
|    card_no     | String |      必填    |银行卡ID          |
| acct_no | String | 必填    |机构端用户编号(机构端唯一) |



- 响应：

```json
{
  "code": 0,
  "msg": "string",
  "result": true
}
```



### 2.4.用户触发设置卡片取款密码邮件（暂不支持）

B 机构调 Noumena 接口，Noumena 调 Bank，触发发送设置取款密码邮件到用户邮箱的动作

```text
url：/api/v1/debit-cards/deposit-pwd-emails
method：POST
```

- 请求

| Parameter |  Type  |   Requirement  |     Description         |
| :------------: | :----: | :----------: |:---------- |
|    card_no     | String |      必填    |银行卡号          |
| acct_no | String | 必填    |机构端用户编号(机构端唯一) |


- 响应：

```json
{
  "code": 0,
  "msg": "string",
  "result": true
}
```



### 2.5 查询所有卡片信息

```text
url：/api/v1/debit-cards
method：GET
```

| Parameter  | Type | Requirement  |Description |
| :------------: | :----: | :----------: |:---------- |
|  page_num   | int  |    选填    |页数     |
|  page_size  | int  |  选填    |页的大小   |
| former_time | long |  选填    |前置时间, UNIX 时间戳，`秒为单位`   |
| latter_time | long |  选填    |后置时间, UNIX 时间戳，`秒为单位`   |
| time_sort | String | 选填|时间排序, asc为正序，desc为反序   |

- 响应：

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


| Parameter  |  Type  |             Description             |
| :--------: | :----: | :------------------------------ |
|   acct_no   | String |     机构端用户编号(机构端唯一)      |
|   card_no   |  int   |              银行卡号               |
|   status    |  int   | 状态码: 0 冻结， 1 激活成功， 2未激活， 3. 待审核， 4. 审核失败 |
| create_time |  long  |              创建时间               |



### 2.6 查询指定用户所有卡片信息

```text
url：/api/v1/debit-cards?acct_no={acct_no}
method：GET
```

- 请求：

| Parameter  | Type | Requirement  |Description |
| :------------: | :----: | :----------: |:---------- |
|   acct_no   | String |    必填 |机构端用户编号(机构端唯一)      |
|  page_num   | int  |    选填    |页数     |
|  page_size  | int  |  选填    |页的大小   |
| former_time | long |  选填    |前置时间, UNIX 时间戳，`秒为单位`   |
| latter_time | long |  选填    |后置时间, UNIX 时间戳，`秒为单位`   |
| time_sort | String | 选填|时间排序, asc为正序，desc为反序   |


- 响应：

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

| Parameter  |  Type  |             Description             |
| :--------: | :----: | :------------------------------ |
|   acct_no   | String |     机构端用户编号(机构端唯一)      |
|   card_no   |  int   |              银行卡ID               |
|   status    |  int   | 状态码: 0 冻结， 1 激活成功， 2未激活， 3. 待审核， 4. 审核失败|
| create_time |  long  |              创建时间               |





## 3.Transactions

### 3.1.用户卡充值

```text
url：/api/v1/deposit-transactions
method：POST
```

- 请求：

| Parameter |  Type  | Requirement  |Description                |
| :------------: | :----: | :----------: |:---------- |
|     card_no     | String | 必填|银行卡ID                   |
|     acct_no     | String | 必填|机构端用户编号(机构端唯一) |
|     amount      | String | 必填|充值对应币种的金额         |
|    coin_type    | String | 必填|币种。一期只支持USDT       |
|   cust_tx_id    | String | 必填|机构的交易流水号           |
|     remark     | String | 选填|交易备注                   |

- 响应：

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
|     tx_id      | String | Noumena 交易流水id  |
|     currency_amount      | String | 到账法币数量  |
|     currency_type      | String | 到账法币类型  |
|     exchange_rate      | String | USDT/法币汇率  |
|     loading_fee      | String | 充值手续费，单位是USDT   |
|     exchange_fee      | String | 充值币种兑换成USDT的费用，单位是USDT  |
|     exchange_fee_rate      | String | 充值币种兑换成USDT的费率   |
|     deposit_usdt      | String | 扣除手续费后,为用户充值的USDT数量，单位是USDT   |

> 从机构扣的USDT费用 = exchange_fee + loading_fee + deposit_usdt。

### 3.2.查询某笔卡充值交易状态

```text
url：/api/v1/deposit-transactions/{tx_id}/status
method：GET
```

- 请求：

| Parameter |  Type  |Requirement  | Description |
| :------------: | :----: | :----------: |:---------- |
|     tx_id      | String | 必填|Noumena 交易流水id  |

- 响应：

```json
{
  "code": 0,
  "msg": "string",
  "result": {
  	"tx_status":0
  }
}
```

| Parameter |  Type  |          Description          |
| :--------: | :----: | :------------------------------ |
|   tx_status   | int |           0, 3 和 4:待处理中，1:充值成功, 5:充值失败           |


### 3.3 查询所有卡充值记录

```text
url：/api/v1/deposit-transactions
method：GET
```

| Parameter  | Type | Requirement  |Description |
| :------------: | :----: | :----------: |:---------- |
|  page_num   | int  |    选填|页数     |
|  page_size  | int  |  选填|页的大小   |
| former_time | long |  选填|前置时间, UNIX 时间戳，`秒为单位`   |
| latter_time | long |  选填|后置时间, UNIX 时间戳，`秒为单位`   |
| time_sort | String | 选填|时间排序, asc为正序，desc为反序   |

- 响应：

```json
{
    "code": 0,
    "msg": "SUCCESS",
    "result": {
        "total": 1,
        "records": [
            {
                "currency_type": "CNY",
                "cust_tx_id": "1223",
                "card_no": "8993152800000013334",
                "acct_no": "03030062",
                "cust_tx_time": 1584350913000,
                "loading_fee": "1.8558",
                "currency_amount": "60",
                "tx_amount": "61.86",
                "exchange_rate": "1",
                "tx_id": "2020031609283339501898843",
                "coin_type": "USDT",
                "tx_status": 3,
                "exchange_fee": "0"
            }
        ]
    }
}
```

|  Parameter   |  Type  |        Description         |
| :--------: | :----: | :------------------------------ |
|     currency_type      | String | 到账法币类型  |
|  cust_tx_id   | String |         机构流水号         |
|    card_no    |  int   |          银行卡ID         |
|    acct_no    | String | 机构端用户编号(机构端唯一) |
|  cust_tx_time  |  long  |          创建时间          |
|      loading_fee      | String |  充值手续费，单位是USDT          |
|     currency_amount      | String | 到账法币数量  |
|   tx_amount   | String |          充值金额          |
| exchange_rate | String |            USDT/法币汇率            |
|  tx_id   | String |        交易id         |
|    coin_type    |  int   |          充值币种          |
|  tx_status   | int |   交易状态       |
|  exchange_fee   | String |     充值币种兑换成USDT的费用，单位是USDT   |



### 3.4 查询指定用户所有卡充值记录

```text
url：/api/v1/deposit-transactions?acct_no={acct_no}
method：GET
```

- 请求：

| Parameter |  Type  | Requirement  |Description |
| :------------: | :----: | :----------: |:---------- |
|    acct_no     | String | 必填|机构用户的唯一id号 |
|  page_num   | int  |    选填|页数     |
|  page_size  | int  |  选填|页的大小   |
| former_time | long |  选填|前置时间, UNIX 时间戳，`秒为单位`   |
| latter_time | long |  选填|后置时间, UNIX 时间戳，`秒为单位`   |
| time_sort | String | 选填|时间排序, asc为正序，desc为反序   |


- 响应：

```json
{
    "code": 0,
    "msg": "SUCCESS",
    "result": {
        "total": 1,
        "records": [
            {
                "currency_type": "CNY",
                "cust_tx_id": "1223",
                "card_no": "8993152800000013334",
                "acct_no": "03030062",
                "cust_tx_time": 1584350913000,
                "loading_fee": "1.8558",
                "currency_amount": "60",
                "tx_amount": "61.86",
                "exchange_rate": "1",
                "tx_id": "2020031609283339501898843",
                "coin_type": "USDT",
                "tx_status": 3,
                "exchange_fee": "0"
            }
        ]
    }
}
```

|  Parameter   |  Type  |        Description         |
| :--------: | :----: | :------------------------------ |
|     currency_type      | String | 到账法币类型  |
|  cust_tx_id   | String |         机构流水号         |
|    card_no    |  int   |          银行卡ID         |
|    acct_no    | String | 机构端用户编号(机构端唯一) |
|  cust_tx_time  |  long  |          创建时间          |
|      loading_fee      | String |  充值手续费，单位是USDT          |
|     currency_amount      | String | 到账法币数量  |
|   tx_amount   | String |          充值金额          |
| exchange_rate | String |            USDT/法币汇率            |
|  tx_id   | String |        交易id         |
|    coin_type    |  int   |          充值币种          |
|  tx_status   | int |   交易状态       |
|  exchange_fee   | String |     充值币种兑换成USDT的费用，单位是USDT   |



## 4.公共接口

邮箱和手机验证码等接口。

### 4.1.发送邮箱验证码

```text
url：/api/v1/emails/{email}/verification-codes
method：POST
```

- 请求：

| Parameter |  Type  |   Requirement  | Description   |
| :------------: | :----: | :----------: |:---------- |
| email | String |必填|email|

- 响应：

```json
{
  "code": 0,
  "msg": "string",
  "result": {
  	"mail_token":"xxxxxx"
  }
}
```

| Parameter |  Type  |          Description          |
| :--------: | :----: | :------------------------------ |
|   mail_token   | String |           分配的验证token           |



### 4.2.校验邮箱验证码

```text
url：/api/v1/emails/{email}/verification-codes?code={code}&mail_token={mail_token}
method：PUT
```

- 请求：

| Parameter |  Type  | Requirement  |Description |
| :------------: | :----: | :----------: |:---------- |
|     email      | String | 必填|email       |
|      code      | String | 必填|code        |
|   mail_token   | String |       必填|    分配的验证token           |

- 响应：

```json
{
  "code": 0,
  "msg": "string",
  "result": true
}
```

### 4.3.发送手机验证码 (暂不支持)

```text
url：/api/v1/mobiles/{mobile}/verification-codes
method：POST
```

- 请求：

| Parameter |  Type  | Requirement  |Description |
| :------------: | :----: | :----------: |:---------- |
|     mobile     | String | 必填|区号+手机号 |

- 响应：

```json
{
  "code": 0,
  "msg": "string",
  "result": {
  	"mobile_token":"xxxxxx"
  }
}
```

| Parameter |  Type  |          Description          |
| :--------: | :----: | :------------------------------ |
|   mobile_token   | String |           分配的验证token         |

### 4.4.校验手机验证码(暂不支持)

```text
url：/api/v1/mobiles/{mobile}/verification-codes?code={code}&mobile_token={mobile_token}
method：PUT
```

- 请求：

| Parameter |  Type  | Requirement  |Description |
| :------------: | :----: | :----------: |:---------- |
|     mobile     | String | 必填|区号+手机号 |
|      code      | String | 必填|code        |
|   mobile_token   | String |       必填|    分配的验证token         |

- 响应：

```json
{
  "code": 0,
  "msg": "string",
  "result": true
}
```



### 4.5 支持的卡种类查询

```text
url：/api/v1/card/type
method：GET
```

- 响应:

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
                "bank_id": "5000",
                "description": "card 1"
            },
            {
                "card_type_id": "50000002",
                "currency_type": "USD",
                "bank_id": "5000",
                "description": "card 2"
            }
        ]
    }
}
```

| Parameter |  Type  |          Description          |
| :--------: | :----: | :------------------------------ |
| card_type_id |String |银行卡种类对应的id,比如 50000001|
| currency_type |String |卡支持的法币类型|
|   bank_id   | String |        银行ID           |
|   description   | String |    卡种描述           |


### 4.6 费率查询

```text
url：/api/v1/rates?card_type_id={card_type_id}
method：GET
```

- 请求：

| Parameter |  Type  |   Requirement  | Description   |
| :------------: | :----: | :----------: |:---------- |
| card_type_id |String |必填 |银行卡种类对应的id,比如 50000001|

- 响应：

```json
{
    "code": 0,
    "msg": "SUCCESS",
    "result": {
        "bank_atm_rate": "0.1",
        "exchange_rate": "1.00505392692",
        "open_card_fee_usdt": "1",
        "loading_rate": [
            {
                "min": "0",
                "max": "1000",
                "step_rate": "0.05"
            },
            {
                "min": "1000",
                "max": "2000",
                "step_rate": "0.03"
            }
        ],
        "bank_atm_fee": "0",
        "bank_transaction_rate": "0.2"
    }
}
```

| Parameter |  Type  |          Description          |
| :--------: | :----: | :------------------------------ |
|   open_card_fee_usdt   | String |           开卡的手续费（USDT）           |
|   exchange_rate   | String |           USDT兑换相应法币的汇率           |
|   loading_rate   | String |           给用户充值时付给 Noumena 的阶梯费率           |
|   bank_transaction_rate   | String |          银行卡刷卡消费的手续费率           |
|   bank_atm_rate   | String |          ATM取款时的手续费率           |
|   bank_atm_fee   | String |          ATM取款时的固定手续费           |


### 4.7 数字货币计算接口

- Request:

```text
url：/api/v1/calculation/crypto
method：POST
```
- Request:

|  Parameter  | Type  | Whether Required |                        Description                         |
| :---------: | :---: | :--------------: | :--------------------------------------------------------|
|  currency_amount  |  String  |    Required     |  需要充值到账的法币金额     |
|  card_type_id  |  String  |    Required     |  卡种ID    |
|  coin_type  |  String  |    Required     |  需要换算的数字货币类型    |

- Response:

```
{
    "code": 0,
    "msg": "SUCCESS",
    "result": {
        "coin_type": "usdt",
        "coin_amount": "106.23",
        "exchange_rate": "1.00145966373",
        "exchange_fee": "1.0623",
        "exchange_fee_rate": "0.01",
        "loading_fee": "5.3115",
        "loading_fee_rate": "0.05"
    }
}
```

|    Parameter    |  Type   |      Description                                                     |
| :---------: | :----:   | :--------------------------- |
|  coin_amount    | String  |    需要的数字货币金额         |
|  coin_type  |  String    |  需要的数字货币类型    |
| exchange_fee    | String  |   其他币种兑换USDT的手续费           |
| exchange_fee_rate    | String  |   其他币种兑换USDT的手续费率           |
|  loading_fee    | String  |       充值手续费       |
|  loading_fee_rate    | String  |  充值手续费率     |
| exchange_rate    | String  | USDT/USD 汇率             |



### 4.8 法币计算接口

- Request:

```text
url：/api/v1/calculation/currency
method：POST
```

|  Parameter  | Type  | Whether Required |                        Description                         |
| :---------: | :---: | :--------------: | :-------------------------------------------------------- |
|  coin_amount  |  String  |    Required     |  充值的数字货币金额     |
| coin_type  |  String  |    Required     |  充值的数字货币类型     |
|  card_type_id  |  String  |    Required     |  卡种ID    |

- Response:

```
{
    "code": 0,
    "msg": "SUCCESS",
    "result": {
        "currency_type": "usd",
        "currency_amount": "94.13",
        "exchange_rate": "1.00145966373",
        "exchange_fee": "1",
        "exchange_fee_rate": "0.01",
        "loading_fee": "5",
        "loading_fee_rate": "0.05"
    }
}
```

|    Parameter    |  Type   |      Description                                                     |
| :---------: | :----:   | :--------------------------- |
|  currency_amount    | String  |    充值到账的法币金额          |
|  currency_type  |  String    |  充值到账的法币类型    |
| exchange_fee    | String  |   其他币种兑换USDT的手续费           |
| exchange_fee_rate    | String  |   其他币种兑换USDT的手续费率          |
|  loading_fee    | String  |       充值手续费       |
|  loading_fee_rate    | String  |   充值手续费率       |
| exchange_rate    | String  | USDT/USD 汇率              |


### 4.9 查询机构余额

- Request:

```text
url：/api/v1/customers/balance
method：GET
```

- Response:

```
{
    "code": 0,
    "msg": "SUCCESS",
    "result": [
        {
            "balance": "872.5262",
            "address": "0x57700ea3429bc5b3c5c215a530d19cbc685389cd",
            "coin_type": "USDT"
        }
    ]
}
```

|    Parameter    |  Type   |      Description  |
| :---------: | :----:   | :--------------------------- |
| balance    | String  |  数字货币余额         |
| address    | String  | 数字货币地址         |
| coin_type    | String  | 数字货币余额   |


## 5.银行卡接口

提供银行卡消费记录等接口

### 5.1 查询卡是否激活

```text
url：/api/v1/bank/account-status
method：POST
```

- 请求：

| Parameter |  Type  | Requirement  |Description |
| :------------: | :----: | :----------: |:---------- |
|     card_no     | String | 必填|银行卡ID |

- 响应：

```json
{
  "code": 0,
  "msg": "string",
  "result": true
}
```


### 5.2 查询卡余额

```text
url：/api/v1/bank/balance
method：POST
```

- 请求：

| Parameter |  Type  | Requirement  |Description |
| :------------: | :----: | :----------: |:---------- |
|     card_no     | String |必填| 银行卡ID |

- 响应：

```json
{
    "code": 0,
    "msg": "SUCCESS",
    "result": {
        "card_number": "438521******2001",
        "card_type": "EGEN BLUE",
        "current_balance": "121.12454",
        "available_balance": "10.23"
    }
}
```

| Parameter |  Type  |          Description          |
| :--------: | :----: | :------------------------------ |
|   card_number   | String |         真实银行卡号           |
|   card_type   | String |         银行卡类型           |
|   current_balance   | String |      当前余额（USD）           |
|   available_balance   | String |   可用余额（USD）           |


### 5.3 查询卡账单 

```text
url：/api/v1/bank/transaction-record
method：POST
```

- 请求：

| Parameter |  Type  | Requirement  |Description |
| :------------: | :----: | :----------: |:---------- |
|     card_no     | String | 必填|银行卡ID |
|     former_month_year     | String | 必填|指定查询的月份(格式“012020”) |
|     latter_month_year     | String | 必填|指定查询的月份(格式“012020”) |

- 响应：

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

| Parameter |  Type  |          Description          |
| :--------: | :----: | :------------------------------ |
|   month_year   | String |  日期，MMyyyy |
|   statement_cycle_date   | String |  报表生成日期 |
|   opening_balance   | String | 起始余额(USD)  |
|   closing_balance   | String | 截止余额(USD)  |
|   available_balance   | String | 可用余额(USD)  |
|   bank_tx_list[n]   | Object | 交易列表  |
|   bank_tx_list[0].transaction_date   | String | 交易日期  |
|   bank_tx_list[0].posting_date   | String | 交易提交日期  |
|   bank_tx_list[0].description   | String | 描述  |
|   bank_tx_list[0].debit   | String | 消费金额(USD)  |
|   bank_tx_list[0].credit   | String | 存入金额(USD)  |
|   bank_tx_list[0].type   | int | 交易类型，1.消费、2.充值、3.取款、4.转账  |


## 6. 错误码

### 6.1 业务逻辑错误码

| 状态值 | 描述|
| :--------------: | --------| 
|   0 |   成功  |
|   111001 |  请求参数错误 |
|   111002 |  KYC 状态异常  |
|   111003 |   KYC 失败   |
|   111004 |   KYC 重复 |
|   111005 |   无法找到对应的 KYC  |
|   111006 |   无法找到对应的机构  |
|   111007 |  机构状态异常  |
|   111008 |  无法找到对应的机构资产配置  |
|   111009 |   机构资产状态异常  |
|   111010 |    无法找到对应的机构配置  |
|   111011 |   机构余额不足  |
|   111012 |   无法找到对应的卡  |
|   111013 |   卡状态异常 |
|   111014 |   该卡种的卡不足  |
|   111015 |   该银行卡已被冻结  |
|   111016 |   无法找到对应的卡种ID  |
|   111017 |   该卡已被开卡  |
|   111018 |   网银没有激活  |
|   111019 |   卡ID重复  |
|   111020 |   用户每种卡只能开一张  |
|   111021 |   该机构没有开此类卡种的权限  |
|   111022 |   无法找到对应的交易  |
|   111023 |   没有查询该交易的权限 |
|   111024 |  邮箱格式不合法  |
|   111025 |   邮箱验证码错误  |
|   111026 |   邮箱重复  |
|   111027 |   不支持该邮箱供应商  |
|   111028 |   手机号重复  |
|   111029 |   手机验证码错误  |
|   111030 |   无法获取银行数据  |
|   111031 |   无法找到对应的银行  |
|   111032 |   无法找到对应的noumena配置  |
|   111033 |   生日参数不能为空  |
|   111034 |   无法上传照片  |
|   111035 |   查询时间不允许超过一个月  |
|   111036 |   时间格式错误  |
|   111037 |   查询的银行账单时间不允许超过6个月  |

### 6.2 身份权限认证错误码

| 状态值 | 描述|
| :--------------: | --------| 
|   112001 |   请求超时  |
|   112002 |   非法权限 |
|   112003 |  非法IP地址  |
|   112004 |   非法时间戳  |
|   112005 |   验签失败 |
|   112006 |   认证格式错误  |
|   112007 |   签名错误  |
|   112008 |   没有找到对应的 app key  |
|   112009 |   无效的 app key秘钥  |
|   112010 |   请求头错误  |

### 6.3 异常错误码

| 状态值 | 描述|
| :--------------: | --------| 
|   119001 |  服务不可用  |
|   119002 |   通信错误  |
|   119003 |   数据加密错误  |
|   119004 |   数据解密错误  |
|   119005 |   api调用过于频繁  |
|   119006 |   该api没有被授权  |



