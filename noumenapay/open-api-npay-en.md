# Noumena pay OpenAPI

* [API Specifications](#1-api-specifications)
* [Noumena Pay API](#2-noumena-pay-api)

## 1. API Specifications

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
            //Delete the final linked string
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

## 2. Noumena Pay API

### 2.1. Deposit funds for a user


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
|  Field_Name   |  Type  |        Description         |
| :-----------: | :----: | :------------------------: |
|    tx_id    | String | Funds transaction ID|
|    bonus_txid    | String | Bonus transaction ID |






### 2.2. Fetch user's transaction records

```text
url：/api/v1/npay/cust/transaction
method：GET
```

- Request:

|  Field_Name   |  Type  |        Description         |
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
|  Field_Name   |  Type  |        Description         |
| :-----------: | :----: | :------------------------: |
|    acct_no    | String | User institution level ID (unique)|
| bonus | String | Bonus |
| bonus_coin_type | String | Bonus currency type |
|  create_time   | long |      Creation time   |
|    cust_user_no    |  String   |   Linked institution ID          |
|    cust_tx_id    |  String   |   Linked transaction ID          |
|  coin_type   | String |      Currency type  |
|  tx_amount   | String |      Transaction amount   |






