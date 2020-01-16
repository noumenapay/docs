# noumena-pay-OpenAPI 

* [接口规范](#接口规范)
* [1.Noumena Pay 接口](#1.-Noumena-Pay-接口)

## 接口规范

- Open API 请求都使用 `HMAC` 认证，以保证请求的完整性，真实性，同时做身份认证和鉴权。

- **分页**。查询记录列表都有分页，分页参数：`page_num` 表示页数，`page_size` 表示每页大小。接口 `DTO` 统一返回 `total`，`records`。

- **国家**。两位国家代码，参照 `ISO 3166-1 alpha-2` 编码标准

- 时间处理。API 请求和返回时间都是 `UNIX` 时间戳，**秒为单位**，避免因为时区导致误差

- 金额处理。API 请求和返回都是 `String` 字符串类型，避免精度丢失

- 所有带 `body` 的请求没有特殊说明body都是 `JSON`格式，`Content-Type：application/json`

- 接口返回格式统一：

  | Parameter |  Type  |           Description            |
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

```java
amount=190&ont_id=did:ont:Ae9ujqUnAtH9yRiepRvLUE3t9R2NbCTZPG&to_address=AUol16ghiT9AtxRDtNeq3ovhWJ5iaY6iyd
```


#### 实现代码示例：

请参考：[https://github.com/noumenapay/noumena-sdk-java](https://github.com/noumenapay/noumena-sdk-java)

## 1. Noumena Pay 接口

### 1.1.给用户充值


```text
url：/api/v1/npay/cust/transaction
method：POST
```


- 请求：

| Parameter |  Type  |   Description   |
|:----------:|:------:|:---------------------------------------------------------------------:|
|   acct_no | String |要充值的用户编号|
|   cust_user_no | String |机构端用户编号(机构端唯一)|
|   cust_tx_id | String |机构端交易流水号(可以空)|
|   coin_type | String |充值的币种|
|   tx_amount | String |充值金额|
|   bonus_coin_type | String |奖励币种(可以为空)|
|   bonus_tx_amount | String |奖励币种金额(bonus_coin_type为空则可以为空,否则不能为空)|
|   remark | String |备注(可以空)|



- 响应：

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
|  Field_Name   |  Type  |        Description         |
| :-----------: | :----: | :------------------------: |
|    tx_id    | String | 主币种充值id|
|    bonus_txid    | String | bonus币种充值id |






### 1.2.获取的用户交易记录

```text
url：/api/v1/npay/cust/transaction
method：GET
```

- 请求：

|  Parameter   |  Type  |        Description         |
| :-----------: | :----: | :------------------------: |
|  page_num   | int  |    页数     |
|  page_size  | int  |  页的大小   |
|  acct_no  | String  |  onto 用戶账号(可选)   |
|  cust_user_no  | String  | 绑定公司下用户编号(可选)   |
|  cust_tx_id  | String  | 绑定公司下交易号(可选)   |

- 响应：

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
		"total": 1
	  }
	}
```
|  Field_Name   |  Type  |        Description         |
| :-----------: | :----: | :------------------------: |
|    acct_no    | String | 机构端用户编号(机构端唯一) |
| bonus | String | 奖励 |
| bonus_coin_type | String | 奖励币种 |
|  create_time   | long |      创建日期   |
|    cust_user_no    |  String   |   绑定公司下用户编号          |
|    cust_tx_id    |  String   |   绑定公司下交易号          |
|  coin_type   | String |      币种   |
|  tx_amount   | String |      金额   |








