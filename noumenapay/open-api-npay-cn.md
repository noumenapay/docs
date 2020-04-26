# noumena-pay-OpenAPI 接口



## 接口规范

- Open API 请求都使用 `HMAC` 认证，以保证请求的完整性，真实性，同时做身份认证和鉴权。

- **分页**。查询记录列表都有分页，分页参数：`page_index` 表示页数，`page_offset` 表示每页大小。接口 `DTO` 统一返回 `total`，`records`。

- **国家**。两位国家代码，参照 `ISO 3166-1 alpha-2` 编码标准

- 时间处理。API 请求和返回时间都是 `UNIX` 时间戳，**秒为单位**，避免因为时区导致误差

- 金额处理。API 请求和返回都是 `String` 字符串类型，避免精度丢失

- 所有带 `body` 的请求没有特殊说明body都是 `JSON`格式，`Content-Type：application/json`

- 接口返回格式统一：

  | Field_Name |  Type  |           Description            |
  | :--------: | :----: | :------------------------------: |
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


## 1. Noumena Pay用户充值



### 1.1.获取机构资产

```text
url：/api/v1/npay/cust/balance
method：GET
```

- 请求：


- 响应：

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
|  coin_type   | String |     Coin 类型   |
|  balance   | String |      余额   |



### 1.3.获取机构交易记录

包括充值和提币。

```text
url：/api/v1/npay/cust/transaction
method：GET
```

- 请求：

|  Field_Name   |  Type  | Whether Required |       Description         |
| :-----------: | :----: | :-----------------: | :-----------: |
|  page_num   | int  |  Required |  页数     |
|  page_size  | int  |  Required | 页的大小   |
|  acct_no  | String  |  Optional | 机构用戶账号(可选)   |
|  cust_tx_id  | String  | Optional | 机构绑定公司下的交易号(可选)   |
|  tx_id  | String  | Optional | Noumena下交易号(可选)   |

- 响应：

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
|    acct_no    | String | 机构端用户编号(机构端唯一) |
|    acct_name    | String | 机构端用户名 |
|    cust_tx_id    |  String   |   绑定公司下交易号          |
|  coin_type   | String |      币种   |
|  tx_amount   | String |      金额   |
|   address | String |   tx_type是withdraw时，用户的提取coin地址 |
| bonus | String | 奖励 |
| bonus_coin_type | String | 奖励币种 |
| tx_id | String | Noumena交易ID |
|  tx_type   | String |      交易类型：deposit 或 withdraw   |
| tx_status | int | 交易状态. 0 处理中， 1 成功， 2 失败 |
|  create_time   | long |      创建日期   |
|  update_time   | long |      更新日期   |



### 1.2.给用户充值


```text
url：/api/v1/npay/cust/deposit
method：POST
```


- 请求：

| Body_Field_Name |  Type  |  Whether Required |    Description   |
|:----------:|:------:| :-----------------:| :-----------------:|
|   acct_no | String | Required |  机构端用户编号(机构端唯一)|
|   acct_name | String | Required |  机构端用户名|
|   cust_tx_id | String | Optional |  机构端交易流水号|
|   coin_type | String | Required |  充值的币种|
|   tx_amount | String | Required |  充值金额|
|   bonus_coin_type | String | Optional |  奖励币种|
|   bonus_tx_amount | String | Optional |  奖励币种金额(bonus_coin_type为空则可以为空,否则不能为空) |
|   remark | String | Optional |  备注(可以空)|



- 响应：

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
|    tx_id    | String | 主币种充值id|



### 1.3.获取用户充值记录

```text
url：/api/v1/npay/cust/deposit
method：GET
```

- 请求：

|  Field_Name   |  Type  | Whether Required |       Description         |
| :-----------: | :----: | :-----------------: | :-----------: |
|  page_num   | int  |  Required |  页数     |
|  page_size  | int  |  Required | 页的大小   |
|  acct_no  | String  |  Optional | 机构用戶账号(可选)   |
|  cust_tx_id  | String  | Optional | 机构绑定公司下的交易号(可选)   |
|  tx_id  | String  | Optional | Noumena下交易号(可选)   |

- 响应：

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
|    acct_no    | String | 机构端用户编号(机构端唯一) |
|    acct_name    | String | 机构端用户名 |
|    cust_tx_id    |  String   |   绑定公司下交易号          |
|  coin_type   | String |      币种   |
|  tx_amount   | String |      金额   |
|   address | String |   为空 |
| bonus | String | 奖励 |
| bonus_coin_type | String | 奖励币种 |
| tx_id | String | Noumena交易ID |
|  tx_type   | String |      交易类型：deposit 或 withdraw   |
| tx_status | int | 交易状态. 0 处理中， 1 成功， 2 失败 |
|  create_time   | long |      创建日期   |
|  update_time   | long |      更新日期   |


### 1.4.用户提现


```text
url：/api/v1/npay/cust/withdrawal
method：POST
```


- 请求：

| Body_Field_Name |  Type  |  Whether Required |  Description   |
|:----------:|:------:| :------:| :--------------:|
|   acct_no | String |  Required |  要提现的用户编号|
|   cust_tx_id | String |  Optional |  机构端交易流水号|
|   coin_type | String | Required |  提现的币种|
|   address | String |  Required |  用户的提取coin地址 |
|   tx_amount | String |  Required |  提现金额|
|   remark | String | Optional |  备注|



- 响应：

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
|    tx_id    | String | 币种提现id|



### 1.5.获取的用户提现记录

```text
url：/api/v1/npay/cust/withdrawal
method：GET
```

- 请求：

|  Field_Name   |  Type  |  Whether Required |        Description         |
| :-----------: | :----: | :------------------------: | :------------------------: |
|  page_num   | int  |  Required |  页数     |
|  page_size  | int  | Required | 页的大小   |
|  acct_no  | String  | Optional | 机构下用戶账号  |
|  cust_tx_id  | String  | Optional | 机构下用戶交易号  |
|  tx_id  | String  | Optional | Noumena下交易号(可选)   |

- 响应：

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
|    acct_no    | String | 机构端用户编号(机构端唯一) |
|    acct_name    | String | 固定为空 |
|    cust_tx_id    |  String   |   绑定公司下交易号          |
|  coin_type   | String |      币种   |
|  tx_amount   | String |      金额   |
| bonus | String | 固定是0 |
| bonus_coin_type | String | 固定为空 |
| tx_id | String | Noumena交易ID |
|  tx_type   | String |      交易类型：deposit 或 withdraw   |
| tx_status | int | 交易状态. 0 处理中， 1 成功， 2 失败 |
|   address | String |   用户的提取coin地址 |
|  create_time   | long |      创建日期   |
|  update_time   | long |      更新日期   |



### 1.6.获取的用户资产

```text
url：/api/v1/npay/cust/user/balance?acct_no=123&coin_type=USDT
method：GET
```

- 请求：

|  Field_Name   |  Type  | Whether Required |        Description         |
| :-----------: | :----: | :------------------------: | :------------------------: |
|  acct_no  | String  | Required | 机构下用戶账号  |
|  coin_type  | String  | Optional | Coin 类型    |

- 响应：

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
|  balance   | String |      剩余金额   |
|  deposit   | String |      充值总额   |
|  withdraw   | String |      提现总额   |


