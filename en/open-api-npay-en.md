# noumena-pay-OpenAPI 

* [接口规范](#接口规范)
* [Noumena Pay 接口](#1.-Noumena-Pay-接口)

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

```java
amount=190&ont_id=did:ont:Ae9ujqUnAtH9yRiepRvLUE3t9R2NbCTZPG&to_address=AUol16ghiT9AtxRDtNeq3ovhWJ5iaY6iyd
```


#### 实现代码示例：

```java


import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import com.github.ontio.network.rest.http;
import com.onchain.noumena.util.HmacSHA256Base64Util;
import okhttp3.*;
import org.junit.Test;

import java.io.IOException;
import java.util.HashMap;
import java.util.Map;
import java.util.TreeMap;

/**
 * @author 
 * @version 1.0
 * @date 2019/12/11
 */
public class HmacTest {

    private static final String appKey = "9781d1588a8241918eeb908e39dd4f24";
    private static final String appSecret = "31da0108-5d1b-44ae-8b2c-054778d140e0";
    private static final String hmacScheme = "Noumena";
    
    public String getAuthorizationStr(String method, String requestPath, String requestQueryStr, JSONObject reqBody) throws Exception {
        String timeStampStr = String.valueOf(System.currentTimeMillis());
        TreeMap map = JSONObject.parseObject(reqBody.toJSONString(), TreeMap.class);
        String sign = HmacSHA256Base64Util.sign(timeStampStr, method, requestPath,
                requestQueryStr, appKey, appSecret, map);
        String authorizationStr = hmacScheme +":"+ appKey +":"+ timeStampStr+":" + sign;
        return authorizationStr;
    }
    @Test
    public void getaccount() throws Exception {
        String requestPath = "/api/v1/customers/accounts/6/kyc-status";
        JSONObject reqBody = new JSONObject();
        String authorizationStr = getAuthorizationStr("POST", requestPath, "", reqBody);
        Map<String, String> header = new HashMap<>();
        header.put("Authorization", authorizationStr);
        header.put("Access-Passphrase", "12345678a");

        String result = http.get("http://172.168.3.30:8585/"+requestPath, header);
        System.out.println(result);
    }

}

```



```java

import com.onchain.noumena.exception.AuthenticationException;
import com.onchain.noumena.model.common.ErrorCode;
import com.onchain.noumena.model.enums.AlgorithmEnum;
import com.onchain.noumena.model.enums.CharsetEnum;
import lombok.extern.slf4j.Slf4j;
import org.springframework.util.Base64Utils;
import org.springframework.util.CollectionUtils;
import org.springframework.util.StringUtils;

import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import javax.management.RuntimeErrorException;
import java.io.UnsupportedEncodingException;
import java.net.URLDecoder;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;
import java.util.Set;
import java.util.TreeMap;

@Slf4j
public class HmacSHA256Base64Util {

    public static String sign(String timestamp, String method, String requestPath,
                              String queryString, String appKey, String secretKey, TreeMap<String, String> body)
            throws CloneNotSupportedException, InvalidKeyException, UnsupportedEncodingException {
        if (StringUtils.isEmpty(secretKey) || StringUtils.isEmpty(method)) {
            throw new AuthenticationException(ErrorCode.BAD_REQUEST);
        }
        String preHash = preHash(timestamp, method, requestPath, queryString, appKey, body);
        log.info("origin sign data:{}",preHash);
        byte[] secretKeyBytes = secretKey.getBytes(CharsetEnum.UTF_8.charset());
        SecretKeySpec secretKeySpec = new SecretKeySpec(secretKeyBytes, AlgorithmEnum.HMAC_SHA256.algorithm());
        Mac mac = (Mac) MAC.clone();
        mac.init(secretKeySpec);
        return Base64Utils.encodeToString(mac.doFinal(preHash.getBytes(CharsetEnum.UTF_8.charset())));
    }

    /**
     * the prehash string = timestamp + method + requestPath + body .<br/>
     *
     * @param timestamp   the number of seconds since Unix Epoch in UTC. Decimal values are allowed.
     *                    eg: 2018-03-08T10:59:25.789Z
     * @param method      eg: POST
     * @param requestPath eg: /orders
     * @param queryString eg: before=2&limit=30
     *                    //     * @param body        json string, eg: {"product_id":"BTC-USD-0309","order_id":"377454671037440"}
     * @return prehash string eg: 2018-03-08T10:59:25.789ZPOST/orders?before=2&limit=30{"product_id":"BTC-USD-0309",
     * "order_id":"377454671037440"}
     */
    public static String preHash(String timestamp, String method, String requestPath, String queryString, String appKey, TreeMap<String, String> body) throws UnsupportedEncodingException {
        StringBuilder preHash = new StringBuilder();
        preHash.append(timestamp);
        preHash.append(method.toUpperCase());
        preHash.append(appKey);
        preHash.append(requestPath);
        if (!StringUtils.isEmpty(queryString)) {
            preHash.append(APIConstants.QUESTION).append(URLDecoder.decode(queryString, "UTF-8"));
        }
        if (!CollectionUtils.isEmpty(body)) {
            preHash.append(appendBody(body));
        }
        return preHash.toString();
    }

    public static String appendBody(TreeMap<String, String> params) {
        StringBuilder str = new StringBuilder("");
        Set<String> setKey = params.keySet();
        for (String key : setKey) {
            str.append(key).append("=").append(String.valueOf(params.get(key))).append("&");
        }
        String strBody = str.toString();
        if(!StringUtils.isEmpty(strBody)){
            //删除最后一个拼接符
            strBody = strBody.substring(0,strBody.length()-1);
        }
        return strBody;
    }

    public static Mac MAC;

    static {
        try {
            MAC = Mac.getInstance(AlgorithmEnum.HMAC_SHA256.algorithm());
        } catch (NoSuchAlgorithmException e) {
            throw new RuntimeErrorException(new Error("Can't get Mac's instance."));
        }
    }
}

```

## 1. Noumena Pay 接口

### 1.1.给用户充值


```text
url：/api/v1/npay/cust/transaction
method：POST
```


- 请求：

| Body_Field_Name |  Type  |   Description   |
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

|  Field_Name   |  Type  |        Description         |
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
			"acct_no": "12345678",
			"bonus": "10",
			"bonus_coin_type": "ONT",
			"create_time": 0,
			"cust_user_no": "mid123",
			"cust_tx_id":"1",
			"coinType":"PAX",
			"tx_amount": "126"
		  }
		],
		"total": 0
	  }
	}
