# 1. API Usage Guide

# Request structure

This article introduces the request method, access address and request header parameters in the cloud development API request.

## Request Method

The Cloud Development Platform API supports the following request methods.

| Method | Description |
| :----- | :--------------------- |
| GET | Request the server to return the specified resource |
| PUT | Request the server to update the specified resource |
| POST | Requests the server to perform a specific operation |
| DELETE | Request the server to delete a specified resource |

> **Note**: When `Body` needs to pass parameters, the `Content-Type` parameter value is `application/json`.

## Access address

The cloud development platform provides different access addresses for different regions. Please select the access address according to the region where the device is located to shorten the debugging response time.

| Region | Address |
| :---------------- | :--------------------------------- |
| Development Data Center | https://devopenapi.protco.in |
| Test Data Center | https://testopenapi.protco.in |
| Production - India Data Center | https://openapi.protco.in |

## Request header parameters

The following are the commonly used request header parameters when calling the cloud development platform interface. Please be sure to add the **required** parameters in the request header.

| Parameter name | Type | Parameter position | Required | Description |
| :----------- | :----- | :------- | :--- | :--------------------- |
| access_id | String | header | Yes | Authorization ID. |
| sign | String | header | Yes| The signature calculated using the specified signature algorithm. |
| sign_method | String | header | Yes | The signature digest algorithm, fixed to HMAC-SHA256. |
| t | Long | header | Yes| 13-bit standard timestamp. |
| access_token | String | header | Yes | Token information. **Description**: This parameter is not required for obtaining and refreshing Token interfaces. |

## Request Example

Take the demo information API `GET:/v1.0/demo/{id}` as an example: the request to obtain information based on Demo Id: 123 is as follows:

- API Request

  ```
  GET:https://devopenapi.protco.in/v1.0/123
  ```

- Request header

  ```
  access_id: ghGyWEsliWoVu94t****
  sign: DCB5430C226179113D79295B5315664C5541128B5A022F66FA5A83EE6D7F****
  sign_method: HMAC-SHA256
  t: 1677570008000
  access_token: 9wOidq1dFqqWI4r4JK59kNnZ9Cm4****
  ```

## GET request carries body parameters

### JAVA Example:

- Override the createHttpUriRequest method:

```
public class HttpComponentsClientRestfulHttpRequestFactory extends HttpComponentsClientHttpRequestFactory {
    @Override
    protected HttpUriRequest createHttpUriRequest(HttpMethod httpMethod, URI uri) {

        if (httpMethod == HttpMethod.GET) {
            return new HttpGetRequestWithEntity(uri);
        }
        return super.createHttpUriRequest(httpMethod, uri);
    }

    /**
     * Define HttpGetRequestWithEntity to implement the HttpEntityEnclosingRequestBase abstract class to support GET requests with body data
     */

    private static final class HttpGetRequestWithEntity extends HttpEntityEnclosingRequestBase {
        public HttpGetRequestWithEntity(final URI uri) {
            super.setURI(uri);
        }

        @Override
        public String getMethod() {
            return HttpMethod.GET.name();

        }
    }
}
```

- Set the request factory class

```
    String body = "{}";
    HttpHeaders httpHeaders = new HttpHeaders();
    restTemplate.setRequestFactory(new HttpComponentsClientRestfulHttpRequestFactory());
    try {
        ResponseEntity<String> exchange = restTemplate.exchange(requestUrl, HttpMethod.GET, new HttpEntity<>(body, httpHeaders), String.class);
        log.info(exchange.getBody());
    } catch (Exception e) {
        log.error("url: {} http get request exception: {}", requestUrl, e);
    }
```

# Authorization

When calling an API, you need to verify the token and then authorize it. The Cloud Development API follows the OAuth 2.0 protocol standard and uses an implicit authorization method.

## Authorization Process

The authorization process is as follows:

