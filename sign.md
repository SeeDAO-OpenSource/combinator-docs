# 签名方案

由于签名和验签过程相对复杂，我们使用`type`从简单到复杂逐步调试。

各类型签名含义如下
- `0`: 仅对固定文本“Hello World!”签名，联调专用。
- `1`: 对msg, time, nonce组合签名
- `2`: EIP-712标准签名

## 方案0格式说明

只对固定文本签名，调试用，服务器端只检查签名帐户，不检查原始文本内容。

**示例**

```bash
 ✗ curl -v '/login' -H 'Content-Type:application/json' -d '{
    "type": 0,
    "time": 0,
    "msg": "Hello World!",
    "account": "0x542A9db710a48b0A952483F256c556E24B000a13",
    "nonce": 0,
    "sig": "0xb31ecfe6932963dd23d1984bdef12feace1cabe15f070ddd56830145ebc16ba0704d68d1e314b64c222ef7204f3ca886c583c739fed5f21c3984bff85aa323251c"
}'
*   Trying ::1:8080...
* Connected to localhost (::1) port 8080 (#0)
> POST /login HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.77.0
> Accept: */*
> Content-Type:application/json
> Content-Length: 266
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Content-Type: application/json; charset=utf-8
< Set-Cookie: jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhY2NvdW50IjoiMHg1NDJBOWRiNzEwYTQ4YjBBOTUyNDgzRjI1NmM1NTZFMjRCMDAwYTEzIiwiZXhwIjoxNjQ4NjA0Mzk4LCJvcmlnX2lhdCI6MTY0Nzk5OTU5OH0.ru7o2JsxDiS-mfttTsGOwp1XDfsrfHdn2GGuUT3hXm0; Path=/; Max-Age=604800
< Date: Wed, 23 Mar 2022 01:39:58 GMT
< Content-Length: 51
<
* Connection #0 to host localhost left intact
{"state":200,"msg":"ok","data":{"IV":0,"result":1}}
```

## 方案1格式说明

`msg`, `time`, `nonce`三个参数按如下格式组合
```
msg:$(msg)
time:$(time)
nonce:$(nonce)
```

示例
```
Message: Hello World!
Timestamp: 1
Nonce: 0
```

## 方案3格式说明

- `Message`
  - `address account`: 用户的以太坊地址
  - `int64 timestamp`: Unix时间戳，单位是秒
  - `int64 nonce`: 来自服务器端分配的随机数
- `EIP712Domain`
  - `string name`: 固定字符串"C Combinator"
  - `string version`: 固定字符串"3"
  - `uint256 chainId`: 所在链的chainId
  - `bytes32 salt`: 盐

```javascript
{
    types:
    {
        EIP712Domain: [
        { name: "name", type: "string" },
        { name: "version", type: "string" },
        { name: "chainId", type: "uint256" },
        { name: "salt", type: "byte32" }
        ],
        Combinator: [
        { name: "account", type: "address" },
        { name: "timestamp", type: "int64" },
        { name: "nonce", type: "int64" }
        ]
    },
    primaryType: "Combinator",
    domain: {
        name: "C Combinator",
        version: "3",
        chainId: netId,
        salt: "longlivebitcoin"
    },
    message: {
        account: signer,
        timestamp: 1,
        nonce: 0
    }
}
```

示例
```bash
 curl -v '/login' -H 'Content-Type:application/json' -d '{
    "type": 3,
    "time": 1,
    "account": "0x9A8FdD9B77Bc026F44A3AaABeA088df2d8239F05",
    "nonce": 0,
    "sig": "0x13e88a5fb3c5daf17c4b9539019cf2047af50d9d5de4fbd2bba2246c60ca9a761f8a2990caa32536ef0d9c707ff2536de89704684d756c5b485032facb666d3b1c"
}'
*   Trying ::1:8080...
* Connected to localhost (::1) port 8080 (#0)
> POST /login HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.77.0
> Accept: */*
> Content-Type:application/json
> Content-Length: 254
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Content-Type: application/json; charset=utf-8
< Set-Cookie: jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhY2NvdW50IjoiMHg5QThGZEQ5Qjc3QmMwMjZGNDRBM0FhQUJlQTA4OGRmMmQ4MjM5RjA1IiwiZXhwIjoxNjQ4NzgwNDIyLCJvcmlnX2lhdCI6MTY0ODE3NTYyMn0.hGUIUkP6Di5TvZsxBu5dLxe9Ge8OO6Sns4KDUhtCnP0; Path=/; Max-Age=604800
< Date: Fri, 25 Mar 2022 02:33:42 GMT
< Content-Length: 51
<
* Connection #0 to host localhost left intact
{"state":200,"msg":"ok","data":{"IV":0,"result":1}}
```