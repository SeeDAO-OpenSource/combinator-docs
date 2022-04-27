# Combinator 合约说明文档

## 合约部署地址
| 网络     | 版本  | 合约                | 地址                                        |
|---------|-------|--------------------|--------------------------------------------|
| Rinkeby | 0.4.0 | SeeDAO Genesis NFT | 0xEFbe15B6986e10092008Ec6F339E967dAD460a35 |
|         |       | SEE                | 0xe11C7eA31B92B6e0eB86154786DD8D302A9D5b82 |
|         |       | Combinator         | 0x790c7AA995B4017A0197F959fF6E368DF8045C66 |
| Rinkeby | 0.3.0 | SeeDAO Genesis NFT | 0x0E57257f8527e62ef861f0677DcCa485BFFC95BF |
|         |       | SEE                | 0xDE3C7dC725aac3719B3076dCd1e949777889A22E |
|         |       | Combinator         | 0xf00c0fD0e0427Fa57C77793B33fAcB8c092D064C |
| Rinkeby | 0.2.0 | SeeDAO Genesis NFT | 0x13602322Ef5D48681ce73fb42F271B08eB6a245c |
|         |       | SEE                | 0xE2B0F0dA168B26216fDCd330d7105F7979B9325F |
|         |       | Combinator         | 0xe5B79283499424Fba1cEbeC871D11d212E24cEb8 |
| Rinkeby | 0.1.0 | SeeDAO Genesis NFT | 0x3EEBc96aDa2503B12Cb933c0f2263748378B246A |
|         |       | SEE                | 0xf4daB28ABad50D4C2A0183502797CE092Fed7c83 |
|         |       | Combinator         | 0xE12FeB4A323B4F90B9b38075aCC1B794CA4e4F63 |

### 合约接口

| 函数 | 说明 | 输入 | 输出 |
| --- | --- | --- | --- |
| depositGenesis(uint256 tokenId)  | 存入/充值GenesisNFT (需要提前approve) | uint256 tokenId：Genesis的tokenId | 无 |
| withdrawGenesis(uint256 tokenId) | 取出/提现 GenesisNFT | uint256 tokenId：Genesis的tokenId | 无 |
| depositSee(uint256 amount)  | 存入/充值 SEE 代币 (需要提前approve) | uint256 amount：SEE的数量 | 无 |
| withdrawSee(uint256 amount) | 取出/提现 SEE 代币。<br/>注意参数是cSEE数量（会被销毁）。 | uint256 amount：cSEE的数量 | 无 |
| genesisBalanceOf(address owner) | 查询用户的Genesis | address owner: 用户地址 | uint256[] tokenIds: 持有的ID，从小到大排列。 |
| genesisStaked(address owner) | 查询用户已经抵押的Genesis | address owner: 用户地址 | uint256[] tokenIds: 抵押过的ID，从小到大排列。 |
| verify(address to, string calldata id, uint256 amount, bytes32 ticket, uint256 timestamp, bytes memory signature) | 验证签名 | `address to`: 帐户地址，该地址也是claim时候使用的地址<br/>`string calldata id`: 项目id<br/>`uint256 amount`: 奖励数量<br/>`bytes32 ticket`: 票价，防止重复领取<br/>`uint256 timestamp`: Unix时间戳，单位是秒<br/>`bytes memory signature`: 签名 | true: 匹配<br/> false：不匹配 |

#### verify
验证签名是否是管理员签发，是返回true，不是返回false。
```solidity
function verify(
        address to,
        string calldata id,
        uint256 amount,
        bytes32 ticket,
        uint256 timestamp,
        bytes memory signature
    ) public view returns (bool)
```

- `address to`: 帐户地址，该地址也是claim时候使用的地址
- `string calldata id`: 项目id
- `uint256 amount`: 奖励数量
- `bytes32 ticket`: 票价，防止重复领取
- `uint256 timestamp`: Unix时间戳，单位是秒
- `bytes memory signature`: 签名

#### claim

批量claim，数组中的元素一一对应。

```solidity
function claim(
        string[] calldata projects,
        uint256[] calldata amounts,
        bytes32[] calldata tickets,
        uint256[] calldata timestamps,
        bytes[] calldata signatures
    ) external
```

- `string[] calldata projects`: 项目id
- `uint256[] calldata amounts`: 奖励数量
- `bytes32[] calldata tickets`: 票据
- `uint256[] calldata timestamps`: Unix时间戳，单位是秒
- `bytes[] calldata signatures`: 签名