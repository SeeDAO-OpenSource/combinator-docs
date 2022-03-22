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

| 路径 | 说明 | 输入 | 输出 |
| --- | --- | --- | --- |
| GET /timestamp | 获取服务器时间戳 | 无 | Unix时间戳，单位是秒。 |
| GET /nonce?account={address}} | 获取随机数 | account: 以太坊地址 | nonce: int64整数并且大于等于0。 |
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