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
- `type`: 签名类型
  - 0: 对固定文本签名
  - 1: 对组合文本签名(msg+time+nonce)
  - 3: EIP-712标准签名
- `msg`: 原始消息  
  测试期间使用`Welcome to C Combinator!`  
  EIP-712不传该字段(`type = 3`时)
- `time`: 签名时间，Unix时间戳（秒）。
  服务器端会检查该时间，必须在5分钟以内才有效。
- `nonce`: 来自服务器的随机数，见`/nonce`接口
- `sig`: 对以上数据的签名

**返回**
返回的登录令环（`Token`）在`Set-Cookie`头里。

**签名说明**


### 认证接口

接口中带有“（认证）”的表示是认证接口，登录后的用户可见。请求时需要提供登录Token。

#### 当前活动项目 `GET /round/current`

**参数**
无

**返回**
- `id`: Round Id，大于0的整型数字
- `end`: 结束时间，Unix时间戳（单位：秒）
- `projects`: 项目列表
    - `image`: 图片链接
    - `desc`: 摘要描述
    - `founders`: 以太坊地址或ENS域名列表

#### 所有项目 `GET /projects`

**参数**
无

**返回**
- `projects`: 项目列表
    - `image`: 图片链接
    - `desc`: 摘要描述
    - `founders`: 以太坊地址或ENS域名列表

#### 单个项目详情页 `GET /project`

**参数**
- `id`：项目编号

**返回**
- `image`: 图片链接
- `desc`: 摘要描述
- `founders`: 以太坊地址或ENS域名列表
- `twitter`: Twitter帐户
- `attachments`: 附件链接

#### 个人主页 `GET /profile`

#### 提案 `POST /proposal`

**参数**
- `image`: 图片链接
- `desc`: 摘要描述
- `founders`: 以太坊地址或ENS域名列表
- `twitter`: Twitter帐户
- `attachments`: 附件链接

**返回**
- `success`: 是否成功，1成功，0失败

#### 投票 `POST /vote`

**参数**
- `id`: 项目id
- `amount`: 投票额度

**返回**
- `success`: 是否成功，1成功，0失败

#### 充值Genesis或SEE `GET /recharge`

**参数**
- `amount`: 充值金额(`$SEE`的数量)

**返回**
- `success`: 是否成功，1成功，0失败

#### 提现Genesis或SEE`GET /withdraw`

**参数**
- `amount`: 提现金额(`$cSEE`的数量)

**返回**
- `success`: 是否成功，1成功，0失败