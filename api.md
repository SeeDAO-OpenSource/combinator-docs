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
  - `GET /partners`: 获取合作伙伴列表
- 认证接口
  - `POST /projects`: 提交一个新项目
  - `POST /cover`: 上传图片
  - `POST /files`: 上传文件
  - `GET /projects/ID`: 获取某个项目详情
  - `GET /votes`: 查看自己的投票
  - `POST /votes`: 投票
  - `GET /myVoting`: 查看自己投票的项目
  - `GET /balance`: 查询余额
  - ~~`GET /rewards`: 列出本人收益~~
  - ~~`GET /rewards/ID`: 列出单个项目收益，ID为项目~~ID
  - ~~`GET /roi`: 列出ROI~~
  - `GET /whitelist`: 验证是否在白名单额度
  - `GET /profile`: 获得用户`Profile`，目前只有头像ID
  - `POST /profile`: 设置用户`Profile`，目前仅支持设置头像ID
  - `GET /favorites`: 获取用户喜欢的项目
  - `POST /favorite`: 喜欢(点赞)一个项目
  - `POST /claim`: Claim 投票
- 管理接口
  - `POST /admin/rounds/`: 提交一个新轮次
  - `DELETE /admin/projects/ID`: 删除某个项目
  - `GET /admin/pending`: 获取未审核项目列表
  - `PUT /admin/project/ID`: 审核某个项目
  - `POST /admin/project/ID`: 修改某个项目的资料
  - `GET /admin/export/project/pending`: 导出未审核项目
  - `POST /admin/partners`: 设置合作伙伴列表(全量)

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
    - `title`: 标题
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
- `ID`: 大于0的整数，或者`current`/`next`

**返回**
- `id`: Round Id，大于0的整型数字
- `desc`: 本轮活动的描述
- `start`: 开始时间，Unix时间戳（单位：秒）
- `end`: 结束时间，Unix时间戳（单位：秒）

### 获取单个项目详情页 `GET /projects/ID`

**参数**

- `ID`：项目编号

**返回**
- `creator`: 创建者
- `image`: 图片链接
- `title`: 标题
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
- `title`: 标题
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

### 查看自己投票的项目: `GET /myVoting`

**参数**

- `round`: 第几轮项目, `current`表示当前轮, `past`表示往期项目, 不填表示所有项目。
- `offset`: 偏移量，可不填，默认值0（不偏移）
- `limit`: 返回最大数量，可不填，默认值100

**返回**

- `projects`: 项目列表
    - `image`: 图片链接
    - `title`: 标题
    - `desc`: 摘要描述
    - `founders`: 以太坊地址或ENS域名列表
    - `myVotes`: 我投的票数

### 查询余额 `GET /balance`

**参数**

无

**返回**

- `account`: 帐户地址
- `total`: 本轮总票数，一般为`6`。
- `balance`: 余额，即还剩几票可以投。

### 验证是否在白名单额度 `GET /whitelist`

**参数**

无

**返回**

- `account`: 帐户
- `quota`: 额度，`1`表示还能新建一次项目，`0`表示无法新建项目

**示例**

请求时需要附上登录token
```bash
curl -s 'http://localhost:8080/whitelist' -H 'Authorization:Bearer ${JWT TOKEN HERE}'
```

返回的结果如下
```json
{
  "state": 200,
  "msg": "ok",
  "data": {
    "account": "0x542A9db710a48b0A952483F256c556E24B000a13",
    "quota": 1
  }
}
```

#### 获得用户`Profile` `GET /profile`

目前有头像ID和是否可以claim票数。
claimable如果是true，表示需要弹窗，让用户claim，点击后后端会给该用户派发6张票。

**参数**

- 无

**返回**
- `avatarId`: 头像的ID编号，指的是SGN编号，一般是`1`到`10000`之间的整数，如1,2,3
- `claimable`: 是否可以claim

**示例**

```bash
curl -s 'http://localhost:8080/profile' -H 'Authorization:Bearer ${JWT TOKEN HERE}'
```

没有设置的用户，返回空结果
```json
{
  "state": 200,
  "msg": "ok",
  "data": {
    "account": "",
    "avatarId": 0,
    "claimable": true
  }
}
```

设置过头像的用户，返回用户账户和头像的ID
```json
{
  "state": 200,
  "msg": "ok",
  "data": {
    "account": "0x542A9db710a48b0A952483F256c556E24B000a13",
    "avatarId": 2,
    "claimable": true
  }
}
```

#### 设置用户`Profile` `POST /profile`

目前仅支持设置头像ID

**参数**
- `avatarId`: 头像的ID编号，指的是SGN编号

**返回**
- `success`: 成功与否

**示例**

```bash
curl -s 'http://localhost:8080/profile' \
    -H 'Authorization:Bearer ${JWT TOKEN HERE}' \
    -H 'Content-Type:application/json' \
    -d '{"avatarId":2}'
```

```json
{
  "state": 200,
  "msg": "ok",
  "data": {
    "success": true
  }
}
```

#### 获取用户喜欢的项目 `GET /favorites`

**参数**

无

**返回**

- `projects`: 项目列表（仅包含部分字段）
    - `id`: 项目ID
    - `title`: 标题
    - `image`: 图片链接
    - `desc`: 摘要描述

**示例**

```bash
curl -s 'http://localhost:8080/favorites' \
    -H 'Authorization:Bearer ${JWT TOKEN HERE}'
```

```json
{
  "state": 200,
  "msg": "ok",
  "data": [
    {
      "id": "629b569d8880b5ef064dd9cc",
      "title": "Project Title",
      "image": "图片链接",
      "desc": "这里是被截断了的描述"
    }
  ]
}
```

