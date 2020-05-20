# Noumena API

- [接口规范](#接口规范)
- [1.机构](#机构)
     - [1.1 支持的卡种类查询](#支持的卡种类查询)
     - [1.2 查询机构余额](#查询机构余额)
     - [1.3 机构信息查询](#机构信息查询)
     - [1.4 上传公钥](#上传公钥)
     - [1.5 费率查询](#费率查询)
     - [1.6 估算将到账的法币金额](#估算将到账的法币金额)
     - [1.7 估算需要充值多少数字货币](#估算需要充值多少数字货币)
- [2.KYC](#KYC)
     - [2.1 提交用户 KYC 数据](#提交用户-KYC-数据)
     - [2.2 查询所有用户 KYC 记录](#查询所有用户-KYC-记录)
     - [2.3 查询指定用户 KYC 记录](#查询指定用户-KYC-记录)
- [3.开卡](#开卡)
     - [3.1 提交用户开卡申请](#提交用户开卡申请)
     - [3.2 提交激活卡需要的附件](#提交激活卡需要的附件)
     - [3.3 用户激活卡片](#用户激活卡片)
     - [3.4 查询所有卡片状态](#查询所有卡片状态)
     - [3.5 查询指定用户所有卡片状态](#查询指定用户所有卡片状态)
- [4.充值](#充值)
     - [4.1 用稳定币给用户卡充值](#用稳定币给用户卡充值)
         - [4.1.2 固定到账法币金额](#用稳定币给用户卡充值-指定到账法币金额-)
     - [4.2 用非稳定币给用户卡充值](#用非稳定币给用户卡充值)
         - [4.2.2 固定到账法币金额](#用非稳定币给用户卡充值-指定到账法币金额-)
     - [4.3 查询币对价格](#查询币对价格)
     - [4.4 查询某笔卡充值交易状态](#查询某笔卡充值交易状态)
     - [4.5 查询所有卡充值记录](#查询所有卡充值记录)
     - [4.6 查询指定用户所有卡充值记录](#查询指定用户所有卡充值记录)
- [5.银行卡查询](#银行卡查询)
     - [5.1 查询卡是否激活](#查询卡是否激活)
     - [5.2 查询卡余额](#查询卡余额)
     - [5.3 查询卡账单](#查询卡账单)
     - [5.4 查询虚拟银行卡信息](#查询虚拟银行卡信息)
     - [5.5 用户请求重置密码（暂不支持）](#用户请求重置密码-暂不支持-)
- [6.验证码](#验证码)
     - [6.1 发送邮箱验证码](#发送邮箱验证码)
     - [6.2 校验邮箱验证码](#校验邮箱验证码)
     - [6.3 发送手机验证码 (暂不支持)](#发送手机验证码-暂不支持-)
     - [6.4 校验手机验证码(暂不支持)](#校验手机验证码-暂不支持-)
- [7.事件推送](#事件推送)    
     - [7.1 推送 KYC 事件](#推送-KYC-事件)
     - [7.2 推送开卡事件](#推送开卡事件)
     - [7.3 推送卡激活事件](#推送卡激活事件)
     - [7.4 推送卡充值事件](#推送卡充值事件)
     - [7.5 测试推送事件](#测试推送事件)
     - [7.5 查询推送失败的事件](#查询推送失败的事件)
     - [7.6 更新推送失败的事件](#更新推送失败的事件)
- [8.错误码](#错误码)
     - [8.1 业务逻辑错误码](#业务逻辑错误码)
     - [8.2 身份权限认证错误码](#身份权限认证错误码)
     - [8.3 异常错误码](#异常错误码)

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


## 机构


### 支持的卡种类查询

```text
url：/api/v1/institution/card/type
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
                "description": "card 1",
                "card_network": "visa",
                "virtual_card": false
            },
            {
                "card_type_id": "50000002",
                "currency_type": "USD",
                "bank_id": "5000",
                "description": "card 2",
                "card_network": "visa",
                "virtual_card": true
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
|   card_network   | String |    发卡机构：visa、master、unionpay           |
|   virtual_card   | Bool |    是否是虚拟卡           |


### 查询机构余额

- Request:

```text
url：/api/v1/institution/balance
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
            "address": [
                "0x78fdf93079152d506f42042005a0ea89db595c28",
                "0x6e033252529baab12b8074e2066c96ed35593335",
                "0xd29d36bb71e4fb2f81fec84828369a668721b2a2"
            ],
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


### 机构信息查询


- Request:

```text
url：/api/v1/institution/info
method：GET
```

- Response:

```
{
    "code": 0,
    "msg": "SUCCESS",
    "result": {
        "publickey":  ""
    }
}
```

| Parameter |  Type  |          Description          |
| :--------: | :----: | :------------------------------ |
|   publickey   | String |           机构公钥           |


### 上传公钥

- Request:

```text
url：/api/v1/institution/publickey
method：POST
```

|  Parameter  | Type  | Whether Required |                        Description                         |
| :---------: | :---: | :--------------: | :-------------------------------------------------------- |
| public_key  |  String  |    Required     | public key     |


- Response:

```
{
    "code": 0,
    "msg": "SUCCESS",
    "result": true
}
```


### 费率查询

```text
url：/api/v1/institution/rates?card_type_id={card_type_id}
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
        "exchange_rate": "1.00505392692",
        "card_application_fee": "1",
        "min_deposit":"0",
        "max_deposit":"10000",
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
        "bank_atm_rate": "0.1",
        "bank_transaction_rate": "0.2"
    }
}
```

| Parameter |  Type  |          Description          |
| :--------: | :----: | :------------------------------ |
|   card_application_fee   | String |           开卡的手续费（USDT）           |
|   min_deposit   | String |           单笔最小充值金额（法币）           |
|   max_deposit   | String |           单笔最大充值金额（法币）           |
|   exchange_rate   | String |           USDT兑换相应法币的汇率           |
|   loading_rate   | String |           给用户充值时付给 Noumena 的阶梯费率，大部分卡只有一阶费率           |
|   bank_transaction_rate   | String |          银行卡刷卡消费的手续费率           |
|   bank_atm_rate   | String |          ATM取款时的手续费率           |
|   bank_atm_fee   | String |          ATM取款时的固定手续费           |


### 估算将到账的法币金额

- Request:

```text
url：/api/v1/institution/estimation/currency
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
        "coin_type": "usdt",
        "coin_amount": "106.23",
        "currency_type": "usd",
        "currency_amount": "94.13",
        "exchange_rate": "1.00145966373",
        "fiat_exchange_rate": "1",
        "exchange_fee": "1.0623",
        "exchange_fee_rate": "0.01",
        "loading_fee": "5.3115",
        "coin_price": "1"
    }
}
```

|    Parameter    |  Type   |      Description           |
| :---------: | :----:   | :--------------------------- |
|  coin_amount  |  String      |  充值的数字货币金额     |
| coin_type  |  String      |  充值的数字货币类型     |
|  currency_amount    | String  |    充值到账的法币金额          |
|  currency_type  |  String    |  充值到账的法币类型    |
| exchange_fee    | String  |   其他币种兑换USDT的手续费，单位是 coin_type            |
| exchange_fee_rate    | String  |   其他币种兑换USDT的手续费率          |
|  loading_fee    | String  |       充值手续费，单位是 coin_type        |
| exchange_rate    | String  | USDT/USD 汇率              |
| fiat_exchange_rate    | String  | 卡支持的法币/USD 汇率              |
| coin_price    | String  | coin_type/USDT 价格              |

> 注： exchange_rate = (currency_amount * fiat_exchange_rate) / ((coin_amount - exchange_fee - loading_fee) * coin_price)

### 估算需要充值多少数字货币

- Request:

```text
url：/api/v1/institution/estimation/crypto
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
        "currency_type": "usd",
        "currency_amount": "94.13",
        "exchange_rate": "1.00145966373",
        "fiat_exchange_rate": "1",
        "exchange_fee": "1.0623",
        "exchange_fee_rate": "0.01",
        "loading_fee": "5.3115",
        "coin_price": "1"
    }
}
```

|    Parameter    |  Type   |      Description          |
| :---------: | :----:   | :--------------------------- |
|  coin_amount    | String  |    需要的数字货币金额         |
|  coin_type  |  String    |  需要的数字货币类型    |
|  currency_amount    | String  |    充值到账的法币金额          |
|  currency_type  |  String    |  充值到账的法币类型    |
| exchange_fee    | String  |   其他币种兑换USDT的手续费，单位是 coin_type            |
| exchange_fee_rate    | String  |   其他币种兑换USDT的手续费率           |
|  loading_fee    | String  |       充值手续费，单位是 coin_type       |
| exchange_rate    | String  | USDT/USD 汇率             |
| fiat_exchange_rate    | String  | 卡支持的法币/USD 汇率              |
| coin_price    | String  | coin_type/USDT 价格              |

> 注： exchange_rate = (currency_amount * fiat_exchange_rate) / ((coin_amount - exchange_fee - loading_fee) * coin_price)

## KYC

提供给机构使用的接口，如机构为用户创建KYC、查询KYC记录等。

### 提交用户 KYC 数据

可选邮件和短信验证码校验，校验通过更新验证码状态为已使用。

```text
url：/api/v1/customers/accounts
method：POST
```


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
|   country | String |必填 |用户所在国家，字符长度最大50位|
|   nationality | String |必填 |国籍，字符长度最大255位|
| doc_no | String |必填 |证件号码，字符长度最大128位|
| doc_type | String |必填 |证件类型(目前只支持passport): passport: 护照，idcard：身份证，字符长度最大8位|
| front_doc | String |必填 |正面照。base64编码, 照片文件大小应小于2M|
| back_doc | String |选填 |反面照，doc_type是idcard时必须填写。base64编码，照片文件大小应小于2M|
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
| poa_doc | String[] |选填 |地址证明照片。base64编码，照片或PDF文件每个文件大小应小于2M|

- 响应：

```json
{
  "code": 0,
  "msg": "string",
  "result": true
}
```

> 如何获取 mail_token and mail_verification_code? 请看 "6.1.发送邮箱验证码". mail_verification_code 由用户自己填写.

### 查询所有用户 KYC 记录

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


### 查询指定用户 KYC 记录

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





## 开卡

提供机构给用户开卡，记录查询等接口。

### 提交用户开卡申请

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
|   status   | int |   状态：2. 开卡申请成功(未激活)， 5. 申请失败(卡片正在制作中，请过会再申请)           |


### 提交激活卡需要的附件

只有激活时需要提交附件的卡类型，才需要调用这个接口。

- Request:

```text
url：/api/v1/debit-cards/attachment
method：POST
```

|  Parameter  | Type  | Whether Required |                        Description                         |
| :---------: | :---: | :--------------: | :-------------------------------------------------------- |
|  card_no  |  String  |    必填     | 卡号    |
|  poa_doc  |  String[]  |    选填     |   地址证明照。base64编码，照片或PDF文件每个文件大小应小于2M。如果已在KYC时提交，无需再提交。  |
|  active_doc  |  String  |    选填     | 手持护照和银行卡照。base64编码，照片文件大小应小于2M  |

- Response:

```
{
    "code": 0,
    "msg": "SUCCESS",
    "result": true
}
```

### 用户激活卡片

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




### 查询所有卡片状态

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
|   status    |  int   | 状态码: 0 冻结， 1 激活成功， 2未激活， 3. 激活待审核， 4. 激活审核失败,  5. 申请失败(卡片正在制作中，请过会再申请) |
| create_time |  long  |              创建时间               |



### 查询指定用户所有卡片状态

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
|   status    |  int   | 状态码: 0 冻结， 1 激活成功， 2未激活， 3. 激活待审核， 4. 激活审核失败, 5. 申请失败(卡片正在制作中，请过会再申请)|
| create_time |  long  |              创建时间               |


## 充值



### 用稳定币给用户卡充值

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
|    coin_type    | String | 必填|币种。只支持USDT       |
|   cust_tx_id    | String | 必填|机构的交易流水号           |
|     remark     | String | 选填|交易备注                   |

- 响应：

```json
{
    "code": 0,
    "msg": "SUCCESS",
    "result": {
        "tx_id": "2020022511324811001637548",
        "coin_type": "USDT",
        "tx_amount": "0.92",         
        "exchange_fee_rate": "0",
        "exchange_fee": "0",
        "loading_fee": "0.0812",        
        "deposit_usdt": "0.9188",
        "currency_type": "USD",
        "currency_amount": "0.92",
        "exchange_rate": "1.00239251357",
        "fiat_exchange_rate": "1"
    }
}
```

| Parameter |  Type    | Description |
| :------------: | :----------: |:---------- |
|     tx_id      | String | Noumena 交易流水id  |
|    coin_type    | String | 充值币种       |
|    tx_amount      | String | 充值币种对应的金额         |
|     deposit_usdt      | String | 扣除手续费后,为用户充值的USDT数量，单位是USDT   |
|     exchange_fee_rate      | String | 充值币种兑换成USDT的费率   |
|     exchange_fee      | String | 充值币种兑换成USDT的费用，单位是USDT  |
|     loading_fee      | String | 充值手续费，单位是USDT   |
|     currency_amount      | String | 到账法币数量  |
|     currency_type      | String | 到账法币类型  |
|     exchange_rate      | String | USDT/USD汇率  |
|     fiat_exchange_rate      | String | 卡支持的法币/USD汇率  |

> 如果coin_type是USDT，从机构扣的USDT费用 = exchange_fee + loading_fee + deposit_usdt。


### 用稳定币给用户卡充值(固定到账法币金额)

```text
url：/api/v1/deposit-transactions/fiat-amount
method：POST
```

- 请求：

| Parameter |  Type  | Requirement  |Description                |
| :------------: | :----: | :----------: |:---------- |
|     card_no     | String | 必填|银行卡ID                   |
|     acct_no     | String | 必填|机构端用户编号(机构端唯一) |
|     credited_amount      | String | 必填|卡将到账的法币金额         |
|    coin_type    | String | 必填|充值使用的币种。只支持USDT      |
|   cust_tx_id    | String | 必填|机构的交易流水号           |
|     remark     | String | 选填|交易备注                   |

- 响应：

```json
{
    "code": 0,
    "msg": "SUCCESS",
    "result": {
        "tx_id": "2020022511324811001637548",
        "coin_type": "USDT",
        "tx_amount": "0.92",          
        "exchange_fee_rate": "0",
        "exchange_fee": "0",
        "loading_fee": "0.0812",        
        "deposit_usdt": "0.9188",
        "currency_type": "USD",
        "currency_amount": "0.92",
        "exchange_rate": "1.00239251357",
        "fiat_exchange_rate": "1"
    }
}
```

| Parameter |  Type    | Description |
| :------------: | :----------: |:---------- |
|     tx_id      | String | Noumena 交易流水id  |
|    coin_type    | String | 充值币种       |
|    tx_amount      | String | 充值币种对应的金额         |
|     deposit_usdt      | String | 扣除手续费后,为用户充值的USDT数量，单位是USDT   |
|     exchange_fee_rate      | String | 充值币种兑换成USDT的费率   |
|     exchange_fee      | String | 充值币种兑换成USDT的费用，单位是USDT  |
|     loading_fee      | String | 充值手续费，单位是USDT   |
|     currency_amount      | String | 到账法币数量  |
|     currency_type      | String | 到账法币类型  |
|     exchange_rate      | String | USDT/USD汇率  |
|     fiat_exchange_rate      | String | 卡支持的法币/USD汇率  |


### 用非稳定币给用户卡充值

 ETH 充值金额请大于或等于0.01，BTC 充值金额请大于或等于0.005。用非稳定币给用户卡充值，Noumena需要先去交易所按市场价兑换，```loading_fee``` 和 ```currency_amount``` 兑换后才能确定。

```text
url：/api/v1/deposit-transactions/crypto
method：POST
```

- 请求：

| Parameter |  Type  | Requirement  |Description                |
| :------------: | :----: | :----------: |:---------- |
|     card_no     | String | 必填|银行卡ID                   |
|     acct_no     | String | 必填|机构端用户编号(机构端唯一) |
|     amount      | String | 必填|充值对应币种的金额         |
|    coin_type    | String | 必填|币种。只支持BTC、ETH      |
|   cust_tx_id    | String | 必填|机构的交易流水号           |
|     remark     | String | 选填|交易备注                   |

- 响应：

```json
{
    "code": 0,
    "msg": "SUCCESS",
    "result": {
        "tx_id": "2020022511324811001637548",
        "coin_type": "BTC",
        "tx_amount": "0.01",        
        "exchange_fee_rate": "0",
        "exchange_fee": "0",
        "currency_type": "USD",
        "fiat_exchange_rate": "1",
        "exchange_rate": "1.00221569722"
    }
}
```

| Parameter |  Type    | Description |
| :------------: | :----------: |:---------- |
|     tx_id      | String | Noumena 交易流水id  |
|    coin_type    | String | 充值币种       |
|    tx_amount      | String | 充值币种对应的金额         |
|     exchange_fee_rate      | String | 充值币种兑换成USDT的费率   |
|     exchange_fee      | String | 充值币种兑换成USDT的费用，单位是 ```coin_type```  |
|     currency_type      | String | 到账法币类型  |
|     exchange_rate      | String | USDT/USD汇率  |
|     fiat_exchange_rate      | String | 卡支持的法币/USD汇率  |

### 用非稳定币给用户卡充值(固定到账法币金额)

```text
url：/api/v1/deposit-transactions/crypto/fiat-amount
method：POST
```

- 请求：

| Parameter |  Type  | Requirement  |Description                |
| :------------: | :----: | :----------: |:---------- |
|     card_no     | String | 必填|银行卡ID                   |
|     acct_no     | String | 必填|机构端用户编号(机构端唯一) |
|     credited_amount      | String | 必填|卡将到账的法币金额         |
|    coin_type    | String | 必填|充值使用的币种。只支持BTC、ETH        |
|   cust_tx_id    | String | 必填|机构的交易流水号           |
|     remark     | String | 选填|交易备注                   |

- 响应：

```json
{
    "code": 0,
    "msg": "SUCCESS",
    "result": {
        "tx_id": "2020022511324811001637548",
        "coin_type": "BTC",
        "tx_amount": "0.01",        
        "exchange_fee_rate": "0",
        "exchange_fee": "0",
        "currency_type": "USD",
        "fiat_exchange_rate": "1",
        "exchange_rate": "1.00221569722"
    }
}
```

| Parameter |  Type    | Description |
| :------------: | :----------: |:---------- |
|     tx_id      | String | Noumena 交易流水id  |
|    coin_type    | String | 充值币种       |
|    tx_amount      | String | 充值币种对应的金额         |
|     exchange_fee_rate      | String | 充值币种兑换成USDT的费率   |
|     exchange_fee      | String | 充值币种兑换成USDT的费用，单位是 ```coin_type```  |
|     currency_type      | String | 到账法币类型  |
|     exchange_rate      | String | USDT/USD汇率  |
|     fiat_exchange_rate      | String | 卡支持的法币/USD汇率  |

### 查询币对价格

```text
url：/api/v1/deposit-transactions/price
method：GET
```

- 请求：


- 响应：

```json
{
    "code": 0,
    "msg": "SUCCESS",
    "result": {
        "total": 2,
        "records": [
            {
                "symbol": "BTC/USDT",
                "price": "7553.5492732425",
                "update_time": "2020-04-26 07:19:30"
            },
            {
                "symbol": "ETH/USDT",
                "price": "193.9202409808",
                "update_time": "2020-04-26 07:19:30"
            }
        ]
    }
}
```

|  Parameter   |  Type  |        Description         |
| :--------: | :----: | :------------------------------ |
|  symbol   | String |         	币对名称        |
|    price    | String | 当前价格 |
|    update_time    | String | 价格更新时间 |

### 查询某笔卡充值交易状态

```text
url：/api/v1/deposit-transactions/{tx_id}/status
method：GET
```

- 请求：

| Parameter |  Type  |Requirement  | Description |
| :------------: | :----: | :----------: |:---------- |
|     tx_id      | String | 必填|Noumena 交易流水 tx_id 或 cust_tx_id  |

- 响应：

```json
{
  "code": 0,
  "msg": "string",
  "result": {
                "cust_tx_id": "1223",
                "acct_no": "03030062",
                "card_no": "8993152800000013334",
                "cust_tx_time": 1584350913000,
                "tx_id": "2020031609283339501898843",
                "coin_type": "USDT",
                "tx_amount": "61.86",     
                "exchange_fee": "0",
                "loading_fee": "1.8558",
                "currency_type": "USD",                
                "currency_amount": "60",
                "exchange_rate": "1.00239251357",
                "fiat_exchange_rate": "1",
                "tx_status": 3,
                "coin_price": "1"
  }
}
```

|  Parameter   |  Type  |        Description         |
| :--------: | :----: | :------------------------------ |
|  cust_tx_id   | String |         机构流水号         |
|    acct_no    | String | 机构端用户编号(机构端唯一) |
|    card_no    |  int   |          银行卡ID         |
|  cust_tx_time  |  long  |          创建时间          |
|  tx_id   | String |        交易id         |
|    coin_type    |  int   |          充值币种          |
|   tx_amount   | String |          充值金额          |
|  exchange_fee   | String |     充值币种兑换成USDT的费用，单位是 ```coin_type```   |
|      loading_fee      | String |  充值手续费，单位是 ```coin_type```           |
|     currency_type      | String | 到账法币类型  |
|     currency_amount      | String | 到账法币数量  |
| exchange_rate | String |            USDT/USD汇率            |
|   fiat_exchange_rate      | String | 卡支持的法币/USD汇率  |
|  tx_status   | int |   交易状态。0、3、4:待处理中，1:充值成功，2充值失败，5:充值失败        |
|   coin_price      | String | coin_type/USD汇率  |

### 查询所有卡充值记录

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
                "cust_tx_id": "1223",
                "acct_no": "03030062",
                "card_no": "8993152800000013334",
                "cust_tx_time": 1584350913000,
                "tx_id": "2020031609283339501898843",
                "coin_type": "USDT",
                "tx_amount": "61.86",     
                "exchange_fee": "0",
                "loading_fee": "1.8558",
                "currency_type": "USD",                
                "currency_amount": "60",
                "exchange_rate": "1.00239251357",
                "fiat_exchange_rate": "1",
                "tx_status": 3,
                "coin_price": "1"
            }
        ]
    }
}
```

|  Parameter   |  Type  |        Description         |
| :--------: | :----: | :------------------------------ |
|  cust_tx_id   | String |         机构流水号         |
|    acct_no    | String | 机构端用户编号(机构端唯一) |
|    card_no    |  int   |          银行卡ID         |
|  cust_tx_time  |  long  |          创建时间          |
|  tx_id   | String |        交易id         |
|    coin_type    |  int   |          充值币种          |
|   tx_amount   | String |          充值金额          |
|  exchange_fee   | String |     充值币种兑换成USDT的费用，单位是 ```coin_type```    |
|      loading_fee      | String |  充值手续费，单位是 ```coin_type```           |
|     currency_type      | String | 到账法币类型  |
|     currency_amount      | String | 到账法币数量  |
| exchange_rate | String |            USDT/法币汇率            |
|   fiat_exchange_rate      | String | 卡支持的法币/USD汇率  |
|  tx_status   | int |   交易状态。0、3、4:待处理中，1:充值成功，2、5:充值失败         |
|   coin_price      | String | coin_type/USD汇率  |



### 查询指定用户所有卡充值记录

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
                "cust_tx_id": "1223",
                "acct_no": "03030062",
                "card_no": "8993152800000013334",
                "cust_tx_time": 1584350913000,
                "tx_id": "2020031609283339501898843",
                "coin_type": "USDT",
                "tx_amount": "61.86",     
                "exchange_fee": "0",
                "loading_fee": "1.8558",
                "currency_type": "USD",                
                "currency_amount": "60",
                "exchange_rate": "1.00239251357",
                "fiat_exchange_rate": "1",
                "tx_status": 3,
                "coin_price": "1"
            }
        ]
    }
}
```

|  Parameter   |  Type  |        Description         |
| :--------: | :----: | :------------------------------ |
|  cust_tx_id   | String |         机构流水号         |
|    acct_no    | String | 机构端用户编号(机构端唯一) |
|    card_no    |  int   |          银行卡ID         |
|  cust_tx_time  |  long  |          创建时间          |
|  tx_id   | String |        交易id         |
|    coin_type    |  int   |          充值币种          |
|   tx_amount   | String |          充值金额          |
|  exchange_fee   | String |     充值币种兑换成USDT的费用，单位是 ```coin_type```    |
|      loading_fee      | String |  充值手续费，单位是 ```coin_type```           |
|     currency_type      | String | 到账法币类型  |
|     currency_amount      | String | 到账法币数量  |
| exchange_rate | String |            USDT/USD汇率            |
|   fiat_exchange_rate      | String | 卡支持的法币/USD汇率  |
|  tx_status   | int |   交易状态。0、3、4:待处理中，1:充值成功，2充值失败，5:充值失败        |
|   coin_price      | String | coin_type/USD汇率  |



## 银行卡查询

提供银行卡消费记录等接口

### 查询卡是否激活

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


### 查询卡余额

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
        "current_balance_usd": "121.12454",
        "available_balance": "10.23",
        "available_balance_usd": "10.23"
    }
}
```

| Parameter |  Type  |          Description          |
| :--------: | :----: | :------------------------------ |
|   card_number   | String |         真实银行卡号           |
|   card_type   | String |         银行卡类型           |
|   current_balance   | String |      当前余额（卡支持的货币）           |
|   current_balance_usd   | String |      当前余额（USD）           |
|   available_balance   | String |   可用余额（卡支持的货币）           |
|   available_balance_usd   | String |   可用余额（USD）           |


### 查询卡账单 

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
          "opening_balance_usd": "0.00",
          "closing_balance": "150.55",
          "closing_balance_usd": "150.55",
          "available_balance": "N/A",
          "available_balance_usd": "N/A",
          "bank_tx_list": [
              {
                  "transaction_date": "20/11/2019",
                  "posting_date": "20/11/2019",
                  "tx_id": "54675678678",                  
                  "description": "MONTHLY FEE",
                  "debit": "2.50",
                  "debit_usd": "2.50",
                  "credit": "",
                  "credit_usd": "",
                  "type": 1
              },
              {
                  "transaction_date": "28/11/2019",
                  "posting_date": "28/11/2019",
                  "tx_id": "54675678677",
                  "description": "MONTHLY FEE",
                  "debit": "2.50",
                  "debit_usd": "2.50",
                  "credit": "",
                  "credit_usd": "",
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
|   opening_balance   | String | 起始余额(卡支持的货币)  |
|   opening_balance_usd   | String | 起始余额(USD)  |
|   closing_balance   | String | 截止余额(卡支持的货币)  |
|   closing_balance_usd    | String | 截止余额(USD)  |
|   available_balance   | String | 可用余额(卡支持的货币)  |
|   available_balance_usd   | String | 可用余额(USD)  |
|   bank_tx_list[n]   | Object | 交易列表  |
|   bank_tx_list[0].transaction_date   | String | 交易日期  |
|   bank_tx_list[0].posting_date   | String | 交易提交日期  |
|   bank_tx_list[0].tx_id   | String | 交易ID |
|   bank_tx_list[0].description   | String | 描述  |
|   bank_tx_list[0].debit   | String | 消费金额(卡支持的货币)  |
|   bank_tx_list[0].debit_usd   | String | 消费金额(USD)  |
|   bank_tx_list[0].credit   | String | 存入金额(卡支持的货币)   |
|   bank_tx_list[0].credit_usd   | String | 存入金额(USD)  |
|   bank_tx_list[0].type   | int | 交易类型，1.消费、2.充值、3.取款、4.转账  |


### 查询虚拟银行卡信息

- Request:

```text
url：/api/v1/bank/virtualcard
method：GET 或 POST
```

|  Parameter  | Type  | Whether Required |                        Description                         |
| :---------: | :---: | :--------------: | :-------------------------------------------------------- |
| card_no  |  String  |    Required     | card no     |
| publickey  |  String  |  Optional     | 我们会使用这个公钥替代默认的公钥加密数据   |

- Response:

```
{
    "code": 0,
    "msg": "SUCCESS",
    "result": {
        "encrypt_data": [
            "3157a4325f87a1febced1871c3f7052b",
            "048eeb04e3b652aa39294379fb5a90b68db0285d52ef5d988ce0ec382f88091c96c96fc3c446021f55cfabbe3c4bf10f697cb957ce2f98755fd42da8d61dd3b31a",
            "6c92b79575b8c0318cafd0b9a89a32d93d5e53a2a881fa54296bda01a287e4185f89e9c6b2a08d8c6fbcfebc6e8ddefc15ab9899831f59b946557458654933cf"
        ],
        "public_key": "0232dc29734993897a418dd693f089edc425dde6c2d12f29eb907cd548d68e275a"
    }
}
```
|    Parameter    |  Type   |      Description                                                     |
| :---------: | :----:   | :--------------------------- |
|  encrypt_data    | String[]  |    加密后的数据          |
|  public_key  |  String    |  公钥    |


### 用户请求重置密码（暂不支持）

机构调 Noumena 接口，Noumena 通知银行发送设置取款密码链接的邮件给用户。

```text
url：/api/v1/bank/reset-pwd
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


## 验证码

邮箱和手机验证码等接口。

### 发送邮箱验证码

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



### 校验邮箱验证码

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

### 发送手机验证码 (暂不支持)

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




### 校验手机验证码(暂不支持)

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



## 事件推送

请先在机构端后台配置 Webhook 回调地址, 收到推送后验签。推送事件的数据结构为：

| 名称| 类型|描述 |
| --- | --- |--- |
| action |String | 推送类型  |
| events | String[] | 推送的消息数组 |
| events[n].params | Object | 本次推送的消息内容，JSON 对象 |
| events[n].id | String | 推送的唯一标识符 |
| events[n].create_time|long |  事件发生的时间，默认为UTC时间 |



推送的请求头数据结构为：

| 名称| 类型|描述 |
| --- | --- |--- |
| Signature |String | 签名  |
| Timestamp | String | 时间戳 |

```
--header Timestamp：1585310160226
--header Signature：UqAwtsx9HF3s5yJh/c8luvUITZNXE/f3aujwndnXLBU=

```

如何验签请见：[noumena推送验签流程](https://github.com/noumenapay/noumena-sdk-java/blob/master/src/test/java/com/noumena/open/api/test/NotificationTest.java)



机构接受推送后，响应的数据结构如下，**如果机构返回正确数据结构，我们将不再推送该事件**：

| 名称| 类型|描述 |
| --- | --- |--- |
| code | int   |  0: success, other: failure |
|msg  |String  | 错误码描述 |


响应例子：

```

{
   "code": 0,
   "msg": "SUCCESS"
}
```



### 推送 KYC 事件

| 名称| 类型|描述 |
| --- | --- |--- |
| action  |String| kyc-status |
| events[n].params.acct_no |String | 机构下用户唯一ID |
| events[n].params.card_type_id |String | 卡类型 |
| events[n].params.status  |int| KYC状态, 1. 成功, 2. 失败 |

示例：
```
{
    "action": "kyc-status",
    "events": [
        "{\"id\":\"bc76488ddda4\",\"create_time\":1585293811000,\"params\":{\"card_type_id\":\"50010003\",\"acct_no\":\"032500004\",\"status\":1}}"
    ]
}


events 数组元素从 string 转成 json:
{
       "id": "bc76488ddda4",
       "create_time": 1585293811000,
       "params":{
           "card_type_id": "50010003",
           "acct_no": "032500004",
           "status": 1
       }
}
```


### 推送开卡事件

银行卡已制作完成，请申请银行卡。

| 名称| 类型|描述 |
| --- | --- |--- |
| action |String  |  card-application-ready|
| events[n].params.acct_no |String | 机构下用户唯一ID |
| events[n].params.card_type_id |String | 卡类型 |

示例：
```
{
    "action": "card-application-ready",
    "events": [
        "{\"id\":\"bc76488ddda4\",\"create_time\":1585293811000,\"params\":{\"card_type_id\":\"50010003\",\"acct_no\":\"032500004\"}}"
    ]
}

events 数组元素从 string 转成 json:
{
       "id": "bc76488ddda4",
       "create_time": 1585293811000,
       "params":{
           "card_type_id": "50010003",
           "acct_no": "032500004"
       }
}
```

### 推送卡激活事件

| 名称| 类型|描述 |
| --- | --- |--- |
| action|String  | card-status |
| events[n].params.card_no |String | 卡ID |
| events[n].params.status |int | 卡激活状态, 0.冻结, 1.卡激活成功, 4.卡激活审核失败 |

示例：
```
{
    "action": "card-status",
    "events": [
        "{\"id\":\"bc76488ddda4\",\"create_time\":1585293811000,\"params\":{\"card_no\":\"123434234343\",\"status\":1}}"
    ]
}


events 数组元素从 string 转成 json:
{
       "id": "bc76488ddda4",
       "create_time": 1585293811000,
       "params":{
           "card_no": "123434234343",
           "status": 1
       }
}
```

### 推送卡充值事件

| 名称| 类型|描述 |
| --- | --- |--- |
| action |String |  deposit-status|
| events[n].params.tx_id |String | 交易ID |
| events[n].params.status  |int| 卡充值状态, 1.成功, 2.失败, 5.取消 |

示例：
```
{
    "action": "deposit-status",
    "events": [
        "{\"id\":\"bc76488ddda4\",\"create_time\":1585293811000,\"params\":{\"tx_id\":\"2020031609283339501898843\",\"status\":1}}"
    ]
}

events element convert string to json:
{
       "id": "bc76488ddda4",
       "create_time": 1585293811000,
       "params":{
           "tx_id": "2020031609283339501898843",
           "status": 1
       }
}
```

### 测试推送事件

```text
url：/api/v1/events/test
method：POST
```

- 请求：

| Parameter |  Type  | Requirement  |Description |
| :------------: | :----: | :----------: |:---------- |


- 响应：


| 名称| 类型|描述 |
| --- | --- |--- |
| action |String |  deposit-status|
| events[n].params |Object | 事件参数 |

示例：
```
{
    "action": "test",
    "events": [
        "{\"id\":\"bc76488ddda4\",\"create_time\":1585293811000,\"params\":{}}"
    ]
}

events element convert string to json:
{
       "id": "bc76488ddda4",
       "create_time": 1585293811000,
       "params":{
       }
}
```

### 查询推送失败的事件

我们每隔一分钟推送一次事件，每个事件我们最多推送5次，推送5次都失败表示该事件推送失败，将不会再推送。查询推送失败的事件请用此接口：

```text
url：/api/v1/events
method：GET
```

- 请求：

| Parameter |  Type  | Requirement  |Description |
| :------------: | :----: | :----------: |:---------- |
|    action     | String | 选填| 事件名称，默认值是所有类型action |
|  page_num   | int  |    选填|页数     |
|  page_size  | int  |  选填|页的大小   |


- 响应：

```
{
  "code": 0,
  "msg": "string",
  "result": {
        "total": 2,
        "records": [{
           "id": "bc7648da4f9c466aa8bad56c3c8ddda4",
           "action": "kyc-status",
           "create_time": 1585293811000,
           "params":{
              "card_type_id": "50010003",
              "acct_no": "032500004",
              "status":1
           }
        }]
  }     
}

```


### 更新推送失败的事件

更新成功后，标记为推送成功，无法再通过```查询推送失败的事件```接口查到。

```text
url：/api/v1/events
method：PUT
```

- 请求：

| Parameter |  Type  | Requirement  |Description |
| :------------: | :----: | :----------: |:---------- |
|    event_id     | String[] | 必填| 事件id |


```json
{
  "event_id": [
      "bc7648da4f9c466aa8bad56c3c8ddda4",
      "cc7648da4f9c466aa8bad56c3c8ddda3"
  ]
}
```



- 响应：

```json
{
  "code": 0,
  "msg": "string",
  "result": true
}
```


## 错误码

### 业务逻辑错误码

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

### 身份权限认证错误码

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

### 异常错误码

| 状态值 | 描述|
| :--------------: | --------| 
|   119001 |  服务不可用  |
|   119002 |   通信错误  |
|   119003 |   数据加密错误  |
|   119004 |   数据解密错误  |
|   119005 |   api调用过于频繁  |
|   119006 |   该api没有被授权  |



