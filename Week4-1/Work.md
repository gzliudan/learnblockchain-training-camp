# 第 4 周第 1 课作业

## 第 1 题：编写前端使用之前的 Vault 合约:

- 用户可通过前端进行存款
  - 两个方式: Approve + deposit
  - 最好使用 ERC2612(Permit) 方式更好(permitDeposit)
- 前端显示用户存款金额
- 用户可通过前端提取用户自己的存款 (withdraw)

## 第 2 题：发行一个 ERC721Token

- 使用 ethers.js 解析 ERC721 转账事件
- 加分项: 记录到数据库中，可方便查询用户持有的所有 NFT
