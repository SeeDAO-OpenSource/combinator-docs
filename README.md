# SeeDAO 孵化器 C Combinator 文档

## Combinator API 说明文档

| 路径 | 说明 | 输入 | 输出 |
| --- | --- | --- | --- |
| GET /timestamp | 获取服务器时间戳 | 无 | Unix时间戳，单位是秒。 |
| GET /nonce?account=0x0F34EC76daCa79425Feec7106BADe663DEfC00fa | 获取随机数 | account: 以太坊地址 | nonce: int64整数并且大于等于0。 |
| POST /login |  | account: 帐户，即以太坊地址  time: 签名时间，Unix时间戳（秒）  msg: 消息  nonce: 来自服务器的随机数  sig: 签名 | token：登录令环。 |
| GET /home（认证） | 主页 |  |  |
| GET /project（认证） | 单个项目详情页 | id：项目编号 |  |
| GET /profile（认证） | 个人主页 |  |  |
| POST /proposal（认证） | 提案 |  | success: 是否成功，1成功，0失败 |
| POST /vote （认证） | 投票 |  | success: 是否成功，1成功，0失败 |
| GET /recharge（认证） | 充值Genesis或SEE |  | success: 是否成功，1成功，0失败 |
| GET /withdraw（认证） | 提现Genesis或SEE |  | success: 是否成功，1成功，0失败 |

### 认证接口

接口中带有“（认证）”的表示是认证接口，登录后的用户可见。请求时需要提供登录Token。