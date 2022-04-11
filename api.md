# Combinator API 说明文档

### 数据结构

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

## 接口总览

- 开放接口
  - `GET /timestamp`: 获取时间戳
  - `GET /rounds`: 列出活动轮次
  - `GET /rounds/ID`: 获取某个轮次活动详情
  - `GET /projects`: 列出项目
  - `GET /static/{filename}`: 下载文件
  - `GET /nonce`: 获取随机数
  - `POST /login`: 登录
- 认证接口
  - `POST /projects`: 提交一个新项目
  - `POST /cover`: 上传图片
  - `POST /files`: 上传文件
  - `GET /projects/ID`: 获取某个项目详情
  - `GET /votes`: 查看自己的投票
  - `POST /votes`: 投票
  - `GET /deposit`: 充值/存款
  - `GET /withdraw`: 提现/取款
  - `GET /rewards`: 列出本人收益
  - `GET /rewards/ID`: 列出单个项目收益，ID为项目ID
  - `POST /claim`: 获取奖励签名，用于合约交互
- 管理接口
  - `POST /admin/rounds/`: 提交一个新轮次
  - `DELETE /admin/projects/ID`: 删除某个项目

## 开放接口

开放接口无需认证，任何人都可以访问。

### 获取服务器时间戳 `GET /timestamp`

Unix时间戳，单位是秒。

该接口主要用于初步测试服务器连通性，只返回一个整数，方便测试和监控使用。

### 列出项目 `GET /projects`

**参数**
- `round`: 第几轮项目, `current`表示当前轮, 不填表示所有项目。
- `offset`: 偏移量，可不填，默认值0（不偏移）
- `limit`: 返回最大数量，可不填，默认值100

**返回**
- `projects`: 项目列表
    - `image`: 图片链接
    - `desc`: 摘要描述
    - `founders`: 以太坊地址或ENS域名列表
### 列出所有轮次 `GET /rounds/`

**参数**
无

**返回**
- `rounds`: 活动轮次
  - `id`: Round Id，大于0的整型数字
  - `start`: 开始时间，Unix时间戳（单位：秒）
  - `end`: 结束时间，Unix时间戳（单位：秒）

### 列出某个轮次 `GET /rounds/ID`

**参数**
- `ID`: 大于0的整数，或者`current`

**返回**
- `id`: Round Id，大于0的整型数字
- `start`: 开始时间，Unix时间戳（单位：秒）
- `end`: 结束时间，Unix时间戳（单位：秒）

### 获取单个项目详情页 `GET /projects/ID`

**参数**
- `ID`：项目编号

**返回**
- `creator`: 创建者
- `image`: 图片链接
- `desc`: 摘要描述
- `founders`: 以太坊地址或ENS域名列表
- `twitter`: Twitter帐户
- `attachments`: 附件链接
- `votes`: (登录时)投票的票数
- `reward`: 
  - `amount`: 奖励数量, 浮点数, 大于0
  - `signature`: 签名

### 获取Nonce `GET /nonce`

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

### 登录 `POST /login`

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
签名说明详见[签名方案](sign.md)

## 认证接口

登录后的用户可见。请求时需要提供登录Token。

### 提案 `POST /projects`

**参数**
- `image`: 图片链接
- `desc`: 摘要描述
- `founders`: 以太坊地址或ENS域名列表
- `twitter`: Twitter帐户
- `attachments`: 附件链接

**返回**
- `success`: 是否成功，1成功，0失败

### 上传图片: `POST /cover`

上传项目的封面图片。

**参数**

**返回**
- success: 是否成功
- link: 图片链接

**示例**

```bash
curl -X POST http://dev.seedao.cc/api/v1/cover  -H 'Authorization:Bearer xxx' \
  -F "file=@/path/to/local/file.png" \
  -H "Content-Type: multipart/form-data"
```

```json
{
  "state": 200,
  "msg": "ok",
  "data": {
    "link": "http://dev.seedao.cc/api/v1/cover/0x42B81011a367d3C7f4e4B570Fadd51209C93F287/314.png",
    "success": true
  }
}
```

### 上传文件: `POST /files`

用multipart form data上传文件

**参数**
文件名

**返回**
成功和失败的文件列表

**示例**
```bash
curl -X POST http://dev.seedao.cc/api/v1/files \
  -H 'Authorization:Bearer xxxx' \
  -F "upload[]=@/path/to/the/file1" \
  -F "upload[]=@/path/to/the/file2" \
  -H "Content-Type: multipart/form-data"
```

```json
{
  "state": 200,
  "msg": "ok",
  "data": {
    "failed": [],
    "success": [
      "http://dev.seedao.cc/api/v1/static/0x542A9db710a48b0A952483F256c556E24B000a13/file1",
      "http://dev.seedao.cc/api/v1/static/0x542A9db710a48b0A952483F256c556E24B000a13/file2"
    ]
  }
}
```
### 查看自己的投票 `GET /votes`

**参数**
- `project`: 项目id, 可不填（表示所有投票）
- `round`: 投票所属活动轮次，1,2,3或者current，可不填（表示所有轮次）
- `offset`: 偏移量，可不填，默认值0（不偏移）
- `limit`: 返回最大数量，可不填，默认值100

**返回**
- `project`: 项目id
- `votes`: 投票数量

### 投票 `POST /votes`

**参数**
- `project`: 项目id
- `votes`: 投票数量

**返回**
- `success`: 是否成功，1成功，0失败

### 充值Genesis或SEE `GET /deposit`

**参数**
- `amount`: 充值金额(`$SEE`的数量)

**返回**
- `success`: 是否成功，1成功，0失败

### 提现Genesis或SEE `GET /withdraw`

**参数**
- `amount`: 提现金额(`$cSEE`的数量)

**返回**
- `success`: 是否成功，1成功，0失败

### 获取奖励签名，用于合约交互 `POST /claim`

**参数**
- `projects`: 项目ID列表

**返回**
- `account`: 帐户地址
- `projects`: 项目ID列表
- `amounts`: 奖励数量列表
- `signature`: 签名

### 提交一个新轮次 `POST /admin/rounds/`

**参数**
- `ID`: 大于0的整数，或者`current`
- `start`: 开始时间，Unix时间戳（单位：秒）
- `end`: 结束时间，Unix时间戳（单位：秒）

**返回**
- `id`: Round Id，大于0的整型数字
- `start`: 开始时间，Unix时间戳（单位：秒）
- `end`: 结束时间，Unix时间戳（单位：秒）

### 删除某个项目 `DELETE /admin/projects/ID`

**参数**
- `ID`: 项目ID

**返回**
- `success`: 是否成功，1成功，0失败