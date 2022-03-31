# Combinator 合约说明文档

## 合约部署地址
| 网络     | 版本  | 合约                | 地址                                        |
|---------|-------|--------------------|--------------------------------------------|
| Rinkeby | 0.2.0 | SeeDAO Genesis NFT | 0x13602322Ef5D48681ce73fb42F271B08eB6a245c |
|         |       | SEE                | 0xE2B0F0dA168B26216fDCd330d7105F7979B9325F |
|         |       | Combinator         | 0xe5B79283499424Fba1cEbeC871D11d212E24cEb8 |
| Rinkeby | 0.1.0 | SeeDAO Genesis NFT | 0x3EEBc96aDa2503B12Cb933c0f2263748378B246A |
|         |       | SEE                | 0xf4daB28ABad50D4C2A0183502797CE092Fed7c83 |
|         |       | Combinator         | 0xE12FeB4A323B4F90B9b38075aCC1B794CA4e4F63 |

### 合约接口

| 函数 | 说明 | 输入 | 输出 |
| --- | --- | --- | --- |
| depositGenesis(uint256 tokenId)  | 存入/充值GenesisNFT | uint256 tokenId：Genesis的tokenId | 无 |
| withdrawGenesis(uint256 tokenId) | 取出/提现 GenesisNFT | uint256 tokenId：Genesis的tokenId | 无 |
| depositSee(uint256 amount)  | 存入/充值 SEE 代币 | uint256 amount：SEE的数量 | 无 |
| withdrawSee(uint256 amount) | 取出/提现 SEE 代币。<br/>注意参数是cSEE数量（会被销毁）。 | uint256 amount：cSEE的数量 | 无 |
| genesisBalanceOf(address owner) | 查询用户的Genesis | address owner: 用户地址 | uint256[] tokenIds: 持有的ID，从小到大排列。 |