#### 喜欢(点赞)一个项目 `POST /favorite`

用户喜欢/点赞/收藏一个项目，该项目会在其个人页面的专门tab页内展示。

**参数**
- `projectId`: 项目ID

**返回**
- `success`: 成功与否

**示例**

```bash
curl -s 'http://localhost:8080/favorite' \
    -H 'Authorization:Bearer ${JWT TOKEN HERE}' \
    -H 'Content-Type:application/json' -d '{"projectId":"629b569d8880b5ef064dd9cc"}'
```

```json
{
  "state": 200,
  "msg": "ok",
  "data": {
    "success": true
  }
}
```

#### `POST /claim`: Claim 投票

根据快照，每个 SGN 持有人有 6 票可以随意支配，Claim 即主动领取投票权。

**参数**
- `round`: 活动轮次

**返回**
- `success`: 成功与否

**示例**

```bash
curl -s 'http://localhost:8080/claim' \
    -H 'Authorization:Bearer ${JWT TOKEN HERE}' \
    -H 'Content-Type:application/json' -d '{"round": 2}'
```

```json
{
  "state": 200,
  "msg": "ok",
  "data": {
    "success": true
  }
}
```

### 提交一个新轮次 `POST /admin/rounds/`

**参数**

- `ID`: 大于0的整数，或者`current`
- `start`: 开始时间，Unix时间戳（单位：秒）
  - 指的是该轮活动总体开始时间，一般和报名阶段开始时间相同。
- `end`: 结束时间，Unix时间戳（单位：秒）
  - 指的是该轮活动总体结束时间，活动的总起止时间要包含各个阶段的时间。
- `phaseSubmit`: 报名阶段
  - `start`: 开始时间，Unix时间戳（单位：秒）
  - `end`: 结束时间，Unix时间戳（单位：秒）
- `phaseWorkshop`: 工作坊阶段
  - `start`: 开始时间，Unix时间戳（单位：秒）
  - `end`: 结束时间，Unix时间戳（单位：秒）
- `phaseIdle`: 闲置期
  - `start`: 开始时间，Unix时间戳（单位：秒）
  - `end`: 结束时间，Unix时间戳（单位：秒）
- `phaseVote`: 投票阶段
  - `start`: 开始时间，Unix时间戳（单位：秒）
  - `end`: 结束时间，Unix时间戳（单位：秒）
- `phaseCount`: 唱票阶段
  - `start`: 开始时间，Unix时间戳（单位：秒）
  - `end`: 结束时间，Unix时间戳（单位：秒）

**返回**

- `success`: 是否成功

**示例**

```bash
curl 'http://localhost:8080/admin/rounds' -H 'Authorization:Bearer ${JWT TOKEN HERE}' -H 'Content-Type:application/json' -d '{
      "id" : 2,
      "start" : 1673319600,
      "end" : 1679673599,
      "phaseSubmit" : {
            "start" : 1673319600,
            "end" : 1676476800
      },
      "phaseWorkshop" : {
            "start" : 1676476800,
            "end" : 1678895999
      },
      "phaseCount" : {
            "start" : 1679587200,
            "end" : 1679673599
      },
      "phaseIdle" : {
            "start" : 1678896000,
            "end" : 1678982399
      },
      "phaseVote" : {
            "start" : 1678982400,
            "end" : 1679587199
      }
}'
```

### 删除某个项目 `DELETE /admin/projects/ID`

**参数**

- `ID`: 项目ID

**返回**

- `success`: 是否成功，1成功，0失败


### 获取未审核项目列表 `GET /admin/pending`

**参数**

无

**返回**

**示例**

### 审核某个项目 `PUT /admin/project/ID`

**参数**

**返回**

- `success`: 是否成功

**示例**

### 修改某个项目的资料 `POST /admin/project/ID`

**参数**

- `image`: 图片链接
- `title`: 标题
- `desc`: 摘要描述
- `founders`: 以太坊地址或 ENS 域名列表
- `email`: Email 地址
- `attachments`: 附件链接

**返回**

- `success`: 是否成功

**示例**

### 导出未审核项目 `GET /admin/export/project/pending`

**参数**

无

**返回**

csv文件

**示例**

无

### 获取合作伙伴列表 `GET /partners`

**参数**

无

**返回**

- `partners`: 合作伙伴列表
  - `name`: 名称
  - `url`: 官网地址
  - `logo`: Logo图片地址

**示例**

```bash
curl 'http://localhost:8080/partners' -H 'Authorization:Bearer ${JWT TOKEN HERE}'
```

```json
{
  "state": 200,
  "msg": "ok",
  "data": [
    {
      "name": "Awesome Project 1",
      "url": "https://example1.com",
      "logo": "https://example1.com/logo.png"
    },
    {
      "name": "Awesome Project 2",
      "url": "https://example2.com",
      "logo": "https://example2.com/logo.png"
    }
  ]
}
```

### 设置合作伙伴列表 `POST /admin/partners`

**参数**

- `partners`: 合作伙伴列表
  - `name`: 名称
  - `url`: 官网地址
  - `logo`: Logo图片地址

**返回**

- `success`: 是否成功

**示例**

```bash
curl 'http://localhost:8080/admin/partners' -H 'Authorization:Bearer ${JWT TOKEN HERE}' -H 'Content-Type:application/json' -d '[
	{
		"name": "Awesome Project 1",
		"url": "https://example1.com",
		"logo": "https://example1.com/logo.png"
	},
	{
		"name": "Awesome Project 2",
		"url": "https://example2.com",
		"logo": "https://example2.com/logo.png"
	}
]'
```

```json
{
  "state": 200,
  "msg": "ok",
  "data": {
    "success": true
  }
}
```