1. Convert the Access ID and Access Secret to a Signature and perform signature authentication.

   How to query `Access ID` and `Access Secret`:

   ```
    Log in to IOT Platform > Cloud Development > Cloud Project > View Cloud Project Details.
   ```

   

   For signature conversion steps, see [Signature Mechanism](https://devdocs.rinoiot.com/1646493111391105024).

2. Verify and issue tokens to third-party cloud platforms.
   ![Description](https://rinioot-iot-dev-1313015441.cos.ap-guangzhou.myqcloud.com/89f8b20e-cb3b-4 6bb-a8a2-17499ee45821-1681439470640-%E6%9C%AA%E5%91%BD%E5%90%8D%E6%96%87%E4%BB%B6%20%281%29.jpg)

## Permission Dimension

The permission dimension that cloud development authorizes to third-party cloud platforms is the developer dimension, that is, only resources that the developer has permission to operate can be operated through Token. For example:

- Developer's app user data
- Device data under developer products
- Device data bound by the user under the developer application

## Get a token

### Interface Description

Developers can create cloud development projects from the IOT platform and apply for cloud API authorization, using a simple mode to implicitly obtain tokens.

### Interface address

```
GET: /v1.0/token
```

### Request Parameters

| Parameter name | Type | Parameter position | Required | Description |
| :----- | :--- | :------- | :------- | :--- |
| | | | | |

### Return Parameters

| Parameter Name | Type | Description |
| :------ | :------- | :------- |
| data | TokenRes | Token data |
| reqId | String | request id |
| time | Integer | Timestamp |
| code | Integer | Status code |
| message | String | message |

**`TokenRes` description**

| Parameter Name | Type | Description |
| :---------- | :------ | :------------------ |
| access_id | String | Authorization id |
| access_token | String | Authorization token |
| expire_time | Integer | Valid time, unit: seconds |
| refresh_token | String | Refresh token |
| tenant_id | String | tenant id |

### Request Example

```
GET: /v1.0/token
```

### Return to Example

```
{
    "reqId": "511714ad9946c030",
    "time": 1681392273,
    "code": 200,
    "message": "success",
    "data": {
        "access_id": "ghGyWEsliWoVu94tCSIO",
        "access_token": "vzPDeDaPGLTLB1JfRhwqrJRUeHq9Td8F",
        "expire_time": 7200,
        "refresh_token": "0dMrjUEQdfGGRBL3419m9kBhl04cxV47",
        "tenant_id": "1231"
    }
}
```

## Refresh Token

### Interface Description

Each access_token is valid for two hours. After it expires, you need to use a refresh_token to replace the previous token. The refresh_token expires in 48 hours. After it expires, you need to obtain a new token.

### Interface address

```
GET: /v1.0/token/{refresh_token}
```

### Request Parameters

| Parameter name | Type | Parameter position | Required | Description |
| :---------------- | :----- | :------- | :------- | :------- |
| refresh_token | String | uri | true | refresh token |

### Return to Example

| Parameter Name | Type | Description |
| :------ | :------- | :------- |
| data | TokenRes | Token data |
| reqId | String | request id |
| time | Integer | Timestamp |
| code | Integer | Status code |
| message | String | message |

**`TokenRes` description**

| Parameter Name | Type | Description |
| :---------- | :------ | :------------------ |
| access_id | String | Authorization id |
| access_token | String | Authorization token |
| expire_time | Integer | Valid time, unit: seconds |
| refresh_token | String | Refresh token |
| tenant_id | String | tenant id |

### Request Example

```
GET: /v1.0/token/xxxxxx
```

### Return to Example

```
{
    "reqId": "635fad9797ed23a7",
    "time": 1681436123,
    "code": 200,
    "message": "success",
    "data": {
        "access_id": "ghGyWEsliWoVu94tCSIO",
        "access_token": "CLahHLs1DhCxPWVcD0kFX0dQZmekOBeQ",
        "expire_time": 7200,
        "refresh_token": "wOwYW09TcaGGVsmfbx6KvrFeDj2pKhlL",
        "tenant_id": "1231"
    }
}
```

# Signature mechanism

To ensure data security, you need to provide a signature for account verification when initiating an API call request. This article mainly introduces how to generate an API signature.

## Signature Algorithm

The API signature uses the HMAC SHA256 method to create a digest. The signature algorithm is slightly different for the **Token Management API** and **Ordinary Business API**.

| API Type | Token Management API | General Business API |
| :------- | :------------------- | :--------------------------------- |
| Scope | Get Token and Refresh Token APIs | Other APIs except Token Management API |
| Signature algorithm | str = access_id + t + stringToSign sign = HMAC-SHA256(str, access_secret).toUpperCase() | str = client_id + access_token + t + stringToSign sign = HMAC-SHA256(str, access_secret).toUpperCase() |
| Field Description | access_id: Authorization key pair key. t: 13-bit standard timestamp. stringToSign: Signature string. For the generation method, see stringToSign signature string below. secret: Authorization key pair value. access_token: Access token. Obtained through the authorization key pair. | |
| Field Description | Concatenate access_id, the 13-digit standard timestamp (t) of the current request, and stringToSign into the string str to be signed. Then perform HMAC-SHA256 hashing on str and access_secret and convert them to uppercase to obtain the signature. | |

The signature algorithms of the token management API and the common business API differ only in the `str` generation logic. The latter has one more access token (`access_token`) parameter concatenation than the former.

## stringToSign signature string

### Structure

`stringToSign` The signature string structure is as follows:

```
String stringToSign=
HTTPMethod + "\n" +
Content-SHA256 + "\n" +
URL
```

It consists of the following parts:

-HTTPMethod
- Content-SHA256
- URL

`stringToSign` The signature string is obtained by concatenating the above parts with line breaks (\n).

HTTPMethod

HTTPMethod refers to the interface request method, such as GET, POST, PUT, DELETE, etc.

### Content-SHA256

Content-SHA256 refers to the SHA256 value of the request body. SHA256 is calculated only when the body is not a form.

The calculation is as follows:

```
String content-SHA256 = SHA256(bodyStream.getbytes("UTF-8")); //bodyStream is a byte array
```

When the Body is empty, encryption is also performed, and the encryption result is `e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855`.

URL

URL refers to the request path formed by concatenating the request path and form parameters.

- URL concatenation: Sort the form parameters by their keys in alphabetical ascending order according to the dictionary, and concatenate them as follows. If the Query parameter is empty, URL = Path, and there is no need to add a half-width question mark (`?`).

```TEXT
  String url =
  Path +
  "?" +
  Key1 + "=" + Value1 +
  "&" + Key2 + "=" + Value2 +
  "&" + Key3 +
  ...
  "&" + KeyN + "=" + ValueN
```

  For example:

```TEXT
  String url = /v1.0/demo/queryPage?size=1&page=10
```

- URL parameter sorting: Sort in natural ascending order of the parameters' letters. If the letters are the same, compare to the next one.

  For example, there are the following parameters:

  -size=1
  - page=10

  The sorted rules are:

  - page=10
  -size=1

## Token Management API Example

Take the Get Token interface as an example, the body parameter is empty.

```TEXT
GET:/v1.0/token
```

The request header structure is as follows:

| Parameter | Value |
| :--------------------------------------------- | :------------------- |
| method | GET |
| access_id | ghGyWEsliWoVu94tCSIO |
| access_secret (no parameter passing, only used for signature encryption) | pKZym70k5RJUnisk9jOV |
| t | 1677570008000 |
| sign_method | HMAC-SHA256 |

### Concatenate stringToSign signature string

First, concatenate the `stringToSign` signature string:

```TEXT
GET
e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
/v1.0/demo/123
```

### Spell the reception signature string

After getting `stringToSign`, you can compose the reception signature string:

```TEXT
ghGyWEsliWoVu94tCSIO1677570008000GET
e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
/v1.0/token
```

### HMAC SHA256 creates digest

Hash digest:

```TEXT
HMAC-SHA256(ghGyWEsliWoVu94tCSIO1677570008000GET
e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
/v1.0/token,pKZym70k5RJUnisk9jOV)
```

- Hash the digest to get the new string and convert it to uppercase:

  `2BEB6AE98F9B235929E6AF59DC5FB8F388348F7D8AB186F6FBBA0DB3528E7487`

## Common business API examples

Take the demo information API as an example. The value of the access_token parameter is vzPDeDaPGLTLB1JfRhwqrJRUeHq9Td8F, and the body parameter is empty.

| Parameter | Value |
| :--------------------------------------------- | :----------------------------- |
| URL | /v1.0/demo/123 |
| method | GET |
| access_id | ghGyWEsliWoVu94tCSIO |
| access_secret (no parameter passing, only used for signature encryption) | pKZym70k5RJUnisk9jOV |
| t | 1677570008000 |
| access_token | vzPDeDaPGLTLB1JfRhwqrJRUeHq9Td8F |
| sign_method | HMAC-SHA256 |

### Concatenate stringToSign signature  string

First, concatenate the `stringToSign` signature string:

```TEXT
GET
e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
/v1.0/demo/123
```

### Spell the reception signature  string

After getting `stringToSign`, you can compose the reception signature string:

```TEXT
ghGyWEsliWoVu94tCSIOvzPDeDaPGLTLB1JfRhwqrJRUeHq9Td8F1677570008000GET
e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
/v1.0/demo/123
```

### HMAC SHA256 creates  digest

Hash digest:

```TEXT
HMAC-SHA256(ghGyWEsliWoVu94tCSIOvzPDeDaPGLTLB1JfRhwqrJRUeHq9Td8F1677570008000GET
e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
/v1.0/demo/123,vzPDeDaPGLTLB1JfRhwqrJRUeHq9Td8F)
```

- Hash the digest to get the new string and convert it to uppercase:

  `2823AEAE6BECD6C60EF04B7F7CDDD34465B0612DA5AB4822071882124E195C2D`

## HMAC SHA256 Implementation

### Java Example

#### Signature generation example

```JAVA
public static void main(String[] args){
        String mehtod = "GET";
        String body = "";
        String url = "https://devopenapi.rinoiot.com/v1.0/demo/123";
        String accessId = "ghGyWEsliWoVu94tCSIO";
        String secret = "pKZym70k5RJUnisk9jOV";
        String t = "1677570008000";

        String accessToken="vzPDeDaPGLTLB1JfRhwqrJRUeHq9Td8F";
// String accessToken=null;
        List<String> lines = new ArrayList<>(16);
        lines.add(mehtod);
        String bodyHash = EMPTY_HASH;
        if (body != null && body.length() > 0) {
            try {
                bodyHash = Sha256Util.encryption(body);
            } catch (Exception e) {
                log.error("Generate signature string exception",e);
            }
        }
        lines.add(bodyHash);
        String paramSortedPath = null;
        try {
            paramSortedPath = getPathAndSortParam(new URL(url));
        } catch (MalformedURLException e) {
            log.error("Signature construction URL exception",e);
        }
        lines.add(paramSortedPath);
        String stringToSign = String.join(SPLIT_CHAR, lines);
        System.out.println("stringToSign:"+stringToSign);
        String sign = sign(accessId,secret,t,accessToken,stringToSign);
        System.out.println("Signature:"+sign);
    }
```

#### Signature tool class  

```JAVA
package com.rinoiot.openapi.utils;

import cn.hutool.core.util.StrUtil;
import cn.hutool.json.JSONUtil;
import com.baomidou.mybatisplus.core.toolkit.StringUtils;
import lombok.extern.slf4j.Slf4j;
import java.net.MalformedURLException;
import java.net.URL;
import java.net.URLDecoder;
import java.util.*;
import java.util.stream.Collectors;

@Slf4j
public class OpenApiSignUtil {
    private static final String EMPTY_HASH = "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855";
    private static final String SPLIT_CHAR = "\n";


    public static String sign(String accessId, String secret, String t, String accessToken, String stringToSign) {
        StringBuilder sb = new StringBuilder();
        sb.append(accessId);
        if (StrUtil.isNotBlank(accessToken)) {
            sb.append(accessToken);
        }
        sb.append(t);
        sb.append(stringToSign);
        System.out.println("Spell reception encryption signature: "+sb.toString());
        return Sha256Util.sha256HMAC(sb.toString(), secret);
    }

    public static String stringToSign(RequestWrapper request) {
        List<String> lines = new ArrayList<>(16);
        lines.add(request.getMethod().toUpperCase());
        String bodyHash = EMPTY_HASH;
        if (request.getBody() != null && request.getBody().length > 0) {
            try {
                bodyHash = Sha256Util.encryption(request.getBody());
            } catch (Exception e) {
                log.error("Generate signature string exception",e);
            }
        }
        lines.add(bodyHash);
        String wholeUrl = request.getRequestURL().toString();
        if(StrUtil.isNotBlank(request.getQueryString())){
            wholeUrl += "?"+request.getQueryString();
        }
        String paramSortedPath = null;
        try {
            paramSortedPath = getPathAndSortParam(new URL(wholeUrl));
        } catch (MalformedURLException e) {
            log.error("Signature construction URL exception",e);
        }
        lines.add(paramSortedPath);
        return String.join(SPLIT_CHAR, lines);
    }

    private static String getPathAndSortParam(URL url) {
        try {
            String query = "";
            if(StringUtils.isNotBlank(url.getQuery())){
                query = URLDecoder.decode(url.getQuery(), "UTF-8");
            }
            String path = url.getPath();
            path = path.replace("/platform","");
            if (StrUtil.isBlank(query)) {
                return path;
            }
            Map<String, String> kvMap = new TreeMap<>();
            String[] kvs = query.split("\\&");
            for (String kv : kvs) {
                String[] kvArr = kv.split("=");
                if (kvArr.length > 1) {
                    kvMap.put(kvArr[0], kvArr[1]);
                } else {
                    kvMap.put(kvArr[0], "");
                }
            }
            return path + "?" + kvMap.entrySet().stream().map(it -> it.getKey() + "=" + it.getValue())
                    .collect(Collectors.joining("&"));
        } catch (Exception e) {
            log.error("Generate stringToSign abnormal url:{}",e, JSONUtil.toJsonStr(url));
            return url.getPath();
        }
    }
}
```

#### HMAC SHA256 encryption algorithm tool class

```JAVA
package com.rinoiot.openapi.utils;

import lombok.extern.slf4j.Slf4j;

import javax.crypto.Mac;
import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;
import javax.xml.bind.annotation.adapters.HexBinaryAdapter;
import java.nio.charset.StandardCharsets;
import java.security.InvalidKeyException;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;

@Slf4j
public class Sha256Util {

    public static String encryption(String str) throws Exception {
        return encryption(str.getBytes(StandardCharsets.UTF_8));
    }

    public static String encryption(byte[] buf) throws Exception {
        MessageDigest messageDigest;
        messageDigest = MessageDigest.getInstance("SHA-256");
        messageDigest.update(buf);
        return byte2Hex(messageDigest.digest());
    }

    private static String byte2Hex(byte[] bytes) {
        StringBuilder stringBuffer = new StringBuilder();
        String temp;
        for (byte aByte : bytes) {
            temp = Integer.toHexString(aByte & 0xFF);
            if (temp.length() == 1) {
                stringBuffer.append("0");
            }
            stringBuffer.append(temp);
        }
        return stringBuffer.toString();
    }

    public static String sha256HMAC(String content, String secret) {
        Mac sha256HMAC = null;
        try {
            sha256HMAC = Mac.getInstance("HmacSHA256");
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        }
        SecretKey secretKey = new SecretKeySpec(secret.getBytes(StandardCharsets.UTF_8), "HmacSHA256");
        try {
            sha256HMAC.init(secretKey);
        } catch (InvalidKeyException e) {
            log.error("Conversion signature string exception content:{},secret:{}",content,secret,e);
        }
        byte[] digest = sha256HMAC.doFinal(content.getBytes(StandardCharsets.UTF_8));
        return new HexBinaryAdapter().marshal(digest).toUpperCase();
    }
}
```

# Global error code 

When an error occurs when calling an API, a custom error message will be returned. You can refer to this article to learn about the global error code of Cloud Development.

## System abnormal error code

System exception error codes refer to error codes starting with `500`. Common error scenarios include signature problems, parameter transmission errors, system timeout problems, etc.

| Error code | Error message | Description |
| :----- | :--------------------- | :------------- |
| 50009 | Missing parameter | Missing parameter |
| 50010 | access_token error | access_token error |
| 50011 | access_token error | access_token(refresh_token) error |
| 50012 | Signature error | Signature error |
| 50013 | Permission verification failed | Permission verification failed |
| 50014 | access token expired, please refresh the authorization token | access_token expired, please refresh the authorization token |
| 50015 | refresh token expired, please re-authorize the token | refresh_token expired, please re-authorize the token |
| 50016 | access_id error | access_id error |

## Business exception error code

Business exception error codes refer to error codes starting with `60`.

| Error code | Error message | Description |
| :----- | :------- | :--- |
| | | |

# 2. API List

## Get the Agora License

**Interface address**:`/platform/v1.0/agora/getAgoraLicense`

**Request method**: `GET`

**Request data type**: `application/x-www-form-urlencoded`

**Response data type**: `*/*`

**Interface Description**:

**Request Parameters**:

**Request Parameters**:

| Parameter name | Parameter description | Request type | Required | Data type | Schema |
| ---------- | -------- | -------- | -------- | -------- | ------ |
| deviceUuid | deviceuuid | query | false | string | |

**Response Status**:

| Status code | Description | Schema |
| ------ | ---------- | ----------- |
| 200 | OK | ResponseResult object «AgoraLicenseVO object» |
| 401 | Unauthorized | |
| 403 | Forbidden | |
| 404 | Not Found | |

**Response parameters**:

| Parameter name | Parameter description | Type | schema |
| ---------------- | ---------------- | ------------------ | ------------------ |
| code | Response status code | integer(int32) | integer(int32) |
| data | Response data | AgoraLicenseVO object | AgoraLicenseVO object |
| activeTime | SoundNet activation time | string | |
| apiLicense | SoundNet License | string | |
| apiLicenseKey | SoundNet license key | string | |
| apiPid | aural network pid | string | |
| appId | Agora appId | string | |
| bindTime | bind time | string | |
| createTime | creation time | string | |
| deviceId | device id | string | |
| deviceName | device name | string | |
| deviceUuid | device uuid | string | |
| expireTime | SoundNet expiration time | string | |
| isTest | whether to test | boolean | |
| licenseId | primary key id | string | |
| productId | product id | string | |
| productName | product name | string | |
| skuData | sku data | string | |
| tenantId | tenant id | string | |
| updateTime | update time | string | |
| message | response message | string | |
| reqId | request ID | string | |
| time | request timestamp (10 bits) | integer(int64) | integer(int64) |

**Sample response**:

```JSON
{
    "code": 200,
    "data": {
        "activeTime": "",
        "apiLicense": "",
        "apiLicenseKey": "",
        "apiPid": "",
        "appId": "",
        "bindTime": "",
        "createTime": "",
        "deviceId": "",
        "deviceName": "",
        "deviceUuid": "",
        "expireTime": "",
        "isTest": true,
        "licenseId": "",
        "productId": "",
        "productName": "",
        "skuData": "",
        "tenantId": "",
        "updateTime": ""
    },
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

## Get the Agora Token Get the Agora Token

**Interface address**:`/platform/v1.0/agora/getAgoraToken`

**Request method**: `GET`

**Request data type**: `application/x-www-form-urlencoded`

**Response data type**: `*/*`

**Interface Description**:

**Request Parameters**:

**Request Parameters**:

| Parameter name | Parameter description | Request type | Required | Data type | Schema |
| ----------- | -------- | -------- | -------- | ----------- | ------ |
| agoraUid | Agora UID | query | false | integer(int32) | |
| channelName | channel name | query | false | string | |

**Response Status**:

| Status code | Description | Schema |
| ------ | ---------- | --------------- |
| 200 | OK | ResponseResult object «AgoraUidTokenVO» |
| 401 | Unauthorized | |
| 403 | Forbidden | |
| 404 | Not Found | |

**Response parameters**:

| Parameter name | Parameter description | Type | schema |
| ------------- | ------------- | --------------- | ------------- |
| code | Response status code | integer(int32) | integer(int32) |
| data | Response data | AgoraUidTokenVO | AgoraUidTokenVO |
| rtcToken | rtc token | AgoraRtcTokenVO | AgoraRtcTokenVO |
| channelName | channel name | string | |
| expireSecond | Expiration time (seconds) | integer | |
| rtcToken | rtc token | string | |
| uid | aural network uid | integer | |
| rtmToken | rtm token | AgoraRtmTokenVO | AgoraRtmTokenVO |
| account | rtm account | string | |
| expireSecond | Expiration time (seconds) | integer | |
| rtmToken | rtm token | string | |
| message | response message | string | |
| reqId | request ID | string | |
| time | request timestamp (10 bits) | integer(int64) | integer(int64) |

**Sample response**:

```JSON
{
    "code": 200,
    "data": {
        "rtcToken": {
            "channelName": "",
            "expireSecond": 0,
            "rtcToken": "",
            "uid": 0
        },
            "rtmToken": {
            "account": "",
            "expireSecond": 0,
            "rtmToken": ""
        }
    },
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

## Create DP group

**Interface address**:`/v1.0/dpGroup/add`

**Request method**: `POST`

**Request data type**: `application/json`

**Response data type**: `*/*`

**Interface Description**:

**Request Example**:

```JSON
{
  "dpGroupId": "",
  "gatewayId": "",
  "gatewayUuid": "",
  "name": "",
  "refList": [
    {
      "deviceId": "",
      "dpGroupId": "",
      "dpIdentifier": "",
      "refId": ""
    }
  ],
  "rootAssetId": ""
}
```

**Request Parameters**:

| Parameter name | Parameter description | Request type | Required | Data type | Schema |
| ------------- | ---------------------- | ------------- | -------- | ----- | ------- |
| appDpGroupDTO | appDpGroupDTO | body | true | AppDpGroupDTO object | AppDpGroupDTO object |
| dpGroupId | primary key ID | | false | string | |
| gatewayId | Gateway ID (automatically detected and filled in by the backend) | | false | string | |
| gatewayUuid | Gateway UUID (automatically detected and filled in by the background) | | false | string | |
| name | name | | false | string | |
| refList | device dp list | | false | array | AppDpGroupRefDTO object |
| deviceId | device ID | | false | string | |
| dpGroupId | dp binding ID | | false | string | |
| dpIdentifier | dp identifier | | false | string | |
| refId | primary key ID | | false | string | |
| rootAssetId | Root asset ID | | false | string | |

**Response Status**:

| Status code | Description | Schema |
| ------ | ------------ | -------------------------- |
| 200 | OK | ResponseResult object «string» |
| 201 | Created | |
| 401 | Unauthorized | |
| 403 | Forbidden | |
| 404 | Not Found | |

**Response parameters**:

| Parameter name | Parameter description | Type | schema |
| -------- | ---------------- | ------------- | ----------- |
| code | Response status code | integer(int32) | integer(int32) |
| data | response data | string | |
| message | response message | string | |
| reqId | request ID | string | |
| time | request timestamp (10 bits) | integer(int64) | integer(int64) |

**Sample response**:

```JSON
{
    "code": 200,
    "data": "",
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

## Update DP group

**Interface address**:`/v1.0/dpGroup/updateById`

**Request method**: `POST`

**Request data type**: `application/json`

**Response data type**: `*/*`

**Interface Description**:

**Request Example**:

```JSON
{
  "dpGroupId": "",
  "gatewayId": "",
  "gatewayUuid": "",
  "name": "",
  "refList": [
    {
      "deviceId": "",
      "dpGroupId": "",
      "dpIdentifier": "",
      "refId": ""
    }
  ],
  "rootAssetId": ""
}
```

**Request Parameters**:

| Parameter name | Parameter description | Request type | Required | Data type | Schema |
| ------------- | -------------------------- | -------- | -------- | ----------------- | -------------------- |
| appDpGroupDTO | appDpGroupDTO | body | true | AppDpGroupDTO object | AppDpGroupDTO object |
| dpGroupId | primary key ID | | false | string | |
| gatewayId | Gateway ID (automatically detected and filled in by the backend) | | false | string | |
| gatewayUuid | Gateway UUID (automatically detected and filled in by the background) | | false | string | |
| name | name | | false | string | |
| refList | device dp list | | false | array | AppDpGroupRefDTO object |
| deviceId | device ID | | false | string | |
| dpGroupId | dp binding ID | | false | string | |
| dpIdentifier | dp identifier | | false | string | |
| refId | primary key ID | | false | string | |
| rootAssetId | Root asset ID | | false | string | |

**Response Status**:

| Status code | Description | Schema |
| ------ | ---------- | ---------------------------- |
| 200 | OK | ResponseResult object «boolean» |
| 201 | Created | |
| 401 | Unauthorized | |
| 403 | Forbidden | |
| 404 | Not Found | |

**Response parameters**:

| Parameter name | Parameter description | Type | schema |
| -------- | ---------------- | ------------- | -------- ------ |
| code | Response status code | integer(int32) | integer(int32) |
| data | response data | boolean | |
| message | response message | string | |
| reqId | request ID | string | |
| time | request timestamp (10 bits) | integer(int64) | integer(int64) |

**Sample response**:

```JSON
{
    "code": 200,
    "data": true,
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 16206417
}
```

## Delete DP group

**Interface address**:`/v1.0/dpGroup/deleteById`

**Request method**: `DELETE`

**Request data type**: `application/x-www-form-urlencoded`

**Response data type**: `*/*`

**Interface Description**:

**Request Parameters**:

| Parameter name | Parameter description | Request type | Required | Data type | Schema |
| -------- | -------- | -------- | -------- | -------- | ------ |
| id | id | query | true | string | |

**Response Status**:

| Status code | Description | Schema |
| ------ | ---------- | ---------------------------- |
| 200 | OK | ResponseResult object «boolean» |
| 204 | No Content | |
| 401 | Unauthorized | |
| 403 | Forbidden | |

**Response parameters**:

| Parameter name | Parameter description | Type | schema |
| -------- | ---------------- | ------------- | -------------- |
| code | Response status code | integer(int32) | integer(int32) |
| data | response data | boolean | |
| message | response message | string | |
| reqId | request ID | string | |
| time | request timestamp (10 bits) | integer(int64) | integer(int64) |

**Sample response**:

```JSON
{
    "code": 200,
    "data": true,
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

## Query DP group by ID

**Interface address**:`/v1.0/dpGroup/getById`

**Request method**: `GET`

**Request data type**: `application/x-www-form-urlencoded`

**Response data type**: `*/*`

**Interface Description**:

**Request Parameters**:

| Parameter name | Parameter description | Request type | Required | Data type | Schema |
| -------- | -------- | -------- | -------- | -------- | ------ |
| id | id | query | true | string | |

**Response Status**:

| Status code | Description | Schema |
| ------ | ---------- | ------------- |
| 200 | OK | ResponseResult object «AppDpGroupVO object» |
| 401 | Unauthorized | |
| 403 | Forbidden | |
| 404 | Not Found | |

**Response parameters**:

| Parameter name | Parameter description | Type | schema |
| ------------ | ---------------------- | ------------- | ------------------- |
| code | Response status code | integer(int32) | integer(int32) |
| data | response data | AppDpGroupVO object | AppDpGroupVO object |
| createTime | creation time | string | |
| dpGroupId | primary key ID | string | |
| gatewayId | Gateway ID | string | |
| gatewayUuid | Gateway UUID | string | |
| localStatus | Local synchronization status: 0 not synchronized; 1 synchronized | boolean | |
| name | name | string | |
| refVOList | Update time | array | AppDpGroupRefVO object |
| createTime | creation time | string | |
| deviceId | device ID | string | |
| dpGroupId | dp binding ID | string | |
| dpIdentifier | dp identifier | string | |
| refId | primary key ID | string | |
| updateTime | update time | string | |
| rootAssetId | root asset ID | string | |
| shortId | local short ID | integer | |
| updateTime | update time | string | |
| message | response message | string | |
| reqId | request ID | string | |
| time | request timestamp (10 bits) | integer(int64) | integer(int64) |

**Sample response**:

```JSON
{
    "code": 200,
    "data": {
        "createTime": "",
        "dpGroupId": "",
        "gatewayId": "",
        "gatewayUuid": "",
        "localStatus": true,
        "name": "",
        "refVOList": [
        {
            "createTime": "",
            "deviceId": "",
            "dpGroupId": "",
            "dpIdentifier": "",
            "refId": "",
            "updateTime": ""
        }
        ],
        "rootAssetId": "",
        "shortId": 0,
        "updateTime": ""
    },
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

## Get a list of user scenario instances

**Interface address**:`/v1.0/stream/rule/user/instance/queryList`

**Request method**: `POST`

**Request data type**: `application/json`

**Response data type**: `*/*`

**Interface Description**:

**Request Example**:

```JSON
{
    "assetId": "",
    "assetIds": [],
    "endUpdateTime": "",
    "expireStatus": 0,
    "instanceType": "",
    "instanceTypes": [],
    "isHidden": true,
    "isHome": true,
    "isLocal": true,
    "localGatewayUuid": "",
    "localGatewayUuids": [],
    "ref": {
        "refId": "",
        "refType": ""
    },
    "startUpdateTime": "",
    "status": 0
}
```

**Request Parameters**:

| Parameter name | Parameter description | Request type | Required | Data type | Schema |
| ---------- | ------------------------- | -------- | -------- | --------- | --------- |
| query | query | body | true | StreamRuleUserInstanceQuery object | StreamRuleUserInstanceQuery object |
| appId | application id | | false | string | |
| assetId | asset id | | false | string | |
| assetIds | asset id list | | false | array | string |
| endUpdateTime | Modification end time | | false | string | |
| expireStatus | Expiration status: 0: expired; 1: valid; 2: partially valid | | false | integer | |
| instanceName | instance name | | false | string | |
| instanceType | Instance type (stream-rule engine, onekey-one-key execution, job-scheduled task, redis-redis task) | | false | string | |
| instanceTypes | Instance type array (stream-rule engine, onekey-one-key execution, job-scheduled task, redis-redis task) | | false | array | string |
| isHidden | Whether to hide, hide after unbinding the local scene, and show it after rebinding | | false | boolean | |
| isHome | Whether to display on the home page | | false | boolean | |
| isLocal | Is it a local scene? | | false | boolean | |
| isRefreshWeather | Whether to refresh the weather | | false | boolean | |
| isShow | whether to display | | false | boolean | |
| localGatewayUuid | local scene gateway uuid | | false | string | |
| localGatewayUuids | Local scenario gateway uuid collection | | false | array | string |
| ref | associated condition| | false | Ref | Ref |
| refId | Related ID: Automation ID/Device ID/Group ID | | false | string | |
| refType | Association type (stream_rule: rule engine, device: device, group: group) | | false | string | |
| startUpdateTime | Modify start time | | false | string | |
| status | status (0-disable, 1-enable) | | false | integer | |
| userId | userid | | false | string | |

**Response Status**:

| Status code | Description | Schema |
| ------ | ---------- | ---------------- |
| 200 | OK | ResponseResult object «List«StreamRuleUserInstanceVO objects»» |
| 201 | Created | |
| 401 | Unauthorized | |
| 403 | Forbidden | |
| 404 | Not Found | |

**Response parameters**:

| Parameter name | Parameter description | Type | schema |
| ------------------- | -------- | --------- | ---------- |
| code | Response status code | integer(int32) | integer(int32) |
| data | response data | array | StreamRuleUserInstanceVO object |
| actions | list of actions to be executed | array | StreamRuleUserActionApiApiVO object |
| actionData | action data | string | |
| actionId | primary key id | string | |
| actionType | Action type (device: device action, stream_rule: rule engine, message_center: message center, delayed_action: delayed execution) | string | |
| createTime | creation time | string | |
| delayedAction | DelayedAction | DelayedAction |
| time | Delay time (for example: 05:59:59, cannot exceed 6 hours) | string | |
| delayedTime | Delay time (e.g. 05:59:59, cannot exceed 6 hours) | string | |
| deviceAction | DeviceAction | DeviceAction |
| actionColumn | action field | string | |
| actionColumnType | action type (dp point type) | string | |
| actionColumnUnit | field unit (dp point unit) | string | |
| actionValue | action value (when the value is equal to toggle, it is control inversion, only supports bool type) | object | |
| deviceActionType | Device action type (device: device, group: group) | string | |
| deviceId | device id (must be passed if deviceActionType=device) | string | |
| deviceName | Device name (background field) | string | |
| deviceType | Device type (rino:rino,ty:ty (default:rino)) | string | |
| deviceUuid | device uuid (used in the background) | string | |
| groupDevices | group id (required if deviceActionType=group) | array | GroupDevice |
| deviceId | device ID | string | |
| deviceName | device name | string | |
| status | status | integer | |
| groupId | Group id (required if deviceActionType=group) | string | |
| groupName | Group name (backend field) | string | |
| groupType | Group type (rino:rino,ty:ty (default:rino)) | string | |
| imgUrl | icon (backend field) | string | |
| productId | product id (backend field) | string | |
| productName | Product name (backend field) | string | |
| shortGroupId | Group short id (required if deviceActionType=group) | integer | |
| instanceId | instance id | string | |
| isDelayed | Whether to delay execution | boolean | |
| isExpire | Is it expired? | boolean | |
| messageAction | Message execution data | MessageAction | MessageAction |
| messageType | Message type (message_center: message center, app_push: APP push) | string | |
| refId | associated id (device id) | string | |
| refType | Association type (stream_rule: rule engine, device: device, group: group) | string | |
| sort | sort | integer | |
| status | Synchronization status: 0: no response, 1: success; 2: failure | integer | |
| streamRuleAction | Rule Engine Execution Data | StreamRuleAction | StreamRuleAction |
| instanceIcon | instance icon | string | |
| instanceId | instance id | string | |
| instanceName | instance name | string | |
| ruleActionType | Rule engine execution type (action: execute action, update_status: modify status) | string | |
| status | Status (1: enabled, 0: disabled. ruleActionType=update_status is required) | integer | |
| updateTime | update time | string | |
| activeTime | Effective time (all day: all, daytime: daytime, night: night, custom: 12:00-06:00) | string | |
| address | address | string | |
| appId | application id | string | |
| appName | Application name | string | |
| assetId | asset id | string | |
| conditions | instance conditions | array | StreamRuleUserConditionApiVO object |
| conditionData | condition data (device | job | weather) |
| conditionId | primary key id | string | |
| conditionType | Condition type (device-device condition, job-timing condition, weather-weather condition, onekey-one-key execution) | string | |
| createTime | creation time | string | |
| deviceCondition | DeviceCondition | DeviceCondition |
| conditionColumn | condition field | string | |
| conditionColumnType | field type (dp point type) | string | |
| conditionColumnUnit | field unit (dp point unit) | string | |
| conditionExpress | Conditional expression ((greater than: >, equal to: ==, less than: <)) | string | |
| conditionValue | condition value | object | |
| deviceId | device id | string | |
| deviceName | Device name (background field) | string | |
| deviceType | Device type (rino:rino,ty:ty (default:rino)) | string | |
| deviceUuid | device uuid (used in the background) | string | |
| imgUrl | Device icon (background field) | string | |
| productId | product id (backend field) | string | |
| productName | Product name (backend field) | string | |
| instanceId | instance id | string | |
| isExpire | Is it expired? | boolean | |
| jobCondition | Timing Condition | JobCondition | JobCondition |
| loops | Cycle, 0000000 7 digits starting from Monday, a value of 1 means that execution is allowed on that day; if all are 0, it will be executed only once by default | string | |
| startDate | Start date (yyyy-mm-dd) | string | |
| time | Time, 24-hour formatted time: e.g. 18:00 | string | |
| refId | associated id (device id) | string | |
| refType | associated type (device: device) | string | |
| sort | sort | integer | |
| updateTime | update time | string | |
| weatherCondition | WeatherCondition |
| weatherCode | Weather code (temperature-temperature, humidity-humidity, climate-weather, pm25-PM2.5, aqi-air quality, sun-sunrise and sunset, wind_speed-wind speed) | string | |
| weatherExpress | Weather expression (greater than: >, equal to: ==, less than: <) | string | |
| weatherValue | Temperature (range: -40 to 40, unit: ℃)\nHumidity (dry, comfort, wet)\nWeather (sunny, cloudy, rainy, snowy, mist)\nPM25 (good, fine, polluted)\nAir quality (good, fine, polluted)\nSunrise and sunset (sunrise, sunset)\nWind speed (range: 0-62, unit: m/s) | string | |
| createTime | creation time | string | |
| curCityLat | Current city latitude | string | |
| curCityLon | Current city longitude | string | |
| curCityName | Current city name | string | |
| curCountryName | Current country name | string | |
| dataCenter | data center code | string | |
| deleted | Delete flag (0: normal, 1 deleted) | integer | |
| executeType | Execution type (all conditions are met: &&, any condition is met: | | ) |
| expireStatus | Expiration status: 0: expired; 1: valid; 2: partially valid | integer | |
| instanceDesc | Instance description | string | |
| instanceIcon | instance icon | string | |
| instanceId | primary key id | string | |
| instanceName | instance name | string | |
| instanceType | Instance type (stream-rule engine, onekey-one-key execution, job-scheduled task, redis-redis task) | string | |
| isHome | Whether to display on the home page | boolean | |
| isLocal | Is it a local scene? | boolean | |
| isRefreshWeather | Whether to refresh the weather | boolean | |
| isShow | Whether to display. The default value is false when the local scene is created. The app should not see the scene that is not displayed. | boolean | |
| jobId | scheduled task id | string | |
| lastActionTime | Last execution time | string | |
| lat | latitude | string | |
| localGatewayUuid | local scene gateway uuid | string | |
| lon | longitude | string | |
| repeatLoops | Repeat cycle (0000000, 7 digits starting from Monday, a value of 1 means execution is allowed on that day) | string | |
| shortId | short id (used by gateway) | integer | |
| sort | sort | integer | |
| status | status (0-disable, 1-enable) | integer | |
| timeZone | time zone | string | |
| timeZoneName | time zone name | string | |
| updateTime | update time | string | |
| userId | userid | string | |
| message | response message | string | |
| reqId | request ID | string | |
| time | request timestamp (10 bits) | integer(int64) | integer(int64) |

**Sample response**:

```JSON
{
    "code": 200,
    "data": [
    {
        "actions": [
        {
            "actionData": "",
            "actionId": "",
            "actionType": "",
            "createTime": "",
            "delayedAction": {
            "time": ""
        },
        "delayedTime": "",
        "deviceAction": {
            "actionColumn": "",
            "actionColumnType": "",
            "actionColumnUnit": "",
            "actionValue": {},
            "deviceActionType": "",
            "deviceId": "",
            "deviceName": "",
            "deviceType": "",
            "deviceUuid": "",
            "groupDevices": [
            {
                "deviceId": "",
                "deviceName": "",
                "status": 0
            }
            ],
            "groupId": "",
            "groupName": "",
            "groupType": "",
            "imgUrl": "",
            "productId": "",
            "productName": "",
            "shortGroupId": 0
        },
        "instanceId": "",
        "isDelayed": true,
        "isExpire": true,
        "messageAction": {
        "messageType": ""
        },
        "refId": "",
        "refType": "",
        "sort": 0,
        "status": 0,
        "streamRuleAction": {
            "instanceIcon": "",
            "instanceId": "",
            "instanceName": "",
            "ruleActionType": "",
            "status": 0
        },
        "updateTime": ""
        }
        ],
        "activeTime": "",
        "address": "",
        "appId": "",
        "appName": "",
        "assetId": "",
        "conditions": [
        {
            "conditionData": "",
            "conditionId": "",
            "conditionType": "",
            "createTime": "",
            "deviceCondition": {
                "conditionColumn": "",
                "conditionColumnType": "",
                "conditionColumnUnit": "",
                "conditionExpress": "",
                "conditionValue": {},
                "deviceId": "",
                "deviceName": "",
                "deviceType": "",
                "deviceUuid": "",
                "imgUrl": "",
                "productId": "",
                "productName": ""
            },
            "instanceId": "",
            "isExpire": true,
            "jobCondition": {
                "loops": "",
                "startDate": "",
                "time": ""
            },
            "refId": "",
            "refType": "",
            "sort": 0,
            "updateTime": "",
            "weatherCondition": {
                "weatherCode": "",
                "weatherExpress": "",
                "weatherValue": ""
            }
        }
        ],
        "createTime": "",
        "curCityLat": "",
        "curCityLon": "",
        "curCityName": "",
        "curCountryName": "",
        "dataCenter": "",
        "deleted": 0,
        "executeType": "",
        "expireStatus": 0,
        "instanceDesc": "",
        "instanceIcon": "",
        "instanceId": "",
        "instanceName": "",
        "instanceType": "",
        "isHome": true,
        "isLocal": true,
        "isRefreshWeather": true,
        "isShow": true,
        "jobId": "",
        "lastActionTime": "",
        "lat": "",
        "localGatewayUuid": "",
        "lon": "",
        "repeatLoops": "",
        "shortId": 0,
        "sort": 0,
        "status": 0,
        "timeZone": "",
        "timeZoneName": "",
        "updateTime": "",
        "userId": ""
    }
    ],
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

## Query rule details based on ID

**Interface address**:`/v1.0/stream/rule/user/instance/getByInstanceId`

**Request method**: `POST`

**Request data type**: `application/json`

**Response data type**: `*/*`

**Interface Description**:

**Request Parameters**:

| Parameter name | Parameter description | Request type | Required | Data type | Schema |
| ---------- | ---------- | -------- | -------- | -------- | ------ |
| instanceId | instanceId | query | true | string | |

**Response Status**:

| Status code | Description | Schema |
| ------ | ---------- | ------- |
| 200 | OK | ResponseResult object «StreamRuleUserInstanceVO object» |
| 201 | Created | |
| 401 | Unauthorized | |
| 403 | Forbidden | |
| 404 | Not Found | |

**Response parameters**:

| Parameter name | Parameter description | Type | schema |
| ------------------- | ------------ | --------- |------- |
| code | Response status code | integer(int32) | integer(int32) |
| data | response data | StreamRuleUserInstanceVO object | StreamRuleUserInstanceVO object |
| actions | list of actions to be executed | array | StreamRuleUserActionApiApiVO object |
| actionData | action data | string | |
| actionId | primary key id | string | |
| actionType | Action type (device: device action, stream_rule: rule engine, message_center: message center, delayed_action: delayed execution) | string | |
| createTime | creation time | string | |
| delayedAction | DelayedAction | DelayedAction |
| time | Delay time (for example: 05:59:59, cannot exceed 6 hours) | string | |
| delayedTime | Delay time (e.g. 05:59:59, cannot exceed 6 hours) | string | |
| deviceAction | DeviceAction | DeviceAction |
| actionColumn | action field | string | |
| actionColumnType | action type (dp point type) | string | |
| actionColumnUnit | field unit (dp point unit) | string | |
| actionValue | action value (when the value is equal to toggle, it is control inversion, only supports bool type) | object | |
| deviceActionType | Device action type (device: device, group: group) | string | |
| deviceId | device id (must be passed if deviceActionType=device) | string | |
| deviceName | Device name (background field) | string | |
| deviceType | Device type (rino:rino,ty:ty (default:rino)) | string | |
| deviceUuid | device uuid (used in the background) | string | |
| groupDevices | group id (required if deviceActionType=group) | array | GroupDevice |
| deviceId | device ID | string | |
| deviceName | device name | string | |
| status | status | integer | |
| groupId | Group id (required if deviceActionType=group) | string | |
| groupName | Group name (backend field) | string | |
| groupType | Group type (rino:rino,ty:ty (default:rino)) | string | |
| imgUrl | icon (backend field) | string | |
| productId | product id (backend field) | string | |
| productName | Product name (backend field) | string | |
| shortGroupId | Group short id (required if deviceActionType=group) | integer | |
| instanceId | instance id | string | |
| isDelayed | Whether to delay execution | boolean | |
| isExpire | Is it expired? | boolean | |
| messageAction | Message execution data | MessageAction | MessageAction |
| messageType | Message type (message_center: message center, app_push: APP push) | string | |
| refId | associated id (device id) | string | |
| refType | Association type (stream_rule: rule engine, device: device, group: group) | string | |
| sort | sort | integer | |
| status | Synchronization status: 0: no response, 1: success; 2: failure | integer |                                   |
| streamRuleAction | Rule Engine Execution Data | StreamRuleAction | StreamRuleAction |
| instanceIcon | instance icon | string | |
| instanceId | instance id | string | |
| instanceName | instance name | string | |
| ruleActionType | Rule engine execution type (action: execute action, update_status: modify status) | string | |
| status | Status (1: enabled, 0: disabled. ruleActionType=update_status is required) | integer | |
| updateTime | update time | string | |
| activeTime | Effective time (all day: all, daytime: daytime, night: night, custom: 12:00-06:00) | string | |
| address | address | string | |
| appId | application id | string | |
| appName | Application name | string | |
| assetId | asset id | string | |
| conditions | instance conditions | array | StreamRuleUserConditionApiVO object |
| conditionData | condition data (device | job | weather) |
| conditionId | primary key id | string | |
| conditionType | Condition type (device-device condition, job-timing condition, weather-weather condition, onekey-one-key execution) | string | |
| createTime | creation time | string | |
| deviceCondition | DeviceCondition | DeviceCondition |
| conditionColumn | condition field | string | |
| conditionColumnType | field type (dp point type) | string | |
| conditionColumnUnit | field unit (dp point unit) | string | |
| conditionExpress | Conditional expression ((greater than: >, equal to: ==, less than: <)) | string | |
| conditionValue | condition value | object | |
| deviceId | device id | string | |
| deviceName | Device name (background field) | string | |
| deviceType | Device type (rino:rino,ty:ty (default:rino)) | string | |
| deviceUuid | device uuid (used in the background) | string | |
| imgUrl | Device icon (background field) | string | |
| productId | product id (backend field) | string | |
| productName | Product name (backend field) | string | |
| instanceId | instance id | string | |
| isExpire | Is it expired? | boolean | |
| jobCondition | Timing Condition | JobCondition | JobCondition |
| loops | Cycle, 0000000 7 digits starting from Monday, a value of 1 means that execution is allowed on that day; if all are 0, it will be executed only once by default | string | |
| startDate | Start date (yyyy-mm-dd) | string | |
| time | Time, 24-hour formatted time: e.g. 18:00 | string | |
| refId | associated id (device id) | string | |
| refType | associated type (device: device) | string | |
| sort | sort | integer | |
| updateTime | update time | string | |
| weatherCondition | WeatherCondition |
| weatherCode | Weather code (temperature-temperature, humidity-humidity, climate-weather, pm25-PM2.5, aqi-air quality, sun-sunrise and sunset, wind_speed-wind speed) | string | |
| weatherExpress | Weather expression (greater than: >, equal to: ==, less than: <) | string | |
| weatherValue | Temperature (range: -40 to 40, unit: ℃)\nHumidity (dry, comfort, wet)\nWeather (sunny, cloudy, rainy, snowy, mist)\nPM25 (good, fine, polluted)\nAir quality (good, fine, polluted)\nSunrise and sunset (sunrise, sunset)\nWind speed (range: 0-62, unit: m/s) | string | |
| createTime | creation time | string | |
| curCityLat | Current city latitude | string | |
| curCityLon | Current city longitude | string | |
| curCityName | Current city name | string | |
| curCountryName | Current country name | string | |
| dataCenter | data center code | string | |
| deleted | Delete flag (0: normal, 1 deleted) | integer | |
| executeType | Execution type (all conditions are met: &&, any condition is met: | | ) |
| expireStatus | Expiration status: 0: expired; 1: valid; 2: partially valid | integer | |
| instanceDesc | Instance description | string | |
| instanceIcon | instance icon | string | |
| instanceId | primary key id | string | |
| instanceName | instance name | string | |
| instanceType | Instance type (stream-rule engine, onekey-one-key execution, job-scheduled task, redis-redis task) | string | |
| isHome | Whether to display on the home page | boolean | |
| isLocal | Is it a local scene? | boolean | |
| isRefreshWeather | Whether to refresh the weather | boolean | |
| isShow | Whether to display. The default value is false when the local scene is created. The app should not see the scene that is not displayed. | boolean | |
| jobId | scheduled task id | string | |
| lastActionTime | Last execution time | string | |
| lat | latitude | string | |
| localGatewayUuid | local scene gateway uuid | string | |
| lon | longitude | string | |
| repeatLoops | Repeat cycle (0000000, 7 digits starting from Monday, a value of 1 means execution is allowed on that day) | string | |
| shortId | short id (used by gateway) | integer | |
| sort | sort | integer | |
| status | status (0-disable, 1-enable) | integer | |
| timeZone | time zone | string | |
| timeZoneName | time zone name | string | |
| updateTime | update time | string | |
| userId | userid | string | |
| message | response message | string | |
| reqId | request ID | string | |
| time | request timestamp (10 bits) | integer(int64) | integer(int64) |

**Sample response**:

```JSON
{
    "code": 200,
    "data": {
    "actions": [
    {
        "actionData": "",
        "actionId": "",
        "actionType": "",
        "createTime": "",
        "delayedAction": {
            "time": ""
        },
        "delayedTime": "",
        "deviceAction": {
            "actionColumn": "",
            "actionColumnType": "",
            "actionColumnUnit": "",
            "actionValue": {},
            "deviceActionType": "",
            "deviceId": "",
            "deviceName": "",
            "deviceType": "",
            "deviceUuid": "",
            "groupDevices": [
            {
                "deviceId": "",
                "deviceName": "",
                "status": 0
            }
            ],
            "groupId": "",
            "groupName": "",
            "groupType": "",
            "imgUrl": "",
            "productId": "",
            "productName": "",
            "shortGroupId": 0
        },
        "instanceId": "",
        "isDelayed": true,
        "isExpire": true,
        "messageAction": {
            "messageType": ""
        },
        "refId": "",
        "refType": "",
        "sort": 0,
        "status": 0,
        "streamRuleAction": {
            "instanceIcon": "",
            "instanceId": "",
            "instanceName": "",
            "ruleActionType": "",
            "status": 0
        },
        "updateTime": ""
        }
        ],
        "activeTime": "",
        "address": "",
        "appId": "",
        "appName": "",
        "assetId": "",
        "conditions": [
        {
            "conditionData": "",
            "conditionId": "",
            "conditionType": "",
            "createTime": "",
            "deviceCondition": {
                "conditionColumn": "",
                "conditionColumnType": "",
                "conditionColumnUnit": "",
                "conditionExpress": "",
                "conditionValue": {},
                "deviceId": "",
                "deviceName": "",
                "deviceType": "",
                "deviceUuid": "",
                "imgUrl": "",
                "productId": "",
                "productName": ""
            },
            "instanceId": "",
            "isExpire": true,
            "jobCondition": {
                "loops": "",
                "startDate": "",
                "time": ""
            },
            "refId": "",
            "refType": "",
            "sort": 0,
            "updateTime": "",
            "weatherCondition": {
                "weatherCode": "",
                "weatherExpress": "",
                "weatherValue": ""
            }
        }
        ],
        "createTime": "",
        "curCityLat": "",
        "curCityLon": "",
        "curCityName": "",
        "curCountryName": "",
        "dataCenter": "",
        "deleted": 0,
        "executeType": "",
        "expireStatus": 0,
        "instanceDesc": "",
        "instanceIcon": "",
        "instanceId": "",
        "instanceName": "",
        "instanceType": "",
        "isHome": true,
        "isLocal": true,
        "isRefreshWeather": true,
        "isShow": true,
        "jobId": "",
        "lastActionTime": "",
        "lat": "",
        "localGatewayUuid": "",
        "lon": "",
        "repeatLoops": "",
        "shortId": 0,
        "sort": 0,
        "status": 0,
        "timeZone": "",
        "timeZoneName": "",
        "updateTime": "",
        "userId": ""
    },
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

## Add new rule data

**Interface address**:`/v1.0/stream/rule/user/instance/add`

**Request method**: `POST`

**Request data type**: `application/json`

**Response data type**: `*/*`

**Interface Description**:

**Request Example**:

```JSON
{
    "actionExecuteType": "",
    "actions": [
    {
        "actionData": "",
        "actionType": "",
        "delayedAction": {
            "time": ""
        },
        "deviceAction": {
            "actionColumn": "",
            "actionColumnType": "",
            "actionColumnUnit": "",
            "actionValue": {},
            "deviceActionType": "",
            "deviceId": "",
            "deviceName": "",
            "deviceType": "",
            "deviceUuid": "",
            "groupDevices": [
            {
                "deviceId": "",
                "deviceName": "",
                "status": 0
            }
            ],
            "groupId": "",
            "groupName": "",
            "groupType": "",
            "imgUrl": "",
            "productId": "",
            "productName": "",
            "shortGroupId": 0
        },
        "instanceId": "",
        "isExpire": true,
        "messageAction": {
            "messageType": ""
        },
        "refDeviceType": "",
        "refId": "",
        "refType": "",
        "sort": 0,
        "status": 0,
        "streamRuleAction": {
            "instanceIcon": "",
            "instanceId": "",
            "instanceName": "",
            "ruleActionType": "",
            "status": 0
        }
    }
    ],
    "activeTime": "",
    "address": "",
    "assetId": "",
    "autoSendLocal": true,
    "conditions": [
    {
        "conditionData": "",
        "conditionType": "",
        "deviceCondition": {
            "conditionColumn": "",
            "conditionColumnType": "",
            "conditionColumnUnit": "",
            "conditionExpress": "",
            "conditionValue": {},
            "deviceId": "",
            "deviceName": "",
            "deviceType": "",
            "deviceUuid": "",
            "imgUrl": "",
            "productId": "",
            "productName": ""
        },
        "instanceId": "",
        "isExpire": true,
        "jobCondition": {
            "loops": "",
            "startDate": "",
            "time": ""
        },
        "refDeviceType": "",
        "refId": "",
        "refType": "",
        "sort": 0,
        "weatherCondition": {
            "weatherCode": "",
            "weatherExpress": "",
            "weatherValue": ""
        }
    }
  ],
  "curCountryName": "",
  "executeType": "",
  "instanceDesc": "",
  "instanceIcon": "",
  "instanceName": "",
  "isHidden": true,
  "isHome": true,
  "lat": "",
  "lon": "",
  "repeatLoops": "",
  "sort": 0,
  "timeZone": ""
}
```

**Request Parameters**:

| Parameter name | Parameter description | Request type | Required | Data type | Schema |
| ------------- | ------------------ | -------- | -------- | ------------- | --------- |
| dto | dto | body | true | StreamRuleUserInstanceDTO object | StreamRuleUserInstanceDTO object |
| actionExecuteType | Action execution type (serial: serial, parallel: parallel. Default is serial) | | false | string | |
| actions | action list | | false | array | StreamRuleUserActionDTO object |
| actionData | Action data (backend field) | | false | string | |
| actionType | Action type (device: device action, stream_rule: rule engine, message_center: message center, delayed_action: delayed execution) | | false | string | |
| delayedAction | Global delayed execution data | | false | DelayedAction | DelayedAction |
| time | Delay time (for example: 05:59:59, cannot exceed 6 hours) | | false | string | |
| deviceAction | Device action | | false | DeviceAction | DeviceAction |
| actionColumn | action field | | false | string | |
| actionColumnType | action type (dp point type) | | false | string | |
| actionColumnUnit | Field unit (dp point unit) | | false | string | |
| actionValue | action value (when the value is equal to toggle, it is control inversion, only supports bool type) | | false | object | |
| deviceActionType | Device action type (device: device, group: group) | | false | string | |
| deviceId | Device id (must be passed if deviceActionType=device) | | false | string | |
| deviceName | Device name (background field) | | false | string | |
| deviceType | Device type (rino:rino,ty:ty (default:rino)) | | false | string | |
| deviceUuid | Device UUID (used in the background) | | false | string | |
| groupDevices | Group id (required if deviceActionType=group) | | false | array | GroupDevice |
| deviceId | device ID | | false | string | |
| deviceName | device name | | false | string | |
| status | status | | false | integer | |
| groupId | Group id (must be passed if deviceActionType=group) | | false | string | |
| groupName | Group name (backend field) | | false | string | |
| groupType | Group type (rino:rino,ty:ty (default:rino)) | | false | string | |
| imgUrl | Icon (backend field) | | false | string | |
| productId | product id (backend field) | | false | string | |
| productName | Product name (backend field) | | false | string | |
| shortGroupId | Group short id (must be passed if deviceActionType=group) | | false | integer | |
| instanceId | instance id | | false | string | |
| isExpire | Is it expired? | | false | boolean | |
| messageAction | Message execution data | | false | MessageAction | MessageAction |
| messageType | Message type (message_center: message center, app_push: APP push) | | false | string | |
| refDeviceType | Associated device type (when refType=device) | | false | string | |
| refId | association id (backend field, rule engine id | device id | group id) | | false |
| refType | Association type (backend field, stream_rule: rule engine, device: device, group: group) | | false | string | |
| sort | sort | | false | integer | |
| status | Synchronization status: 0: no response, 1: success; 2: failure | | false | integer | |
| streamRuleAction | Rule engine execution data | | false | StreamRuleAction | StreamRuleAction |
| instanceIcon | instance icon | | false | string | |
| instanceId | instance id | | false | string | |
| instanceName | instance name | | false | string | |
| ruleActionType | Rule engine execution type (action: execute action, update_status: modify status) | | false | string | |
| status | Status (1: enabled, 0: disabled. ruleActionType=update_status is required) | | false | integer | |
| activeTime | Effective time (all day: all, daytime: daytime, night: night, custom: 12:00-06:00) | | false | string | |
| address | address| | false | string | |
| appId | application id | | false | string | |
| assetId | asset id | | false | string | |
| autoSendLocal | Whether to automatically send local data | | false | boolean | |
| conditions | list of conditions | | false | array | StreamRuleUserConditionDTO object |
| conditionData | Condition data (background field, device | job | weather) | | false |
| conditionType | Condition type (device-device condition, job-timing condition, weather-weather condition, onekey-one-key execution) | | false | string | |
| deviceCondition | Device Condition | | false | DeviceCondition | DeviceCondition |
| conditionColumn | condition field | | false | string | |
| conditionColumnType | field type (dp point type) | | false | string | |
| conditionColumnUnit | Field unit (dp point unit) | | false | string | |
| conditionExpress | Conditional expression ((greater than: >, equal to: ==, less than: <)) | | false | string | |
| conditionValue | condition value | | false | object | |
| deviceId | device id | | false | string | |
| deviceName | Device name (background field) | | false | string | |
| deviceType | Device type (rino:rino,ty:ty (default:rino)) | | false | string | |
| deviceUuid | Device UUID (used in the background) | | false | string | |
| imgUrl | Device icon (background field) | | false | string | |
| productId | product id (backend field) | | false | string | |
| productName | Product name (backend field) | | false | string | |
| instanceId | instance id | | false | string | |
| isExpire | Is it expired? | | false | boolean | |
| jobCondition | Timing condition | | false | JobCondition | JobCondition |
| loops | Cycle, 0000000 7 digits starting from Monday, a value of 1 means that execution is allowed on that day; if all are 0, it will be executed only once by default | | false | string | |
| startDate | Start date (yyyy-mm-dd) | | false | string | |
| time | Time, 24-hour formatted time: e.g. 18:00 | | false | string | |
| refDeviceType | Associated device type (when refType=device) | | false | string | |
| refId | Related id (backend field, device id) | | false | string | |
| refType | associated type (device: device) | | false | string | |
| sort | sort | | false | integer | |
| weatherCondition | weather conditions | | false | WeatherCondition | WeatherCondition |
| weatherCode | Weather code (temperature-temperature, humidity-humidity, climate-weather, pm25-PM2.5, aqi-air quality, sun-sunrise and sunset, wind_speed-wind speed) | | false | string | |
| weatherExpress | Weather expression (greater than: >, equal to: ==, less than: <) | | false | string | |
| weatherValue | Temperature (range: -40 to 40, unit: ℃)\nHumidity (dry, comfort, wet)\nWeather (sunny, cloudy, rainy, snowy, mist)\nPM25 (good, fine, polluted)\nAir quality (good, fine, polluted)\nSunrise and sunset (sunrise, sunset)\nWind speed (range: 0-62, unit: m/s) | | false | string | |
| curCityLat | Current city latitude | | false | string | |
| curCityLon | Current city longitude | | false | string | |
| curCityName | Current city name | | false | string | |
| curCountryName | Current country name | | false | string | |
| executeType | Execution type (all conditions are met: &&, any condition is met: | | ) | | false |
| instanceDesc | Instance description | | false | string | |
| instanceIcon | instance icon | | false | string | |
| instanceId | primary key id | | false | string | |
| instanceName | instance name | | false | string | |
| instanceType | Instance type (stream-rule engine, onekey-one-key execution, job-scheduled task, redis-redis task) | | false | string | |
| isHidden | Whether to hide, hide after unbinding the local scene, and show it after rebinding | | false | boolean | |
| isHome | Whether to display on the home page | | false | boolean | |
| isLocal | Is it a local scene? | | false | boolean | |
| isRefreshWeather | Whether to refresh the weather | | false | boolean | |
| lastActionTime | Last execution time | | false | string | |
| lat | latitude | | false | string | |
| localGatewayUuid | local scene gateway uuid | | false | string | |
| lon | longitude | | false | string | |
| repeatLoops | Repeat cycle (0000000, 7 digits starting from Monday, a value of 1 means execution is allowed on that day) | | false | string | |
| shortId | short id (used by gateway) | | false | integer | |
| sort | sort | | false | integer | |
| timeZone | Time zone, default is user's current time zone | | false | string | |
| userId | userid | | false | string | |

**Response Status**:

| Status code | Description | Schema |
| ------ | ---------- | -------------------- |
| 200 | OK | ResponseResult object «StreamRuleUserOperatApiVO» |
| 201 | Created | |
| 401 | Unauthorized | |
| 403 | Forbidden | |
| 404 | Not Found | |

**Response parameters**:

| Parameter name | Parameter description | Type | schema |
| ------------ | ---------------- | ---------- | ----------- |
| code | Response status code | integer(int32) | integer(int32) |
| data | Response data | StreamRuleUserOperatApiVO | StreamRuleUserOperatApiVO |
| instanceIcon | instance icon | string | |
| instanceId | primary key id | string | |
| instanceName | instance name | string | |
| result | Operation result | boolean | |
| message | response message | string | |
| reqId | request ID | string | |
| time | request timestamp (10 bits) | integer(int64) | integer(int64) |

**Sample response**:

```JSON
{
    "code": 200,
    "data": {
        "instanceIcon": "",
        "instanceId": "",
        "instanceName": "",
        "result": true
    },
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

## Update rule data

**Interface address**:`/v1.0/stream/rule/user/instance/updateByInstanceId`

**Request method**: `POST`

**Request data type**: `application/json`

**Response data type**: `*/*`

**Interface Description**:

**Request Example**:

```JSON
{
    "actionExecuteType": "",
    "actions": [
    {
        "actionData": "",
        "actionType": "",
        "delayedAction": {
            "time": ""
        },
        "deviceAction": {
            "actionColumn": "",
            "actionColumnType": "",
            "actionColumnUnit": "",
            "actionValue": {},
            "deviceActionType": "",
            "deviceId": "",
            "deviceName": "",
            "deviceType": "",
            "deviceUuid": "",
            "groupDevices": [
            {
                "deviceId": "",
                "deviceName": "",
                "status": 0
            }
            ],
            "groupId": "",
            "groupName": "",
            "groupType": "",
            "imgUrl": "",
            "productId": "",
            "productName": "",
            "shortGroupId": 0
        },
        "instanceId": "",
        "isExpire": true,
        "messageAction": {
            "messageType": ""
        },
        "refDeviceType": "",
        "refId": "",
        "refType": "",
        "sort": 0,
        "status": 0,
        "streamRuleAction": {
            "instanceIcon": "",
            "instanceId": "",
            "instanceName": "",
            "ruleActionType": "",
            "status": 0
        }
    }
    ],
    "activeTime": "",
    "address": "",
    "assetId": "",
    "autoSendLocal": true,
    "conditions": [
    {
        "conditionData": "",
        "conditionType": "",
        "deviceCondition": {
            "conditionColumn": "",
            "conditionColumnType": "",
            "conditionColumnUnit": "",
            "conditionExpress": "",
            "conditionValue": {},
            "deviceId": "",
            "deviceName": "",
            "deviceType": "",
            "deviceUuid": "",
            "imgUrl": "",
            "productId": "",
            "productName": ""
        },
        "instanceId": "",
        "isExpire": true,
        "jobCondition": {
            "loops": "",
            "startDate": "",
            "time": ""
        },
        "refDeviceType": "",
        "refId": "",
        "refType": "",
        "sort": 0,
        "weatherCondition": {
            "weatherCode": "",
            "weatherExpress": "",
            "weatherValue": ""
        }
    }
  ],
  "curCountryName": "",
  "executeType": "",
  "instanceDesc": "",
  "instanceIcon": "",
  "instanceName": "",
  "isHidden": true,
  "isHome": true,
  "lat": "",
  "lon": "",
  "repeatLoops": "",
  "sort": 0,
  "timeZone": ""
}
```

**Request Parameters**:

| Parameter name | Parameter description | Request type | Required | Data type | Schema |
| -------------- | ------------------- | -------- | -------- | -------------- | -------------- |
| dto | dto | body | true | StreamRuleUserInstanceDTO object | StreamRuleUserInstanceDTO object |
| actionExecuteType | Action execution type (serial: serial, parallel: parallel. Default is serial) | | false | string | |
| actions | action list | | false | array | StreamRuleUserActionDTO object |
| actionData | Action data (backend field) | | false | string | |
| actionType | Action type (device: device action, stream_rule: rule engine, message_center: message center, delayed_action: delayed execution) | | false | string | |
| delayedAction | Global delayed execution data | | false | DelayedAction | DelayedAction |
| time | Delay time (for example: 05:59:59, cannot exceed 6 hours) | | false | string | |
| deviceAction | Device action | | false | DeviceAction | DeviceAction |
| actionColumn | action field | | false | string | |
| actionColumnType | action type (dp point type) | | false | string | |
| actionColumnUnit | Field unit (dp point unit) | | false | string | |
| actionValue | action value (when the value is equal to toggle, it is control inversion, only supports bool type) | | false | object | |
| deviceActionType | Device action type (device: device, group: group) | | false | string | |
| deviceId | Device id (must be passed if deviceActionType=device) | | false | string | |
| deviceName | Device name (background field) | | false | string | |
| deviceType | Device type (rino:rino,ty:ty (default:rino)) | | false | string | |
| deviceUuid | Device UUID (used in the background) | | false | string | |
| groupDevices | Group id (required if deviceActionType=group) | | false | array | GroupDevice |
| deviceId | device ID | | false | string | |
| deviceName | device name | | false | string | |
| status | status | | false | integer | |
| groupId | Group id (must be passed if deviceActionType=group) | | false | string | |
| groupName | Group name (backend field) | | false | string | |
| groupType | Group type (rino:rino,ty:ty (default:rino)) | | false | string | |
| imgUrl | Icon (backend field) | | false | string | |
| productId | product id (backend field) | | false | string | |
| productName | Product name (backend field) | | false | string | |
| shortGroupId | Group short id (must be passed if deviceActionType=group) | | false | integer | |
| instanceId | instance id | | false | string | |
| isExpire | Is it expired? | | false | boolean | |
| messageAction | Message execution data | | false | MessageAction | MessageAction |
| messageType | Message type (message_center: message center, app_push: APP push) | | false | string | |
| refDeviceType | Associated device type (when refType=device) | | false | string | |
| refId | association id (backend field, rule engine id | device id | group id) | | false |
| refType | Association type (backend field, stream_rule: rule engine, device: device, group: group) | | false | string | |
| sort | sort | | false | integer | |
| status | Synchronization status: 0: no response, 1: success; 2: failure | | false | integer | |
| streamRuleAction | Rule engine execution data | | false | StreamRuleAction | StreamRuleAction |
| instanceIcon | instance icon | | false | string | |
| instanceId | instance id | | false | string | |
| instanceName | instance name | | false | string | |
| ruleActionType | Rule engine execution type (action: execute action, update_status: modify status) | | false | string | |
| status | Status (1: enabled, 0: disabled. ruleActionType=update_status is required) | | false | integer | |
| activeTime | Effective time (all day: all, daytime: daytime, night: night, custom: 12:00-06:00) | | false | string | |
| address | address| | false | string | |
| appId | application id | | false | string | |
| assetId | asset id | | false | string | |
| autoSendLocal | Whether to automatically send local data | | false | boolean | |
| conditions | list of conditions | | false | array | StreamRuleUserConditionDTO object |
| conditionData | Condition data (background field, device | job | weather) | | false |
| conditionType | Condition type (device-device condition, job-timing condition, weather-weather condition, onekey-one-key execution) | | false | string | |
| deviceCondition | Device Condition | | false | DeviceCondition | DeviceCondition |
| conditionColumn | condition field | | false | string | |
| conditionColumnType | field type (dp point type) | | false | string | |
| conditionColumnUnit | Field unit (dp point unit) | | false | string | |
| conditionExpress | Conditional expression ((greater than: >, equal to: ==, less than: <)) | | false | string | |
| conditionValue | condition value | | false | object | |
| deviceId | device id | | false | string | |
| deviceName | Device name (background field) | | false | string | |
| deviceType | Device type (rino:rino,ty:ty (default:rino)) | | false | string | |
| deviceUuid | Device UUID (used in the background) | | false | string | |
| imgUrl | Device icon (background field) | | false | string | |
| productId | product id (backend field) | | false | string | |
| productName | Product name (backend field) | | false | string | |
| instanceId | instance id | | false | string | |
| isExpire | Is it expired? | | false | boolean | |
| jobCondition | Timing condition | | false | JobCondition | JobCondition |
| loops | Cycle, 0000000 7 digits starting from Monday, a value of 1 means that execution is allowed on that day; if all are 0, it will be executed only once by default | | false | string | |
| startDate | Start date (yyyy-mm-dd) | | false | string | |
| time | Time, 24-hour formatted time: e.g. 18:00 | | false | string | |
| refDeviceType | Associated device type (when refType=device) | | false | string | |
| refId | Related id (backend field, device id) | | false | string | |
| refType | associated type (device: device) | | false | string | |
| sort | sort | | false | integer | |
| weatherCondition | weather conditions | | false | WeatherCondition | WeatherCondition |
| weatherCode | Weather code (temperature-temperature, humidity-humidity, climate-weather, pm25-PM2.5, aqi-air quality, sun-sunrise and sunset, wind_speed-wind speed) | | false | string | |
| weatherExpress | Weather expression (greater than: >, equal to: ==, less than: <) | | false | string | |
| weatherValue | Temperature (range: -40 to 40, unit: ℃)\nHumidity (dry, comfort, wet)\nWeather (sunny, cloudy, rainy, snowy, mist)\nPM25 (good, fine, polluted)\nAir quality (good, fine, polluted)\nSunrise and sunset (sunrise, sunset)\nWind speed (range: 0-62, unit: m/s) | | false | string | |
| curCityLat | Current city latitude | | false | string | |
| curCityLon | Current city longitude | | false | string | |
| curCityName | Current city name | | false | string | |
| curCountryName | Current country name | | false | string | |
| executeType | Execution type (all conditions are met: &&, any condition is met: | | ) | | false |
| instanceDesc | Instance description | | false | string | |
| instanceIcon | instance icon | | false | string | |
| instanceId | primary key id | | false | string | |
| instanceName | instance name | | false | string | |
| instanceType | Instance type (stream-rule engine, onekey-one-key execution, job-scheduled task, redis-redis task) | | false | string | |
| isHidden | Whether to hide, hide after unbinding the local scene, and show it after rebinding | | false | boolean | |
| isHome | Whether to display on the home page | | false | boolean | |
| isLocal | Is it a local scene? | | false | boolean | |
| isRefreshWeather | Whether to refresh the weather | | false | boolean | |
| lastActionTime | Last execution time | | false | string | |
| lat | latitude | | false | string | |
| localGatewayUuid | local scene gateway uuid | | false | string | |
| lon | longitude | | false | string | |
| repeatLoops | Repeat cycle (0000000, 7 digits starting from Monday, a value of 1 means execution is allowed on that day) | | false | string | |
| shortId | short id (used by gateway) | | false | integer | |
| sort | sort | | false | integer | |
| timeZone | Time zone, default is user's current time zone | | false | string | |
| userId | userid | | false | string | |

**Response Status**:

| Status code | Description | Schema |
| ------ | ---------- | ------------------- |
| 200 | OK | ResponseResult object «StreamRuleUserOperatApiVO» |
| 201 | Created | |
| 401 | Unauthorized | |
| 403 | Forbidden | |
| 404 | Not Found | |

**Response parameters**:

| Parameter name | Parameter description | Type | schema |
| ------------ | ---------------- | --------- | --------------- |
| code | Response status code | integer(int32) | integer(int32) |
| data | Response data | StreamRuleUserOperatApiVO | StreamRuleUserOperatApiVO |
| instanceIcon | instance icon | string | |
| instanceId | primary key id | string | |
| instanceName | instance name | string | |
| result | Operation result | boolean | |
| message | response message | string | |
| reqId | request ID | string | |
| time | request timestamp (10 bits) | integer(int64) | integer(int64) |

**Sample response**:

```JSON
{
    "code": 200,
    "data": {
        "instanceIcon": "",
        "instanceId": "",
        "instanceName": "",
        "result": true
    },
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

## Execute instance (one-click execution only)

**Interface address**:`/v1.0/stream/rule/user/instance/executeInstance`

**Request method**: `POST`

**Request data type**: `application/json`

**Response data type**: `*/*`

**Interface Description**:

**Request Parameters**:

| Parameter name | Parameter description | Request type | Required | Data type | Schema |
| ---------- | -------- | -------- | -------- | -------- | ------ |
| instanceId | instance id | query | false | string | |

**Response Status**:

| Status code | Description | Schema |
| ------ | ---------- | ---------------------------- |
| 200 | OK | ResponseResult object «boolean» |
| 201 | Created | |
| 401 | Unauthorized | |
| 403 | Forbidden | |
| 404 | Not Found | |

**Response parameters**:

| Parameter name | Parameter description | Type | schema |
| -------- | ---------------- | ------------- | -------- ------ |
| code | Response status code | integer(int32) | integer(int32) |
| data | response data | boolean | |
| message | response message | string | |
| reqId | request ID | string | |
| time | request timestamp (10 bits) | integer(int64) | integer(int64) |

**Sample response**:

```JSON
{
    "code": 200,
    "data": true,
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

## Delete rule data

**Interface address**:`/v1.0/stream/rule/user/instance/removeByInstanceId`

**Request method**: `POST`

**Request data type**: `application/json`

**Response data type**: `*/*`

**Interface Description**:

**Request Parameters**:

| Parameter name | Parameter description | Request type | Required | Data type | Schema |
| ---------- | -------- | -------- | -------- | -------- | ------ |
| instanceId | instance id | query | false | string | |

**Response Status**:

| Status code | Description | Schema |
| ------ | ---------- | ------------ |
| 200 | OK | ResponseResult object «boolean» |
| 201 | Created | |
| 401 | Unauthorized | |
| 403 | Forbidden | |
| 404 | Not Found | |

**Response parameters**:

| Parameter name | Parameter description | Type | schema |
| -------- | ---------- | ------------- | -------- ------ |
| code | Response status code | integer(int32) | integer(int32) |
| data | response data | boolean | |
| message | response message | string | |
| reqId | request ID | string | |
| time | request timestamp (10 bits) | integer(int64) | integer(int64) |

**Sample response**:

```JSON
{
    "code": 200,
    "data": true,
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

## Add DP binding scene data

**Interface address**:`/v1.0/stream/rule/user/dp/add`

**Request method**: `POST`

**Request data type**: `application/json`

**Response data type**: `*/*`

**Interface Description**:

**Request Example**:

```JSON
{
    "actionData": "",
    "actionDataMap": {},
    "aliasName": "",
    "bgColor": "",
    "delayedTime": 0,
    "deviceId": "",
    "dpIdentifier": "",
    "dpValue": "",
    "gatewayUuid": "",
    "instanceId": "",
    "productId": "",
    "type": ""
}
```

**Request Parameters**:

| Parameter name | Parameter description | Request type | Required | Data type | Schema |
| ------------- | --------------------- | -------- | -------- | -------- | ----------- |
| dto | Scene dp point binding relationship table | body | true | StreamRuleUserDpDTO object | StreamRuleUserDpDTO object |
| actionData | execution data json | | false | string | |
| actionDataMap | Execution data: required when type=device or type=group | | false | object | |
| aliasName | alias name| | false | string | |
| bgColor | background color | | false | string | |
| delayedTime | Delay time (s) | | false | integer | |
| deviceId | device id | | false | string | |
| deviceUuid | device uuid | | false | string | |
| dpIdentifier | dp point | | false | string | |
| dpValue | dp point value | | false | string | |
| gatewayUuid | gateway uuid | | false | string | |
| instanceId | User automation id/group id/device id | | false | string | |
| productId | product id | | false | string | |
| recordId | primary key id | | false | string | |
| type | Type: stream_rule: rule instance; device: device; group: group | | false | string | |

**Response Status**:

| Status code | Description | Schema |
| ------ | ------------ | ---------------- |
| 200 | OK | ResponseResult object «string» |
| 201 | Created | |
| 401 | Unauthorized | |
| 403 | Forbidden | |
| 404 | Not Found | |

**Response parameters**:

| Parameter name | Parameter description | Type | schema |
| -------- | ---------------- | ------------- | ---------- |
| code | Response status code | integer(int32) | integer(int32) |
| data | response data | string | |
| message | response message | string | |
| reqId | request ID | string | |
| time | request timestamp (10 bits) | integer(int64) | integer(int64) |

**Sample response**:

```JSON
{
    "code": 200,
    "data": "",
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

## Update DP binding scene data

**Interface address**:`/v1.0/stream/rule/user/dp/updateByRecordId`

**Request method**: `POST`

**Request data type**: `application/json`

**Response data type**: `*/*`

**Interface Description**:

**Request Example**:

```JSON
{
    "actionData": "",
    "actionDataMap": {},
    "aliasName": "",
    "bgColor": "",
    "delayedTime": 0,
    "deviceId": "",
    "deviceUuid": "",
    "dpIdentifier": "",
    "dpValue": "",
    "gatewayUuid": "",
    "instanceId": "",
    "productId": "",
    "recordId": "",
    "type": ""
}
```

**Request Parameters**:

| Parameter name | Parameter description | Request type | Required | Data type | Schema |
| ------------- | --------------------- | -------- | -------- | --------- | ----------- |
| dto | Scene dp point binding relationship table | body | true | StreamRuleUserDpDTO object | StreamRuleUserDpDTO object |
| actionData | execution data json | | false | string | |
| actionDataMap | Execution data: required when type=device or type=group | | false | object | |
| aliasName | alias name| | false | string | |
| bgColor | background color | | false | string | |
| delayedTime | Delay time (s) | | false | integer | |
| deviceId | device id | | false | string | |
| deviceUuid | device uuid | | false | string | |
| dpIdentifier | dp point | | false | string | |
| dpValue | dp point value | | false | string | |
| gatewayUuid | gateway uuid | | false | string | |
| instanceId | User automation id/group id/device id | | false | string | |
| productId | product id | | false | string | |
| recordId | primary key id | | false | string | |
| type | Type: stream_rule: rule instance; device: device; group: group | | false | string | |

**Response Status**:

| Status code | Description | Schema |
| ------ | ---------- | ------- |
| 200 | OK | ResponseResult object «boolean» |
| 201 | Created | |
| 401 | Unauthorized | |
| 403 | Forbidden | |
| 404 | Not Found | |

**Response parameters**:

| Parameter name | Parameter description | Type | schema |
| -------- | ---------------- | ------------- | ------- |
| code | Response status code | integer(int32) | integer(int32) |
| data | response data | boolean | |
| message | response message | string | |
| reqId | request ID | string | |
| time | request timestamp (10 bits) | integer(int64) | integer(int64) |

**Sample response**:

```JSON
{
    "code": 200,
    "data": true,
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

## Delete DP binding scene data

**Interface address**:`/v1.0/stream/rule/user/dp/removeByRecordId`

**Request method**: `POST`

**Request data type**: `application/json`

**Response data type**: `*/*`

**Interface Description**:

**Request Parameters**:

| Parameter name | Parameter description | Request type | Required | Data type | Schema |
| -------- | -------- | -------- | -------- | -------- | ------ |
| recordId | recordId | query | true | string | |

**Response Status**:

| Status code | Description | Schema |
| ------ | ---------- | ------------ |
| 200 | OK | ResponseResult object «boolean» |
| 201 | Created | |
| 401 | Unauthorized | |
| 403 | Forbidden | |
| 404 | Not Found | |

**Response parameters**:

| Parameter name | Parameter description | Type | schema |
| -------- | ---------------- | ------------- | --------- |
| code | Response status code | integer(int32) | integer(int32) |
| data | response data | boolean | |
| message | response message | string | |
| reqId | request ID | string | |
| time | request timestamp (10 bits) | integer(int64) | integer(int64) |

**Sample response**:

```JSON
{
    "code": 200,
    "data": true,
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

## Query DP binding scene data based on primary key id

**Interface address**:`/v1.0/stream/rule/user/dp/getByRecordId`

**Request method**: `POST`

**Request data type**: `application/json`

**Response data type**: `*/*`

**Interface Description**:

**Request Parameters**:

| Parameter name | Parameter description | Request type | Required | Data type | Schema |
| -------- | -------- | -------- | -------- | -------- | ------ |
| recordId | recordId | query | true | string | |

**Response Status**:

| Status code | Description | Schema |
| ------ | ---------- | ------------- |
| 200 | OK | ResponseResult object «StreamRuleUserDpVO object» |
| 201 | Created | |
| 401 | Unauthorized | |
| 403 | Forbidden | |
| 404 | Not Found | |

**Response parameters**:

| Parameter name | Parameter description | Type | schema |
| ------------- | ------------------- | -------- | ---------- |
| code | Response status code | integer(int32) | integer(int32) |
| data | response data | StreamRuleUserDpVO object | StreamRuleUserDpVO object |
| actionData | execution data json | string | |
| actionDataMap | Execution data: required when type=device or type=group | object | |
| aliasName | alias name | string | |
| bgColor | background color | string | |
| createTime | creation time | string | |
| delayedTime | Delay time (s) | integer | |
| deviceId | device id | string | |
| deviceUuid | device uuid | string | |
| dpIdentifier | dp point | string | |
| dpValue | dp point value | string | |
| gatewayUuid | gateway uuid | string | |
| instanceIcon | instance icon | string | |
| instanceId | userautomationid | string | |
| instanceName | instance name | string | |
| isLocal | Whether local triggering is supported | boolean | |
| productId | product id | string | |
| recordId | primary key id | string | |
| type | Type: stream_rule: rule instance; device: device; group: group | string | |
| updateTime | update time | string | |
| message | response message | string | |
| reqId | request ID | string | |
| time | request timestamp (10 bits) | integer(int64) | integer(int64) |

**Sample response**:

```JSON
{
    "code": 200,
    "data": {
        "actionData": "",
        "actionDataMap": {},
        "aliasName": "",
        "bgColor": "",
        "createTime": "",
        "delayedTime": 0,
        "deviceId": "",
        "deviceUuid": "",
        "dpIdentifier": "",
        "dpValue": "",
        "gatewayUuid": "",
        "instanceIcon": "",
        "instanceId": "",
        "instanceName": "",
        "isLocal": true,
        "productId": "",
        "recordId": "",
        "type": "",
        "updateTime": ""
    },
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

## Create assets

**Interface address**:`/v1.0/appAsset/createByUserId`

**Request method**: `POST`

**Request data type**: `application/json`

**Response data type**: `*/*`

**Interface Description**:

**Request Example**:

```JSON
{
    "address": "",
    "appId": "",
    "lat": 0,
    "lng": 0,
    "name": "",
    "parentId": "",
    "userId": ""
}
```

**Request Parameters**:

| Parameter name | Parameter description | Request type | Required | Data type | Schema |
| ----------- | ------------------ | -------- | -------- | ---------- | ------------ |
| createAssetApiDTO | createAssetApiDTO | body | true | CreateAssetApiDTO | CreateAssetApiDTO |
| address | Detailed address | | false | string | |
| appId | AppId | | false | string | |
| lat | longitude| | false | number | |
| lng | longitude| | false | number | |
| name | name | | false | string | |
| parentId | parent ID | | false | string | |
| userId | userId | | false | string | |

**Response Status**:

| Status code | Description | Schema |
| ------ | ---------- | --------- |
| 200 | OK | ResponseResult object «AppAssetApiVo» |
| 201 | Created | |
| 401 | Unauthorized | |
| 403 | Forbidden | |
| 404 | Not Found | |

**Response parameters**:

| Parameter name | Parameter description | Type | schema |
| -------- | ---------------- | ------------- | ------------ |
| code | Response status code | integer(int32) | integer(int32) |
| data | Response data | AppAssetApiVo | AppAssetApiVo |
| address | Detailed address | string | |
| id | asset id | string | |
| lat | longitude | number | |
| lng | longitude | number | |
| name | name | string | |
| parentId | parent ID | string | |
| rootId | Root asset ID | string | |
| message | response message | string | |
| reqId | request ID | string | |
| time | request timestamp (10 bits) | integer(int64) | integer(int64) |

**Sample response**:

```JSON
{
    "code": 200,
    "data": {
        "address": "",
        "id": "",
        "lat": 0,
        "lng": 0,
        "name": "",
        "parentId": "",
        "rootId": ""
    },
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

## Update Assets

**Interface address**:`/v1.0/appAsset/updateByAssetId`

**Request method**: `POST`

**Request data type**: `application/json`

**Response data type**: `*/*`

**Interface Description**:

**Request Example**:

```JSON
{
    "address": "",
    "assetId": "",
    "lat": 0,
    "lng": 0,
    "name": ""
}
```

**Request Parameters**:

| Parameter name | Parameter description | Request type | Required | Data type | Schema |
| ------------ | ----------- | -------- | -------- | ---------- | ------------------ |
| updateAssetApiDTO | updateAssetApiDTO | body | true | UpdateAssetApiDTO | UpdateAssetApiDTO |
| address | Detailed address | | false | string | |
| assetId | asset ID | | false | string | |
| lat | longitude| | false | number | |
| lng | longitude| | false | number | |
| name | name | | false | string | |

**Response Status**:

| Status code | Description | Schema |
| ------ | ---------- | ---------- |
| 200 | OK | ResponseResult object «boolean» |
| 201 | Created | |
| 401 | Unauthorized | |
| 403 | Forbidden | |
| 404 | Not Found | |

**Response parameters**:

| Parameter name | Parameter description | Type | schema |
| -------- | ---------------- | ------------- | ----------- |
| code | Response status code | integer(int32) | integer(int32) |
| data | response data | boolean | |
| message | response message | string | |
| reqId | request ID | string | |
| time | request timestamp (10 bits) | integer(int64) | integer(int64) |

**Sample response**:

```JSON
{
    "code": 200,
    "data": true,
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

## Deleting assets

**Interface address**:`/v1.0/appAsset/deleteById`

**Request method**: `DELETE`

**Request data type**: `application/x-www-form-urlencoded`

**Response data type**: `*/*`

**Interface Description**:

**Request Parameters**:

| Parameter name | Parameter description | Request type | Required | Data type | Schema |
| -------- | -------- | -------- | -------- | -------- | ------ |
| assetId | assetId | query | true | string | |

**Response Status**:

| Status code | Description | Schema |
| ------ | ---------- | -------- |
| 200 | OK | ResponseResult object «boolean» |
| 204 | No Content | |
| 401 | Unauthorized | |
| 403 | Forbidden | |

**Response parameters**:

| Parameter name | Parameter description | Type | schema |
| -------- | ---------------- | ------------- | ----------- |
| code | Response status code | integer(int32) | integer(int32) |
| data | response data | boolean | |
| message | response message | string | |
| reqId | request ID | string | |
| time | request timestamp (10 bits) | integer(int64) | integer(int64) |

**Sample response**:

```JSON
{
    "code": 200,
    "data": true,
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

## Query the user asset tree

**Interface address**:`/platform/v1.0/appAsset/userAssetTree`

**Request method**: `GET`

**Request data type**: `application/x-www-form-urlencoded`

**Response data type**: `*/*`

**Interface Description**:

**Request Parameters**:

**Request Parameters**:

| Parameter name | Parameter description | Request type | Required | Data type | Schema |
| --------- | --------- | -------- | -------- | -------- | ------ |
| appUserId | appUserId | query | true | string | |

**Response Status**:

| Status code | Description | Schema |
| ------ | ---------- | --------- |
| 200 | OK | ResponseResult Object «List«AppAssetApiVo»» |
| 401 | Unauthorized | |
| 403 | Forbidden | |
| 404 | Not Found | |

**Response parameters**:

| Parameter name | Parameter description | Type | schema |
| --------- | ----------- | -------------- | -------------- |
| code | Response status code | integer(int32) | integer(int32) |
| data | response data | array | AppAssetApiVo |
| address | Detailed address | string | |
| childList | child asset list | array | AppAssetApiVo |
| id | asset id | string | |
| lat | longitude | number | |
| lng | longitude | number | |
| name | name | string | |
| parentId | parent ID | string | |
| rootId | Root asset ID | string | |
| message | response message | string | |
| reqId | request ID | string | |
| time | request timestamp (10 bits) | integer(int64) | integer(int64) |

**Sample response**:

```JSON
{
    "code": 200,
    "data": [
    {
        "address": "",
        "childList": [
        {
            "address": "",
            "childList": [
                {}
            ],
            "id": "",
            "lat": 0,
            "lng": 0,
            "name": "",
            "parentId": "",
            "rootId": ""
        }
        ],
        "id": "",
        "lat": 0,
        "lng": 0,
        "name": "",
        "parentId": "",
        "rootId": ""
    }
    ],
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

## App User Registration

**Interface address**:`/v1.0/appUser/registry`

**Request method**: `POST`

**Request data type**: `application/json`

**Response data type**: `*/*`

**Interface Description**:

**Request Example**:

```JSON
{
    "appId": "",
    "countryCode": "",
    "password": "",
    "userName": ""
}
```

**Request Parameters**:

| Parameter name | Parameter description | Request type | Required | Data type | Schema |
| ----------- | ----------------------- | -------- | -------- | ---------- | ------- |
| appUserRegistryApiDTO | appUserRegistryApiDTO | body | true | AppUserRegistryApiDTO | AppUserRegistryApiDTO |
| appId | Application ID | | false | string | |
| countryCode | International code | | false | string | |
| password | password| | false | string | |
| userName | Username | | false | string | |

**Response Status**:

| Status code | Description | Schema |
| ------ | ---------- | --------- |
| 200 | OK | ResponseResult object «AppUserVO object» |
| 201 | Created | |
| 401 | Unauthorized | |
| 403 | Forbidden | |
| 404 | Not Found | |

**Response parameters**:

| Parameter name | Parameter description | Type | schema |
| -------- | ---------------- | ------------- | ---------- |
| code | Response status code | integer(int32) | integer(int32) |
| data | response data | AppUserVO object | AppUserVO object |
| appId | Application ID | string | |
| areaCode | area code | string | |
| email | Email | string | |
| id | userid | string | |
| nickname | nickname | string | |
| phone | mobile number | string | |
| tenantId | tenant id | string | |
| userName | username | string | |
| message | response message | string | |
| reqId | request ID | string | |
| time | request timestamp (10 bits) | integer(int64) | integer(int64) |

**Sample response**:

```JSON
{
    "code": 200,
    "data": {
        "appId": "",
        "areaCode": "",
        "email": "",
        "id": "",
        "nickname": "",
        "phone": "",
        "tenantId": "",
        "userName": ""
    },
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

## Paginated query of APP users

### Interface  Description

Query APP users by page

### Interface  address

```TEXT
GET: /v1.0/appUser/page
```

### Request  Parameters

**`body`** Description **`Content-Type: application/json`**

| Parameter name | Type | Parameter position | Required | Description |
| :------------- | :---- | :---------------- | :------- | :------------ |
| pageSize | Integer | | false | Number of queries per page default: 10 |
| currentPage | Integer | | false | Query the current page number Default: 1 |
| asc | Boolean | | false | Sorting method true: ascending, false: descending (default descending) |
| orderByColumn | String | | false | Default sorting field: create_time (creation time) |
| queryCondition | queryCondition | | false | queryCondition |

**`queryCondition`** Description

| Parameter name | Type | Parameter position | Required | Description |
| -------- | ------ | ------------------- | -------- | ---------- |
| id | String | | false | User ID |
| appId | String | | false | APP ID |
| userName | String | | false | Username |
| status | int | | false | Enable status (1=enabled, 0=disabled) |

### Return  Parameters

| Parameter Name | Type | Description |
| :------ | :------ | :--------------- |
| data | Page | Device list return results |
| reqId | String | request id |
| time | Integer | Timestamp |
| code | Integer | Status code |
| message | String | message |

**`Page` Description**

| Parameter Name | Type | Description |
| :------ | :---------- | :---------- |
| total | Integer | total |
| pages | Integer | Total number of pages |
| current | Integer | current page |
| size | Integer | Query paging number |
| records | AppUser Array | Result Set |

**`AppUser` Description**

| Parameter Name | Type | Description |
| :--------- | :------ | :-------- |
| id | String | User id |
| tenantId | String | tenant id |
| nickname | String | nickname |
| userName | String | Username |
| appId | String | appId |
| appName | String | app name |
| email | String | Email |
| areaCode | String | Area code |
| avatarUrl | String | avatar URL |
| status | Integer | User status (1=enabled, 2=disabled) |
| createTime | Long | Registration Time |

### Request  Example

```TEXT
GET: /v1.0/appUser/page
```

**`body`**

```JSON
{
    "pageSize":10,
    "currentPage":1,
    "queryCondition":{
        "id":"",
        "appId":"",
        "userName":"",
        "status":1
    }
}
```

### Return  to Example

```JSON
{
    "reqId": "1963f0a0c7d06221",
    "time": 1682508594,
    "code": 200,
    "message": "success",
    "data": {
        "records": [
            {
                "id": "cn1699246765202640896",
                "tenantId": null,
                "nickname": "XQ_w42659e4",
                "userName": "15119228320",
                "appId": "bBcDt1WghhBgWF3M",
                "appName": "Rino Smart",
                "email": null,
                "areaCode": null,
                "avatarUrl": null,
                "status": 1,
                "createTime": 1693966974000,
                "deviceAccount": 0
            },
            //....
        ],
        "total": 167,
        "size": 10,
        "current": 1,
        "orders": [],
        "optimizeCountSql": true,
        "searchCount": true,
        "countId": null,
        "maxLimit": null,
        "pages": 17
    }
}
```

## Modify user password

**Interface address**:`/v1.0/appUser/updatePassword`

**Request method**: `POST`

**Request data type**: `application/json`

**Response data type**: `*/*`

**Interface Description**:

**Request Parameters**:

| Parameter name | Parameter description | Request type | Required | Data type | Schema |
| -------- | -------- | -------- | -------- | -------- | ------ |
| password | password | query | true | string | |
| userId | userId | query | true | string | |

**Response Status**:

| Status code | Description | Schema |
| ------ | ---------- | -------- |
| 200 | OK | ResponseResult object «boolean» |
| 201 | Created | |
| 401 | Unauthorized | |
| 403 | Forbidden | |
| 404 | Not Found | |

**Response parameters**:

| Parameter name | Parameter description | Type | schema |
| -------- | ---------------- | ------------- | ------------- |
| code | Response status code | integer(int32) | integer(int32) |
| data | response data | boolean | |
| message | response message | string | |
| reqId | request ID | string | |
| time | request timestamp (10 bits) | integer(int64) | integer(int64) |

**Sample response**:

```JSON
{
    "code": 200,
    "data": true,
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

## Get product intent category

**Interface address**:`/v1.0/radio/getIntentCategoryByProductId`

**Request method**: `GET`

**Request data type**: `application/x-www-form-urlencoded`

**Response data type**: `*/*`

**Interface Description**:

**Request Parameters**:

| Parameter name | Parameter description | Request type | Required | Data type | Schema |
| --------- | --------- | -------- | -------- | -------- | ------ |
| productId | productId | query | true | string | |

**Response Status**:

| Status code | Description | Schema |
| ------ | ---------- | ---------------------- |
| 200 | OK | ResponseResult Object «List«ProductIntentCategoryApiVO»» |
| 401 | Unauthorized | |
| 403 | Forbidden | |
| 404 | Not Found | |

**Response parameters**:

| Parameter name | Parameter description | Type | schema |
| ------------------ | ---------------- | ------ |-----|
| code | Response status code | integer(int32) | integer(int32) |
| data | response data | array | ProductIntentCategoryApiVO |
| intentCategory | intent category | string | |
| intentCategoryId | primary key id | string | |
| intentCategoryName | intent category name | string | |
| productId | product id | string | |
| radioType | voice type | string | |
| message | response message | string | |
| reqId | request ID | string | |
| time | request timestamp (10 bits) | integer(int64) | integer(int64) |

**Sample response**:

```JSON
{
    "code": 200,
    "data": [
    {
        "intentCategory": "",
        "intentCategoryId": "",
        "intentCategoryName": "",
        "productId": "",
        "radioType": ""
    }
    ],
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

## Get product intent

**Interface address**:`/v1.0/radio/getIntentByProductId`

**Request method**: `GET`

**Request data type**: `application/x-www-form-urlencoded`

**Response data type**: `*/*`

**Interface Description**:

**Request Parameters**:

| Parameter name | Parameter description | Request type | Required | Data type | Schema |
| --------- | --------- | -------- | -------- | -------- | ------ |
| productId | productId | query | true | string | |

**Response Status**:

| Status code | Description | Schema |
| ------ | ---------- | ------------- |
| 200 | OK | ResponseResult Object «List«ProductIntentApiVO»» |
| 401 | Unauthorized | |
| 403 | Forbidden | |
| 404 | Not Found | |

**Response parameters**:

| Parameter name | Parameter description | Type | schema |
| --------------- | --------------- | --------- | --------- |
| code | Response status code | integer(int32) | integer(int32) |
| data | response data | array | ProductIntentApiVO |
| dpDataType | dp point data type | string | |
| dpIdentifier | dp point | string | |
| dpName | dp point name | string | |
| dpValue | dp point data | string | |
| intentCode | intent code (corresponding to rino_intent table code) | string | |
| intentValue | intent value | string | |
| isSendDp | Whether to send dp point | boolean | |
| productId | product id | string | |
| productIntentId | primary key id | string | |
| message | response message | string | |
| reqId | request ID | string | |
| time | request timestamp (10 bits) | integer(int64) | integer(int64) |

**Sample response**:

```JSON
{
    "code": 200,
    "data": [
    {
        "dpDataType": "",
        "dpIdentifier": "",
        "dpName": "",
        "dpValue": "",
        "intentCode": "",
        "intentValue": "",
        "isSendDp": true,
        "productId": "",
        "productIntentId": ""
    }
    ],
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

## Query Baidu intent list

**Interface address**:`/v1.0/baidu/intent/queryList`

**Request method**: `POST`

**Request data type**: `application/json`

**Response data type**: `*/*`

**Interface Description**:

**Request Example**:

```JSON
{
    "intentCode": "",
    "intentName": ""
}
```

**Request Parameters**:

| Parameter name | Parameter description | Request type | Required | Data type | Schema |
| ---------- | -------- | -------- | -------- | -------------- |---------------- |
| query | query | body | true | BaiduIntentQuery | BaiduIntentQuery |
| intentCode | intent code | | false | string | |
| intentName | intent name | | false | string | |

**Response Status**:

| Status code | Description | Schema |
| ------ | ---------- | ---------------- |
| 200 | OK | ResponseResult object «List«BaiduIntentApiVO»» |
| 201 | Created | |
| 401 | Unauthorized | |
| 403 | Forbidden | |
| 404 | Not Found | |

**Response parameters**:

| Parameter name | Parameter description | Type | schema |
| ------------- | -------------- | ----------------- | ---------------- |
| code | Response status code | integer(int32) | integer(int32) |
| data | response data | array | BaiduIntentApiVO |
| baiduIntent | Baidu intent | string | |
| baiduIntentId | primary key | string | |
| baiduValue | attribute value (Baidu) | string | |
| createTime | creation time | string | |
| intentCode | Platform intent code, corresponding to rino_intent table code | string | |
| intentName | Intent name, corresponding to rino_intent table name | string | |
| intentValue | command value | string | |
| realName | intent name | string | |
| remark | remark | string | |
| updateTime | modification time | string | |
| message | response message | string | |
| reqId | request ID | string | |
| time | request timestamp (10 bits) | integer(int64) | integer(int64) |

**Sample response**:

```JSON
{
    "code": 200,
    "data": [
    {
        "baiduIntent": "",
        "baiduIntentId": "",
        "baiduValue": "",
        "createTime": "",
        "intentCode": "",
        "intentName": "",
        "intentValue": "",
        "realName": "",
        "remark": "",
        "updateTime": ""
    }
    ],
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

## Get the weather conditions of the specified city (Moji Weather)

**Interface address**:`/v1.0/weather/getByCityId`

**Request method**: `GET`

**Request data type**: `application/x-www-form-urlencoded`

**Response data type**: `*/*`

**Interface Description**:

**Request Parameters**:

| Parameter name | Parameter description | Request type | Required | Data type | Schema |
| -------- | -------- | -------- | -------- | -------- | ------ |
| cityId | cityId | query | true | string | |

**Response Status**:

| Status code | Description | Schema |
| ------ | ---------- | -------- |
| 200 | OK | ResponseResult object «ConditionVO» |
| 401 | Unauthorized | |
| 403 | Forbidden | |
| 404 | Not Found | |

**Response parameters**:

| Parameter name | Parameter description | Type | schema |
| -------------------------- | ----------- | -------- | ----------- |
| code | Response status code | integer(int32) | integer(int32) |
| data | response data | ConditionVO | ConditionVO |
| airQuality | air quality index value | string | |
| cityId | city ID | string | |
| condition | Real-time weather phenomenon | string | |
| conditionId | Real-time weather ID | string | |
| humidity | humidity, unit: % | string | |
| icon | weatherICON | string | |
| lat | latitude | number | |
| lon | longitude | number | |
| name | city name | string | |
| pm25 | PM2.5 index | string | |
| provinceName | province name | string | |
| source | Data source (1=ink, 2=Weatherbit) | integer | |
| sunrise | sunset | string | |
| sunset | sunrise | string | |
| temp | temperature, unit: Celsius | string | |
| windSpeed ​​| wind speed | string | |
| message | response message | string | |
| reqId | request ID | string | |
| time | request timestamp (10 bits) | integer(int64) | integer(int64) |

**Sample response**:

```JSON
{
    "code": 200,
    "data": {
        "airQuality": "",
        "cityId": "",
        "condition": "",
        "conditionId": "",
        "humidity": "",
        "icon": "",
        "lat": 0,
        "lon": 0,
        "name": "",
        "pm25": "",
        "provinceName": "",
        "source": 0,
        "sunrise": "",
        "sunset": "",
        "temp": "",
        "windSpeed": ""
    },
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

## Get the product scene linkage action function point list

### Interface Description

Get a list of all DP points that support actions in the product scene linkage settings

### Interface address

```TEXT
GET: /v1.0/product/dp/getDpListByProductId
```

### Request Parameters

| Parameter name | Type | Parameter position | Required | Description |
| :-------- | :----- | :------- | :------- | :----- |
| productId | String | path | true | productId |

### Return Parameters

| Parameter Name | Type | Description |
| :------ | :------ | :------- |
| data | Product | Equipment data |
| reqId | String | request id |
| time | Integer | Timestamp |
| code | Integer | Status code |
| message | String | message |

**`Product` Description**

| Parameter Name | Type | Description |
| :----- | :----- | :------- |
| dpId | String | Function point id |

### Request Example

```JSON
GET: /v1.0/productLinkage/getActionDpListByProductId/8mixAlpJIEnIWU
```

### Return to Example

```JSON
{
    "reqId": "1963f0a0c7d06221",
    "time": 1682508594,
    "code": 200,
    "message": "success",
    "data": [
    {
    
    }
    ]
}
```

## Paginated product query

### Interface Description

Pagination query product list

### Interface address

```TEXT
GET: /v1.0/product/page
```

### Request Parameters

**`body`** Description **`Content-Type: application/json`**

| Parameter name | Type | Parameter position | Required | Description |
| :------------- | :------ | :------- | :------- | :----------- |
| pageSize | Integer | | false | Number of queries per page default: 10 |
| currentPage | Integer | | false | Query the current page number Default: 1 |
| asc | Boolean | | false | Sorting method true: ascending, false: descending (default descending) |
| orderByColumn | String | | false | Default sorting field: create_time (creation time) |
| queryCondition | queryCondition | | false | queryCondition |

**`queryCondition`** Description

| Parameter name | Type | Parameter position | Required | Description |
| ------ | ------ | -------- | -------- | -------- |
| name | String | | false | Product name |
| typeId | String | | false | Category ID |

### Return Parameters

| Parameter Name | Type | Description |
| :------ | :------ | :--------------- |
| data | Page | Device list return results |
| reqId | String | request id |
| time | Integer | Timestamp |
| code | Integer | Status code |
| message | String | message |

**`Page` Description**

| Parameter Name | Type | Description |
| :------ | :---------- | :---------- |
| total | Integer | total |
| pages | Integer | Total number of pages |
| current | Integer | current page |
| size | Integer | Query paging number |
| records | Product Array | Result Set |

**`Product` Description**

| Parameter Name | Type | Description |
| :----------- | :------ | :----------------------- |
| id | String | product id |
| name | String | Product name |
| typeId | String | product type id |
| typeName | String | Product type name |
| tenantId | String | tenant id |
| imageUrl | String | Product image |
| model | String | Product model |
| nodeType | Integer | node type |
| status | Integer | Device status (0 disabled, 1 enabled) |
| protocolType | String | Communication protocol |
| createTime | Long | creation time, timestamp |

### Request Example

```TEXT
GET: /v1.0/product/page
```

**`body`**

```JSON
{
    "pageSize":10,
    "currentPage":1,
    "queryCondition":{
        "name":"",
        "typeId":""
    }
}
```

### Return to Example

```JSON
{
    "reqId": "1963f0a0c7d06221",
    "time": 1682508594,
    "code": 200,
    "message": "success",
    "data": {
        "records": [
            {
                "id": "8mixAlpJIEnIWU",
                "typeId": "rgb-strip",
                "typeName": "RGB light strip",
                "tenantId": "LSHkcfojvV5a0m29",
                "name": "Changsha--android_tets",
                "imageUrl": "https://rinioot-app-1313015441.cos.ap-guangzhou.myqcloud.com/rgb_dd.png",
                "model": "dd001",
                "nodeType": 1,
                "protocolType": "1",
                "status": 1,
                "createTime": 1682215983000
            },
            ///....
        ],
        "total": 167,
        "size": 10,
        "current": 1,
        "orders": [],
        "optimizeCountSql": true,
        "searchCount": true,
        "countId": null,
        "maxLimit": null,
        "pages": 17
    }
}
```

## Query Products

### Interface Description

Query device information based on product ID

### Interface address

```TEXT
GET: /v1.0/product/detail/{productId}
```

### Request Parameters

| Parameter name | Type | Parameter position | Required | Description |
| :-------- | :----- | :------- | :------- | :----- |
| productId | String | path | true | productId |

### Return Parameters

| Parameter Name | Type | Description |
| :------ | :------ | :------- |
| data | Product | Equipment data |
| reqId | String | request id |
| time | Integer | Timestamp |
| code | Integer | Status code |
| message | String | message |

**`Product` Description**

| Parameter Name | Type | Description |
| :----------- | :------ | :--------- |
| id | String | product id |
| name | String | Product name |
| typeId | String | product type id |
| typeName | String | Product type name |
| tenantId | String | tenant id |
| imageUrl | String | Product image |
| model | String | Product model |
| nodeType | Integer | node type |
| status | Integer | Device status (0 disabled, 1 enabled) |
| protocolType | String | Communication protocol |
| createTime | Long | creation time, timestamp |

### Request Example

```TEXT
GET: /v1.0/product/detail/8mixAlpJIEnIWU
```

### Return to Example

```JSON
{
    "reqId": "1963f0a0c7d06221",
    "time": 1682508594,
    "code": 200,
    "message": "success",
    "data": {
        "id": "8mixAlpJIEnIWU",
        "typeId": "rgb-strip",
        "typeName": "RGB light strip",
        "tenantId": "LSHkcfojvV5a0m29",
        "name": "Changsha--android_tets",
        "imageUrl": "https://rinioot-app-1313015441.cos.ap-guangzhou.myqcloud.com/rgb_dd.png",
        "model": "dd001",
        "nodeType": 1,
        "protocolType": "1",
        "status": 1,
        "createTime": 1682215983000
    }
}
```

## Get the product scene linkage action function point list

### Interface Description

Get a list of all DP points that support actions in the product scene linkage settings

### Interface address

```TEXT
GET: /v1.0/productLinkage/getActionDpListByProductId/{productId}
```

### Request Parameters

| Parameter name | Type | Parameter position | Required | Description |
| :-------- | :----- | :------- | :------- | :----- |
| productId | String | path | true | productId |

### Return Parameters

| Parameter Name | Type | Description |
| :------ | :------ | :------- |
| data | Product | Equipment data |
| reqId | String | request id |
| time | Integer | Timestamp |
| code | Integer | Status code |
| message | String | message |

**`Product` Description**

| Parameter Name | Type | Description |
| :------------- | :------ | :-------- |
| dpId | String | Function point id |
| dpName | String | Function point name |
| identifier | String | Function point unique identifier |
| type | String | Function point type |
| dataType | String | Function point data type |
| specs | String | Function point extension parameters |
| valueCastType | Integer | Whether to perform percentage conversion (1=yes) |

### Request Example

```TEXT
GET: /v1.0/productLinkage/getActionDpListByProductId/8mixAlpJIEnIWU
```

### Return to Example

```JSON
{
    "reqId": "1963f0a0c7d06221",
    "time": 1682508594,
    "code": 200,
    "message": "success",
    "data": [
    {
        "dpId":"1963f0a0c7d06221",
        "dpName":"Switch",
        "identifier":"switch_led",
        "type":"properties",
        "dataType":"bool",
        "specs":{
            "_true":"Open",
            "_false":"Off"
        },
        "valueCastType": null
    }
    ]
}
```

## Get product shortcut switch list

**Interface address**:`/v1.0/productFastSwitch/getFastSwitchListByProductId`

**Request method**: `GET`

**Request data type**: `application/x-www-form-urlencoded`

**Response data type**: `*/*`

**Interface Description**:

**Request Parameters**:

| Parameter name | Parameter description | Request type | Required | Data type | Schema |
| --------- | --------- | -------- | -------- | -------- | ------ |
| productId | productId | query | true | string | |

**Response Status**:

| Status code | Description | Schema |
| ------ | ---------- | ------------- |
| 200 | OK | ResponseResult Object «List«ProductFastSwitchVO Objects»» |
| 401 | Unauthorized | |
| 403 | Forbidden | |
| 404 | Not Found | |

**Response parameters**:

| Parameter name | Parameter description | Type | schema |
|------------- | ------------- | ---------- | ---------- |
| code | Response status code | integer(int32) | integer(int32) |
| data | response data | array | ProductFastSwitchVO object |
| dpDataType | data type | string | |
| dpId | DP_ID | string | |
| dpJson | function json | string | |
| dpKey | property identifier | string | |
| dpName | property identifier | string | |
| id | primary key | string | |
| imageUrl | Feature point image | string | |
| productId | product ID | string | |
| showToApp | Whether to display to APP | integer | |
| sortNumber | sort | integer | |
| type | Type (1=quick switch, 2=common function) | integer | |
| value | property value | object | |
| valueCastType | DP value conversion type (0=original value, 1=percentage) | integer | |
| message | response message | string | |
| reqId | request ID | string | |
| time | request timestamp (10 bits) | integer(int64) | integer(int64) |

**Sample response**:

```JSON
{
    "code": 200,
    "data": [
    {
        "dpDataType": "",
        "dpId": "",
        "dpJson": "",
        "dpKey": "",
        "dpName": "",
        "id": "",
        "imageUrl": "",
        "productId": "",
        "showToApp": 0,
        "sortNumber": 0,
        "type": 0,
        "value": {},
        "valueCastType": 0
    }
    ],
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

## Get the supported scene linkage action list in batches based on product ID

**Interface address**:`/v1.0/productLinkage/getActionDpListByProductIds`

**Request method**: `POST`

**Request data type**: `application/json`

**Response data type**: `*/*`

**Interface Description**:

**Request Example**:

```TEXT
[]
```

**Request Parameters**:

| Parameter name | Parameter description | Request type | Required | Data type | Schema |
| ---------- | ---------- | -------- | -------- | -------- | ------ |
| productIds | productIds | body | true | array | string |

**Response Status**:

| Status code | Description | Schema |
| ------ | ---------- | ------------- |
| 200 | OK | ResponseResult object «List«ProductLinkageActionDpVO»» |
| 201 | Created | |
| 401 | Unauthorized | |
| 403 | Forbidden | |
| 404 | Not Found | |

**Response parameters**:

| Parameter name | Parameter description | Type | schema |
| ------------- | -------------------- | ----- |------------|
| code | Response status code | integer(int32) | integer(int32) |
| data | response data | array | ProductLinkageActionDpVO |
| actionDpList | | array | DpOption |
| checked | Is it checked | boolean | |
| dataType | data type | string | |
| dpId | DPID | string | |
| dpName | dp name | string | |
| identifier | dp identifier | string | |
| specs | data value | object | |
| type | data type | string | |
| valueCastType | (0=original value, 1=percentage) | integer | |
| productId | | string | |
| message | response message | string | |
| reqId | request ID | string | |
| time | request timestamp (10 bits) | integer(int64) | integer(int64) |

**Sample response**:

```JSON
{
    "code": 200,
    "data": [
    {
        "actionDpList": [
        {
            "checked": true,
            "dataType": "",
            "dpId": "",
            "dpName": "",
            "identifier": "",
            "specs": {},
            "type": "",
            "valueCastType": 0
        }
        ],
        "productId": ""
    }
    ],
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

## Get the product multi-channel configuration function point list

**Interface address**:`/v1.0/product/multiChannel/getListByProductId`

**Request method**: `GET`

**Request data type**: `application/x-www-form-urlencoded`

**Response data type**: `*/*`

**Interface Description**:

**Request Parameters**:

| Parameter name | Parameter description | Request type | Required | Data type | Schema |
| --------- | -------- | -------- | -------- | -------- | ------ |
| productId | productId | query | false | string | |

**Response Status**:

| Status code | Description | Schema |
| ------ | ---------- | ---------------------- |
| 200 | OK | ResponseResult Object «List«ProductMultiChannelApiVO»» |
| 401 | Unauthorized | |
| 403 | Forbidden | |
| 404 | Not Found | |

**Response parameters**:

| Parameter name | Parameter description | Type | schema |
| ------------ | ---------------- | --------- | ---------------- |
| code | Response status code | integer(int32) | integer(int32) |
| data | response data | array | ProductMultiChannelApiVO |
| dpIdentifier | Function point identifier | string | |
| id | Function point ID | string | |
| mccId | record ID | string | |
| message | response message | string | |
| reqId | request ID | string | |
| time | request timestamp (10 bits) | integer(int64) | integer(int64) |

**Sample response**:

```JSON
{
    "code": 200,
    "data": [
    {
        "dpIdentifier": "",
        "id": "",
        "mccId": ""
    }
    ],
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

## Product Category Tree

### Interface Description

Get product category tree data

### Interface address

```TEXT
GET: /v1.0/productType/tree
```

### Request Parameters

| Parameter Name | Type | Description |
| :----- | :--- | :--- |
| | | |

### Return Parameters

| Parameter Name | Type | Description |
| :------ | :---------------- | :--------------- |
| data | ProductType Array | Product category data array |
| reqId | String | request id |
| time | Integer | Timestamp |
| code | Integer | Status code |
| message | String | message |

**`ProductType` Description**

| Parameter Name | Type | Description |
| :-------- | :---------------- | :-------------------- - |
| childList | ProductType Array | Product subcategory array |
| id | String | Product type id |
| imageUrl | String | Image URL |
| name | String | Product type name |
| pid | String | parent id |
| status | Integer | Status (0: disabled, 1: enabled) |

### Request Example

```TEXT
GET: /v1.0/productType/tree
```

### Return to Example

```JSON
{
    "code": 200,
    "data": [
    {
        "childList": [
        {
            "childList": [
                {}
            ],
            "id": "",
            "imageUrl": "",
            "name": "",
            "pid": "",
            "status": 0
        }
        ],
        "id": "",
        "imageUrl": "",
        "name": "",
        "pid": "",
        "status": 0
    }
    ],
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

## Check if the device firmware needs to be upgraded (if firmwareType is not passed, the module firmware will be detected by default. It is recommended to pass the parameter.)

**Interface address**:`/v1.0/device/ota/checkUserDeviceUpgrade/{deviceId}`

**Request method**: `POST`

**Request data type**: `application/json`

**Response data type**: `*/*`

**Interface Description**:

**Request Parameters**:

| Parameter name | Parameter description | Request type | Required | Data type | Schema |
| -------------- | ---------------------- | -------- | -------- | ---------- | ------ |
| deviceId | device ID | path | false | string | |
| firmwareType | Firmware type (1-public firmware 2-module firmware 3-mcu firmware) | query | false | integer(int32) | |
| userId | userId | query | false | string | |

**Response Status**:

| Status code | Description | Schema |
| ------ | ---------- | ------------ |
| 200 | OK | ResponseResult Object «List«DeviceOtaResultApiVO»» |
| 201 | Created | |
| 401 | Unauthorized | |
| 403 | Forbidden | |
| 404 | Not Found | |

**Response parameters**:

| Parameter name | Parameter description | Type | schema |
|---------------- |------------------- | ------- | -------- |
| code | Response status code | integer(int32) | integer(int32) |
| data | response data | array | DeviceOtaResultApiVO |
| deviceId | device ID | string | |
| deviceName | device name | string | |
| deviceVersion | Current firmware version of the device | string | |
| firmwareFileSize | Firmware file size (bytes) | integer | |
| firmwareId | firmware ID | string | |
| firmwareMd5 | firmware file MD5 | string | |
| firmwareName | Firmware file name | string | |
| firmwareType | Firmware type (1 public firmware, 2 module firmware (standard firmware), 3 MCU firmware, 4 Bluetooth firmware, 5 Zigbee firmware, 6 extended firmware, 7 Thread firmware) | integer | |
| firmwareUrl | Firmware download address | string | |
| imgUrl | device image | string | |
| publishTime | publish time | integer | |
| taskId | Firmware upgrade task ID | string | |
| upgradeInfo | Upgrade content | string | |
| upgradeType | Upgrade type (1=App reminder upgrade, 2=App detection upgrade, 3=App forced upgrade) | integer | |
| versionTarget | Current firmware version | string | |
| message | response message | string | |
| reqId | request ID | string | |
| time | request timestamp (10 bits) | integer(int64) | integer(int64) |

**Sample response**:

```JSON
{
    "code": 200,
    "data": [
    {
        "deviceId": "",
        "deviceName": "",
        "deviceVersion": "",
        "firmwareFileSize": 0,
        "firmwareId": "",
        "firmwareMd5": "",
        "firmwareName": "",
        "firmwareType": 0,
        "firmwareUrl": "",
        "imgUrl": "",
        "publishTime": 0,
        "taskId": "",
        "upgradeInfo": "",
        "upgradeType": 0,
        "versionTarget": ""
    }
    ],
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

## Device online status statistics

### Interface Description

Get online status statistics based on product ID list and category ID list

### Interface address

```TEXT
GET: /v1.0/statistics/countByOnlineStatus
```

### Request Parameters

**`body`** Description **`Content-Type: application/json`**

| Parameter name | Type | Parameter position | Required | Description |
| :---------- | :----------- | :------- | :------- | :--------- |
| productIds | String Array | | false | product id list |
| typeIds | String Array | | false | Category id list |

### Return Parameters

| Parameter Name | Type | Description |
| :------ | :---------- | :------------- |
| data | OnlineStatusCount | Device DP data array |
| reqId | String | request id |
| time | Integer | Timestamp |
| code | Integer | Status code |
| message | String | message |

**`OnlineStatusCount` description**

| Parameter Name | Type | Description |
| :--------- | :------ | :--------- |
| allNum | integer | total number of devices |
| offlineNum | integer | Number of offline devices |
| onlineNum | integer | Number of online devices |

### Request Example

```TEXT
GET: /v1.0/statistics/countByOnlineStatus
```

**`body`**

```JSON
{
    "productIds":[],
    "typeIds":[]
}
```

### Return to Example

```JSON
{
    "code": 200,
    "data": {
        "allNum": 0,
        "offlineNum": 0,
        "onlineNum": 0
    },
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

## Equipment product group statistics

### Interface Description

Get online status group statistics based on product ID list and category ID list

### Interface address

```TEXT
GET: /v1.0/statistics/countByProductId
```

### Request Parameters

**`body`** Description **`Content-Type: application/json`**

| Parameter name | Type | Parameter position | Required | Description |
| :---------- | :----------- | :------- | :------- | :--------- |
| productIds | String Array | | false | product id list |
| typeIds | String Array | | false | Category id list |

### Return Parameters

| Parameter Name | Type | Description |
| :------ | :------- | :------------- |
| data | DeviceProductCount Array | Device DP data array |
| reqId | String | request id |
| time | Integer | Timestamp |
| code | Integer | Status code |
| message | String | message |

**`DeviceProductCount` description**

| Parameter Name | Type | Description |
| :---------- | :------ | :---------- |
| allNum | Integer | Total number of devices |
| offlineNum | Integer | Number of offline devices |
| onlineNum | Integer | Number of online devices |
| productId | String | Product ID |
| productName | String | Product name |

### Request Example

```TEXT
GET: /v1.0/statistics/countByProductId
```

**`body`**

```JSON
{
    "productIds":[],
    "typeIds":[]
}
```

### Return to Example

```JSON
{
    "code": 200,
    "data":[
    {
        "allNum": 0,
        "offlineNum": 0,
        "onlineNum": 0,
        "productId": "",
        "productName": ""
    }
    ],
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

## Equipment category grouping statistics

### Interface Description

Get online status and group statistics by category based on product ID list and category ID list

### Interface address

```TEXT
GET: /v1.0/statistics/countByTypeId
```

### Request Parameters

**`body`** Description **`Content-Type: application/json`**

| Parameter name | Type | Parameter position | Required | Description |
| :---------- | :----------- | :------- | :------- | :------ |
| productIds | String Array | | false | product id list |
| typeIds | String Array | | false | Category id list |

### Return Parameters

| Parameter Name | Type | Description |
| :------ | :-------- | :------- |
| data | DeviceProductTypeCount Array | Device DP data array |
| reqId | String | request id |
| time | Integer | Timestamp |
| code | Integer | Status code |
| message | String | message |

**`DeviceProductCount` description**

| Parameter Name | Type | Description |
| :---------- | :------ | :----------- |
| allNum | Integer | Total number of devices |
| offlineNum | Integer | Number of offline devices |
| onlineNum | Integer | Number of online devices |
| typeId | String | Product category ID |
| typeName | String | Product category name |

### Request Example

```TEXT
GET: /v1.0/statistics/countByTypeId
```

**`body`**

```JSON
{
    "productIds":[],
    "typeIds":[]
}
```

### Return to Example

```JSON
{
    "code": 200,
    "data":[
    {
        "allNum": 0,
        "offlineNum": 0,
        "onlineNum": 0,
        "typeId": "",
        "typeName": ""
    }
    ],
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

## Device DP data statistics

### Interface Description

Get device DP data statistics based on device information

### Interface address

```TEXT
GET: /v1.0/statistics/deviceDpSum
```

### Request Parameters

**`body`** Description **`Content-Type: application/json`**

| Parameter name | Type | Parameter position | Required | Description |
| :----------- | :------ | :------- | :------- | :----------- |
| productId | String | | false | Product id (at least one of the device id/device uuid must be passed) |
| deviceId | String | | false | Device ID (at least one of the product ID/device UUID must be passed) |
| deviceUuid | String | | false | Device UUID (pass at least one with product id/device id) |
| dp | String | | true | dp point (required) |
| startTime | Long | | true | Start time (millisecond timestamp, required) |
| endTime | Long | | true | End time (millisecond timestamp, required) |
| intervalType | Integer | | true | Time interval type: 1 hour; 2 days; 3 weeks; 4 months; (required) |

### Return Parameters

| Parameter Name | Type | Description |
| :------ | :---------------- | :----------------- |
| data | DeviceDpSum Array | Device DP statistics array |
| reqId | String | request id |
| time | Integer | Timestamp |
| code | Integer | Status code |
| message | String | message |

**`DeviceDpSum` description**

| Parameter Name | Type | Description |
| :--------------- | :------ | :------- |
| startTime | Long | start time |
| endTime | Long | end time |
| apercentileValue | Integer | median value |
| avgValue | Integer | Average value |
| countValue | Integer | Number of records |
| maxvalue | Integer | maximum value |
| sumValue | Integer | sum value |

### Request Example

```TEXT
GET: /v1.0/statistics/deviceDpSum
```

**`body`**

```JSON
{
    "productId": "",
    "deviceId": "",
    "deviceUuid": "",
    "dp": "",
    "startTime": 1601234667000,
    "endTime": 1601234667000,
    "intervalType": 1

}
```

### Return to Example Return to Example

```JSON
{
    "code": 200,
    "data": [{
        "startTime": 1601234567000,
        "endTime": 1601234667000,
        "apercentileValue": 0,
        "avgValue": 0,
        "countValue": 0,
        "maxvalue": 0,
        "sumValue": 0
    }],
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

## Set device properties

### Interface Description

Set the device DP attribute value

### Interface address

```TEXT
POST: /v1.0/deviceIssue/{deviceId}/propertySet
```

### Request Parameters

| Parameter name | Type | Parameter position | Required | Description |
| :------- | :----- | :------- | :------- | :----- |
| deviceId | String | path | true | device ID |

**`body`** Description **`Content-Type: application/json`**

| Parameter name | Type | Parameter position | Required | Description |
| :--------- | :----- | :------- | :------- | :--------------- |
| properties | Object | | true | DP property settings, JSON format key-value pair, key=dp property identifier; value=property value |

### Return Parameters

| Parameter Name | Type | Description |
| :------ | :------ | :------- |
| data | boolean | return result |
| reqId | String | request id |
| time | Integer | Timestamp |
| code | Integer | Status code |
| message | String | message |

### Request Example

```TEXT
GET: /v1.0/deviceIssue/rn4035353332422/propertySet
```

**`body`**

```JSON
{
    "properties":{
       "color":"red",
       "switch":true
    }
}
```

### Return to Example

```JSON
{
    "reqId": "1963f0a0c7d06221",
    "time": 1682508594,
    "code": 200,
    "message": "success",
    "data": true
}
```

## Set device properties

### Interface Description

Set the device DP attribute value

### Interface address

```TEXT
POST: /v1.0/deviceIssue/batchPropertySet
```

### Request Parameters

**`body`** Description **`Content-Type: application/json`**

| Parameter name | Type | Parameter position | Required | Description |
| :-------------- | :---------------- | :------ | :------- | :------------- |
| propertySetList | PropertySet Array | | true | DP property setting list |

**`PropertySet` description**

| Parameter Name | Type | Description |
| :--------- | :----- | :----------------------- |
| deviceId | String | device id |
| properties | Object | DP property settings, key-value pairs in JSON format, key=dp property identifier; value=property value |

### Return Parameters

| Parameter Name | Type | Description |
| :------ | :------ | :------- |
| data | boolean | return result |
| reqId | String | request id |
| time | Integer | Timestamp |
| code | Integer | Status code |
| message | String | message |

### Request Example

```TEXT
GET: /v1.0/deviceIssue/batchPropertySet
```

Body

```JSON
[  
    {
        "deviceId":"xxxxx",
        "properties":{
             "color":"red",
             "switch":true
        }
    }
    //......
]
```

### Return to Example

```JSON
{
    "reqId": "1963f0a0c7d06221",
    "time": 1682508594,
    "code": 200,
    "message": "success",
    "data": true
}
```

## Detect device online status

**Interface address**: `/v1.0/deviceIssue/{deviceId}/ping`

**Request method**: `POST`

**Request data type**: `application/json`

**Response data type**: `*/*`

**Interface Description**:

**Request Parameters**:

| Parameter name | Parameter description | Request type | Required | Data type | Schema |
| -------- | -------- | -------- | -------- | -------- | ------ |
| deviceId | device ID | path | false | string | |

**Response Status**:

| Status code | Description | Schema |
| ------ | ---------- | ------------ |
| 200 | OK | ResponseResult object «boolean» |
| 201 | Created | |
| 401 | Unauthorized | |
| 403 | Forbidden | |
| 404 | Not Found | |

**Response parameters**:

| Parameter name | Parameter description | Type | schema |
| -------- | ---------------- | ------------- | --------- |
| code | Response status code | integer(int32) | integer(int32) |
| data | response data | boolean | |
| message | response message | string | |
| reqId | request ID | string | |
| time | request timestamp (10 bits) | integer(int64) | integer(int64) |

**Sample response**:

```JSON
{
    "code": 200,
    "data": true,
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

## Update device local configuration

**Interface address**:`/v1.0/deviceIssue/{uuid}/updateLocalConfig`

**Request method**: `POST`

**Request data type**: `application/json`

**Response data type**: `*/*`

**Interface Description**:

**Request Parameters**:

| Parameter name | Parameter description | Request type | Required | Data type | Schema |
| -------- | -------- | -------- | -------- | -------- | ------ |
| uuid | device UUID | path | false | string | |

**Response Status**:

| Status code | Description | Schema |
| ------ | ---------- | ---------------------------- |
| 200 | OK | ResponseResult object «boolean» |
| 201 | Created | |
| 401 | Unauthorized | |
| 403 | Forbidden | |
| 404 | Not Found | |

**Response parameters**:

| Parameter name | Parameter description | Type | schema |
| -------- | ---------------- | ------------- | ----------- |
| code | Response status code | integer(int32) | integer(int32) |
| data | response data | boolean | |
| message | response message | string | |
| reqId | request ID | string | |
| time | request timestamp (10 bits) | integer(int64) | integer(int64) |

**Sample response**:

```JSON
{
    "code": 200,
    "data": true,
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

## Paging query device

### Interface Description

Paginated query device list

### Interface address

```TEXT
GET: /v1.0/device/page
```

### Request Parameters

**`body`** Description **`Content-Type: application/json`**

| Parameter name | Type | Parameter position | Required | Description |
| :------------- | :------------- | :------- | :------- | :------------------- |
| pageSize | Integer | | false | Number of queries per page default: 10 |
| currentPage | Integer | | false | Query the current page number Default: 1 |
| asc | Boolean | | false | Sorting method true: ascending, false: descending (default descending) |
| orderByColumn | String | | false | Default sorting field: create_time (creation time) |
| queryCondition | queryCondition | | false | queryCondition |

**`queryCondition`** Description

| Parameter name | Type | Parameter position | Required | Description |
| ------------ | ------- | -------- | -------- | ------------------ |
| key | String | | false | Device name/uuid (fuzzy matching) |
| name | String | | false | Device name (fuzzy matching) |
| uuid | String | | false | Module serial number |
| productId | String | | false | Product ID |
| onlineStatus | Integer | | false | Online status (0 offline, 1 online) |

### Return Parameters

| Parameter Name | Type | Description |
| :------ | :------ | :--------------- |
| data | Page | Device list return results |
| reqId | String | request id |
| time | Integer | Timestamp |
| code | Integer | Status code |
| message | String | message |

**`Page` Description**

| Parameter Name | Type | Description |
| :------ | :----------- | :---------- |
| total | Integer | total |
| pages | Integer | Total number of pages |
| current | Integer | current page |
| size | Integer | Query paging number |
| records | Device Array | Result Set |

**`Device` Description**

| Parameter Name | Type | Description |
| :------------- | :------ | :------------- |
| id | String | device id |
| productId | String | product id |
| productName | String | Product name |
| name | String | Device name |
| uuid | String | Module serial number |
| protocolType | String | Communication protocol |
| ip | String | ip address |
| activationTime | Long | Activation timestamp accurate to milliseconds |
| status | Integer | Device status (0 disabled, 1 enabled) |
| gateway | Integer | Is it a gateway: 1 for yes, 0 for no |
| gatewayType | Integer | Gateway type: 1 normal gateway 2 edge gateway |
| typeId | String | product type id |
| typeName | String | Product category name |
| tenantId | String | tenant id |

### Request Example

```TEXT
GET: /v1.0/device/page
```

**`body`**

```JSON
{
    "pageSize":10,
    "currentPage":1,
    "queryCondition":{
        "key":"Light strip"
    }
}
```

### Return to Example

```JSON
{
    "reqId": "1963f0a0c7d06221",
    "time": 1682508594,
    "code": 200,
    "message": "success",
    "data": {
        "records": [
            {
                "id": "1650435719915794432",
                "productId": "UPAxVaiQ7monsG",
                "productName": "Lightstrip t",
                "name": "Colorful Light Strip",
                "uuid": "rn01327e6a4a0c7c",
                "protocolType": "1",
                "ip": "175.9.141.162",
                "activationTime": 1682329514000,
                "onlineStatus": 0,
                "status": 1,
                "gateway": 0,
                "gatewayType": null,
                "typeId": "rgb-strip",
                "typeName": "RGB Strip Light",
                "tenantId": "LSHkcfojvV5a0m29"
            },
            //…
        ],
        "total": 167,
        "size": 10,
        "current": 1,
        "orders": [],
        "optimizeCountSql": true,
        "searchCount": true,
        "countId": null,
        "maxLimit": null,
        "pages": 17
    }
}
```

## Query device

### Interface Description

Query device information based on module serial number

### Interface address

```TEXT
GET: /v1.0/device/detail/{uuid}
```

### Request Parameters

| Parameter name | Type | Parameter position | Required | Description |
| :----- | :----- | :------- | :------- | :------- |
| uuid | String | path | true | Device UUID |

### Return Parameters

| Parameter Name | Type | Description |
| :------ | :------ | :------- |
| data | Device | Device data |
| reqId | String | request id |
| time | Integer | Timestamp |
| code | Integer | Status code |
| message | String | message |

**`Device` Description**

| Parameter Name | Type | Description |
| :------------- | :------ | :----------- |
| id | String | device id |
| productId | String | product id |
| productName | String | Product name |
| name | String | Device name |
| uuid | String | Module serial number |
| protocolType | String | Communication protocol |
| ip | String | ip address |
| activationTime | Long | Activation timestamp accurate to milliseconds |
| status | Integer | Device status (0 disabled, 1 enabled) |
| gateway | Integer | Is it a gateway: 1 for yes, 0 for no |
| gatewayType | Integer | Gateway type: 1 normal gateway 2 edge gateway |
| typeId | String | product type id |
| typeName | String | Product category name |
| tenantId | String | tenant id |

### Request Example

```TEXT
GET: /v1.0/device/detail/rn01327e6a4a0c7c
```

### Return to Example

```JSON
{
    "reqId": "1963f0a0c7d06221",
    "time": 1682508594,
    "code": 200,
    "message": "success",
    "data": {
        "id": "1650435719915794432",
        "productId": "UPAxVaiQ7monsG",
        "productName": "Lightstrip t",
        "name": "Colorful Light Strip",
        "uuid": "rn01327e6a4a0c7c",
        "protocolType": "1",
        "ip": "175.9.141.162",
        "activationTime": 1682329514000,
        "onlineStatus": 0,
        "status": 1,
        "gateway": 0,
        "gatewayType": null,
        "typeId": "rgb-strip",
        "typeName": "RGB Strip Light",
        "tenantId": "LSHkcfojvV5a0m29"
    }
}
```

## Get device shadow

### Interface Description

Get device shadow data based on module serial number

### Interface address

```TEXT
GET: /v1.0/device/shadow/{uuid}
```

### Request Parameters

| Parameter name | Type | Parameter position | Required | Description |
| :----- | :----- | :------- | :------- | :------- |
| uuid | String | | true | Device UUID |

### Return Parameters

| Parameter Name | Type | Description |
| :------ | :------------- | :------------- |
| data | DeviceDp Array | Device DP data array |
| reqId | String | request id |
| time | Integer | Timestamp |
| code | Integer | Status code |
| message | String | message |

**`Device` Description**

| Parameter Name | Type | Description |
| :----- | :----- | :----------- |
| key | String | property identifier |
| name | String | Property name |
| type | String | dp point value type |
| value | Object | property value |

### Request Example

```TEXT
GET: /v1.0/device/shadow/rn01327e6a4a0c7c
```

### Return to Example

```JSON
{
    "reqId": "1963f0a0c7d06221",
    "time": 1682508594,
    "code": 200,
    "message": "success",
    "data": [
        {
            "key": "",
            "name": "",
            "type": "",
            "value": ""
        },
        {
            "key": "",
            "name": "",
            "type": "",
            "value": 0.1
        }
    ]
}
```

## Device Unbinding

**Interface address**: `/v1.0/device/unbind/{deviceId}`

**Request method**: `DELETE`

**Request data type**: `application/x-www-form-urlencoded`

**Response data type**: `*/*`

**Interface Description**:

**Request Parameters**:

| Parameter name | Parameter description | Request type | Required | Data type | Schema |
| ----------- | ----------- | -------- | -------- | -------- | ------ |
| deviceId | device ID | path | false | string | |
| isCleanData | isCleanData | query | false | boolean | |

**Response Status**:

| Status code | Description | Schema |
| ------ | ---------- | ---------- |
| 200 | OK | ResponseResult object «boolean» |
| 204 | No Content | |
| 401 | Unauthorized | |
| 403 | Forbidden | |

**Response parameters**:

| Parameter name | Parameter description | Type | schema |
| -------- | ---------------- | ------------- | ------------ |
| code | Response status code | integer(int32) | integer(int32) |
| data | response data | boolean | |
| message | response message | string | |
| reqId | request ID | string | |
| time | request timestamp (10 bits) | integer(int64) | integer(int64) |

**Sample response**:

```JSON
{
    "code": 200,
    "data": true,
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

## Get device by id

**Interface address**:`/v1.0/device/getDetailById/{id}`

**Request method**: `GET`

**Request data type**: `application/x-www-form-urlencoded`

**Response data type**: `*/*`

**Interface Description**:

**Request Parameters**:

| Parameter name | Parameter description | Request type | Required | Data type | Schema |
| -------- | -------- | -------- | -------- | -------- | ------ |
| id | deviceId | path | false | string | |

**Response Status**:

| Status code | Description | Schema |
| ------ | ---------- | ------- |
| 200 | OK | ResponseResult object «DeviceDTO object» |
| 401 | Unauthorized | |
| 403 | Forbidden | |
| 404 | Not Found | |

**Response parameters**:

| Parameter name | Parameter description | Type | schema |
| -------------- | ----------- | ---------- | --------------- |
| code | Response status code | integer(int32) | integer(int32) |
| data | response data | DeviceDTO object | DeviceDTO object |
| activationTime | activation time | string | |
| barcode | barcode | string | |
| bindTime | bind time | string | |
| gateway | Is it a gateway: 1 yes 0 no | integer | |
| gatewayType | Gateway type: 1 normal gateway 2 edge gateway | integer | |
| id | device id | string | |
| ip | ip address | string | |
| name | device name | string | |
| networkType | Network type (0=wifi, 1=ble) | integer | |
| onlineStatus | Online status (0 offline, 1 online) | integer | |
| productId | product id | string | |
| productName | product name | string | |
| protocolType | communication protocol | string | |
| status | Device status (0 disabled, 1 enabled) | integer | |
| tenantId | tenant id | string | |
| thirdUuid | third party uuid | string | |
| timeZone | timezone | string | |
| typeId | product type id | string | |
| typeName | Product category name | string | |
| uuid | module serial number | string | |
| message | response message | string | |
| reqId | request ID | string | |
| time | request timestamp (10 bits) | integer(int64) | integer(int64) |

**Sample response**:

```JSON
{
    "code": 200,
    "data": {
        "activationTime": "",
        "barcode": "",
        "bindTime": "",
        "gateway": 0,
        "gatewayType": 0,
        "id": "",
        "ip": "",
        "name": "",
        "networkType": 0,
        "onlineStatus": 0,
        "productId": "",
        "productName": "",
        "protocolType": "",
        "status": 0,
        "tenantId": "",
        "thirdUuid": "",
        "timeZone": "",
        "typeId": "",
        "typeName": "",
        "uuid": ""
    },
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

## Device Binding

**Interface address**:`/platform/v1.0/device/bind`

**Request method**: `POST`

**Request data type**: `application/json`

**Response data type**: `*/*`

**Interface Description**:

**Request Example**:

```JSON
{
    "appUserId": "",
    "assetId": "",
    "deviceUuid": "",
    "firmwareVersion": "",
    "mcuVersion": "",
    "networkType": 0,
    "productType": ""
}
```

**Request Parameters**:

**Request Parameters**:

| Parameter name | Parameter description | Request type | Required | Data type | Schema |
| --------------- | ------------------- | -------- | -------- | --------- | -------------------- |
| deviceBindApiDTO | deviceBindApiDTO | body | true | DeviceBindApiDTO object | DeviceBindApiDTO object |
| appUserId | app user ID | | false | string | |
| assetId | asset ID | | false | string | |
| deviceUuid | device UUID | | false | string | |
| firmwareVersion | firmware version | | false | string | |
| mcuVersion | mcu version | | false | string | |
| networkType | Network mode (0=network, 1=bluetooth) | | false | integer | |
| productType | Product type (rino,ry) | | false | string | |

**Response Status**:

| Status code | Description | Schema |
| ------ | ---------- | -------------- |
| 200 | OK | ResponseResult object «DeviceDTO object» |
| 201 | Created | |
| 401 | Unauthorized | |
| 403 | Forbidden | |
| 404 | Not Found | |

**Response parameters**:

| Parameter name | Parameter description | Type | schema |
| -------------- | ----------------------------- | ---------- | --------------- |
| code | Response status code | integer(int32) | integer(int32) |
| data | response data | DeviceDTO object | DeviceDTO object |
| activationTime | activation time | string | |
| barcode | barcode | string | |
| bindTime | bind time | string | |
| gateway | Is it a gateway: 1 yes 0 no | integer | |
| gatewayType | Gateway type: 1 normal gateway 2 edge gateway | integer | |
| id | device id | string | |
| ip | ip address | string | |
| name | device name | string | |
| networkType | Network type (0=wifi, 1=ble) | integer | |
| onlineStatus | Online status (0 offline, 1 online) | integer | |
| productId | product id | string | |
| productName | product name | string | |
| protocolType | communication protocol | string | |
| status | Device status (0 disabled, 1 enabled) | integer | |
| tenantId | tenant id | string | |
| thirdUuid | third party uuid | string | |
| timeZone | timezone | string | |
| typeId | product type id | string | |
| typeName | Product category name | string | |
| uuid | module serial number | string | |
| message | response message | string | |
| reqId | request ID | string | |
| time | request timestamp (10 bits) | integer(int64) | integer(int64) |

**Sample response**:

```JSON
{
    "code": 200,
    "data": {
        "activationTime": "",
        "barcode": "",
        "bindTime": "",
        "gateway": 0,
        "gatewayType": 0,
        "id": "",
        "ip": "",
        "name": "",
        "networkType": 0,
        "onlineStatus": 0,
        "productId": "",
        "productName": "",
        "protocolType": "",
        "status": 0,
        "tenantId": "",
        "thirdUuid": "",
        "timeZone": "",
        "typeId": "",
        "typeName": "",
        "uuid": ""
    },
    "message": "success",
    "reqId": "4a0ede228f9f6fac",
    "time": 1620641786
}
```

# 3. Message Queue

# Quick Start

The message queue actively pushes various pre-subscribed event data through [**Pulsar** ](https://pulsar.apache.org/) to meet the requirements of message real-time and message persistence.

## Access address

The cloud development platform provides different access addresses for different regions. Please select the access address according to the region where the device is located to shorten the debugging response time.

| Region | Address |
| :----------- | :----------------------------------- |
| Development Data Center | pulsar://devpulsar.rinoiot.com:16651 |
| Test Data Center | pulsar://testpulsar.rinoiot.com:6650 |

## JAVA Example

Import POM coordinates

```XML
<dependency>
    <groupId>com.rinoiot</groupId>
    <artifactId>rinoiot-common-pulsar</artifactId>
    <version>1.0<version>
</dependency>
```

Configuration Instructions

```TEXT
Rino:
  pulsar:
    client:
      #pulsar cluster address
      url: pulsar://testpulsar.rinioot.com:6650
      #pulsar cluster access token
      token: eyJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJyaW5vaW90XzEyMzEifQ.WmKaGvJs2IbLMjA-e09UF0myBZEzX26IJPZM5qVcd2kOkJ O6R19THu8C7PXgDjDFwYd8y4pcINeqVtxN6dxt5I0gC-3YrRcxvfYRpC5mRBPHUNSf7aPmkNdEdhZez2zNTT202Q6KMLyPch5d2 TCh89OlKIyfGF8NTRRN_6c6KNfmFARcQa10T5fsKKSfPxdqZnqCEE7-6S3aKb6GHQlK-VtfRgmCthWtR7xtSabdQktIdjL8_A0 Fbu6wU4YdgLslJVenoB002JGD9pt1E9vQJXV6lTiENJNJ1CsV2Q8ZePzD1qKIHN6EdCH-lXJ9FNTdWvF46sAY9qsXLbjctDpM0w
```

Notes

```JAVA
/**
* Whether to enable the pulsar consumer function in the project startup class
*/
@EnablePulsarConsumer
```

Message subscription implementation

```JAVA
package com.rinioot.business.admin.listener;

import com.rinioot.common.pulsar.service.PulsarConsumerService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;


@Slf4j
@Component
public class TestTopicListener {
    @Autowired(required = false)
    private PulsarConsumerService pulsarConsumerService;

    //Subscribe to topic
    private String topic = "test_topic";

    //Subscription ID is randomly assigned by IoT platform
    private String subscribeId = "1231";

    //The namespace is randomly assigned by the IoT platform
    private String namespace = "randomstr";

    @PostConstruct
    public void listener(){
        if(pulsarConsumerService == null){
            return;
        }
        log.info("Start listening to pulsar topic:{}",topic);
        try {
            pulsarConsumerService.initConsumer(subscribeId,namespace,topic).messageListener((c,msg)->{
                try {
                    log.info(c.getConsumerName() + " received: " + new String(msg.getData()));
                    //Implement specific business code
                    XXXXXXXXXXXXXXXXXXXXXXXX
                        
                    //Message ack confirmation
                    c.acknowledge(msg);
                } catch (Exception e) {
                    c.negativeAcknowledge(msg);
                    log.error("Listening pulsar topic:{}",topic,e);
                }
            }).subscribe();
        } catch (Exception e) {
            log.error("Listening pulsar topic:{}",topic,e);
        }
    }
}
```
