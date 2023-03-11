# 第 1 周第 1 课作业

## 第 1 题：安装 Metamask、并创建好账号

![1678497647549](https://user-images.githubusercontent.com/7695325/224456968-bfabdfb8-16d1-4f2c-a6c6-662c29f9231f.png)

账号：0xD4CE02705041F04135f1949Bc835c1Fe0885513c

## 第 2 题：执行一次转账

0xD4CE02705041F04135f1949Bc835c1Fe0885513c 向 0x85f33e1242d87a875301312bd4ebaee8876517ba 转 0.1 个 ETH，交易信息如下：

![1678497550964](https://user-images.githubusercontent.com/7695325/224456865-54494882-6c94-4951-aa5a-4c919edb9540.png)

https://goerli.etherscan.io/tx/0x8a5a90d72a79939bc40a54e742b2c55e7050c4133670da6554ca9792bfceb058

## 第 3 题：使用 Remix 创建一个 Counter 合约并部署，Counter 合约有一个 add(x) 方法

### 3.1 代码

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

contract Counter {
    uint256 public sum;

    function add(uint256 x) external returns (uint256) {
        uint256 result = sum + x;
        sum = result;
        return result;
    }
}
```

### 3.2 部署

https://goerli.etherscan.io/tx/0x3911ed211dce9c993b39b0fa3d0dd911ae9105426606c40e3d3d39989de2244d

![1678498422634](https://user-images.githubusercontent.com/7695325/224457693-ee6a4c49-598c-4175-b30c-b8358942a1f3.png)

## 3.3 校验合约

地址： 0xa9398e2761857C8E3cAaB7b3ec01Cc8EBbf04764
浏览器： https://goerli.etherscan.io/address/0xa9398e2761857c8e3caab7b3ec01cc8ebbf04764#code

![1678498946509](https://user-images.githubusercontent.com/7695325/224458115-ef24db7e-bf19-433c-8b0c-b8d77286f044.png)

![1678499075147](https://user-images.githubusercontent.com/7695325/224458248-6d981090-f2d5-4bee-9d0b-58b5063e4334.png)

![1678499115132](https://user-images.githubusercontent.com/7695325/224458284-65b7409b-956b-4f86-8abf-92faf13a5fc3.png)
