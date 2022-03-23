# SeeDAO 孵化器 C Combinator 文档

## 合约部署地址

-  测试网 Rinkeby
    - SeeDAO Genesis NFT
      - v0.1.0 0x3EEBc96aDa2503B12Cb933c0f2263748378B246A
    - SEE
      - v0.1.0 0xf4daB28ABad50D4C2A0183502797CE092Fed7c83
    - Combinator
      - v0.1.0 0xE12FeB4A323B4F90B9b38075aCC1B794CA4e4F63

### 合约接口

| 函数 | 说明 | 输入 | 输出 |
| --- | --- | --- | --- |
| depositGenesis(uint256 tokenId) | 存入/充值GenesisNFT | uint256 tokenId：Genesis的tokenId | 无 |
| withdrawGenesis(uint256 tokenId) | 取出/提现 GenesisNFT | uint256 tokenId：Genesis的tokenId | 无 |
| depositSee(uint256 amount) | 存入/充值 SEE 代币 | uint256 amount：SEE的数量 | 无 |
| withdrawSee(uint256 amount) | 取出/提现 SEE 代币。  注意参数是cSEE数量（会被销毁）。 | uint256 amount：cSEE的数量 | 无 |

## Combinator API 说明文档

#### 数据结构

API返回的数据均遵守以下数据结构
- `state`: 状态。一般都等于HTTP状态码。
- `msg`: 正确返回的时候是`ok`；遇到错误的时候是错误提示信息。
- `data`: 接口自定义的返回数据，会在下面各个接口中详述。

**举例**
```bash
 ✗ curl -s '/nonce?account=0x4B3fB561b1a4BfB6532A5911DcFf2B5a510c1142' | jq .
{
  "state": 200,
  "msg": "ok",
  "data": {
    "account": "0x4B3fB561b1a4BfB6532A5911DcFf2B5a510c1142",
    "nonce": 2139605246700315600
  }
}
```

### 开放接口

开放接口无需认证，任何人都可以访问。

#### 获取服务器时间戳 `GET /timestamp`

Unix时间戳，单位是秒。

该接口主要用于初步测试服务器连通性，只返回一个整数，方便测试和监控使用。

#### 获取Nonce `GET /nonce`

**参数**
- `account`: 帐户，即以太坊地址

**返回**
- `account`: 帐户，即以太坊地址
- `nonce`: 随机数，64位整数且大于等于0，登录用

**举例**

```bash
curl '/nonce?account=0x4B3fB561b1a4BfB6532A5911DcFf2B5a510c1142'
```

```json
{
  "state": 200,
  "msg": "ok",
  "data": {
    "account": "0x4B3fB561b1a4BfB6532A5911DcFf2B5a510c1142",
    "nonce": 2139605246700315600
  }
}
```

#### 登录 `POST /login`

**参数**
- `account`: 以太坊帐户(地址)，如`0x542A9db710a48b0A952483F256c556E24B000a13`
- `msg`: 原始消息
  测试期间使用`Welcome to C Combinator!`
- `time`: 签名时间，Unix时间戳（秒）。
  服务器端会检查该时间，必须在5分钟以内才有效。
- `nonce`: 来自服务器的随机数，见`/nonce`接口
- `sig`: 对以上数据的签名

**返回**
返回的登录令环（`Token`）在`Set-Cookie`头里。

**签名说明**

将`account`、`msg`、`time`、`nonce`参数使用`abi.encode`组合在一起后，用`keccak256`进行哈希编码，对该hash进行签名。（联调期间暂不实现，仅用"Hello World"替代。）

**举例**

```bash
 ✗ curl -v '/login' -H 'Content-Type:application/json' -d '{
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

### 认证接口

接口中带有“（认证）”的表示是认证接口，登录后的用户可见。请求时需要提供登录Token。

#### 主页 `GET /home`

#### 单个项目详情页 `GET /project`

**参数**
- `id`：项目编号

#### 个人主页 `GET /profile`

#### 提案 `POST /proposal`

**返回**
- `success`: 是否成功，1成功，0失败

#### 投票 `POST /vote`

**返回**
- `success`: 是否成功，1成功，0失败

#### 充值Genesis或SEE `GET /recharge`

**返回**
- `success`: 是否成功，1成功，0失败

#### 提现Genesis或SEE`GET /withdraw`

**返回**
- `success`: 是否成功，1成功，0失败