```
|  Field_Name   |  Type  |        Description         |
| :-----------: | :----: | :------------------------: |
|    acct_no    | String | 机构端用户编号(机构端唯一) |
| bonus | String | 奖励 |
| bonus_coin_type | String | 奖励币种 |
|  create_time   | long |      创建日期   |
|   cust_id   | int |          机构id          |
|  cust_name   | String |      机构名称   |
|    cust_user_no    |  String   |   绑定公司下用户编号          |
|    cust_tx_id    |  String   |   绑定公司下交易号          |
|    logo    |  String   |          公司图标          |
|  coin_type   | String |      币种   |
|  tx_amount   | String |      金额   |




### 1.3.获取的用户资产总额

```text
url：/api/v1/npay/cust/asset?acct_no=123
method：GET
```

- 请求：

- 响应：

```json
	{
	  "code": 0,
	  "msg": "string",
	  "result": {
		"ont_balance": "8",
		"ont_expenditure": "2",
		"ont_income": "10",
		"pax_balance": "10",
		"pax_expenditure": "10",
		"pax_income": "20"
	  }
	}
```
|  Field_Name   |  Type  |        Description         |
| :-----------: | :----: | :------------------------: |
|  ont_balance   | String |     ONT资产总余额   |
|  ont_income   | String |      ONT收入金额   |
|  ont_expenditure   | String |      ONT支出金额   |
|  pax_balance   | String |      PAX资产总余额   |
|  pax_income   | String |      PAX收入金额   |
|  pax_expenditure   | String |     PAX支出金额   |




