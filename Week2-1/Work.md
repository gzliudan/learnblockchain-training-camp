# 第 2 周第 1 课作业

## 第 1 题：编写一个 Bank 合约

- 通过 Metamask 向 Bank 合约转账 ETH
- 在 Bank 合约记录每个地址转账金额
- 编写 Bank 合约 withdraw 函数，实现提取出所有的 ETH

### 1.1 合约代码：

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.9;

contract Bank {
    event Deposit(address from, uint256 amount, uint256 oldBalance, uint256 newBalance);
    event Withdraw(address to, uint256 amount);

    mapping(address => uint256) public balances;

    constructor() payable {
        balances[msg.sender] = msg.value;
    }

    receive() external payable {
        uint256 oldBalance = balances[msg.sender];
        uint256 newBalance = oldBalance + msg.value;
        balances[msg.sender] = newBalance;

        emit Deposit(msg.sender, msg.value, oldBalance, newBalance);
    }

    function withdraw() public {
        uint256 amount = balances[msg.sender];

        // zero the balance before transfer to prevent re-entrancy attacks
        balances[msg.sender] = 0;

        // payable(msg.sender).transfer(amount);
        (bool success, ) = msg.sender.call{value: amount}(new bytes(0));
        require(success, "withdraw failed");

        emit Withdraw(msg.sender, amount);
    }
}
```

部署地址：https://mumbai.polygonscan.com/address/0x72ccacb91cfe0ff342c5876ec8aa671c862717a3#code

### 1.2 用 MetaMash 向合约转 ETH

转账前在区块链浏览器上查询账号 0xD4CE02705041F04135f1949Bc835c1Fe0885513c 的余额为 0：

![image](https://user-images.githubusercontent.com/7695325/225321668-f74c5088-9fdb-490b-aba4-975c597f4618.png)

向合约 0x72ccacb91cfe0ff342c5876ec8aa671c862717a3 转 1 个 ETH：

![1678886851015](https://user-images.githubusercontent.com/7695325/225322778-f4935508-59ea-48b2-bd24-f5b7c3b9af6a.png)

![1678886926322](https://user-images.githubusercontent.com/7695325/225322981-1861a3d0-2964-436f-9f0e-af8fe3e26bbc.png)

![1678887013938](https://user-images.githubusercontent.com/7695325/225323354-b2ddc574-722d-4cd2-b98c-8c96c97898c5.png)

查看转账对应的交易：

https://mumbai.polygonscan.com/tx/0x498bb9bbc8adcc467f86d15f77e9e0268ff8b926edc8cc4a24da51283cae5d03

![1678887368713](https://user-images.githubusercontent.com/7695325/225324945-c6eecd98-dfe1-43fe-a582-db6db3193455.png)

查看交易发射的事件：

https://mumbai.polygonscan.com/tx/0x498bb9bbc8adcc467f86d15f77e9e0268ff8b926edc8cc4a24da51283cae5d03#eventlog

![1678887459349](https://user-images.githubusercontent.com/7695325/225325386-75d86e98-574d-4f2b-8d67-2776d854c172.png)

在区块链浏览器上再次查询账号 0xD4CE02705041F04135f1949Bc835c1Fe0885513c 的余额，已经更新为 1 个 ETH：

![1678887099802](https://user-images.githubusercontent.com/7695325/225323728-4857f6a7-d12a-40c4-b00e-e9101a89e200.png)

### 1.3 withdraw 函数提现

https://mumbai.polygonscan.com/address/0x72ccacb91cfe0ff342c5876ec8aa671c862717a3#writeContract

![1678887574813](https://user-images.githubusercontent.com/7695325/225325955-bc0ebeae-6903-4b10-8920-5dde5cf4d6d3.png)

点击 `Connect to Web3`，连接到 MetaMask

![1678887729564](https://user-images.githubusercontent.com/7695325/225326669-a369c2e0-5355-4a1d-90d2-4b6e00062b19.png)

再点击 `Write` 按钮：

![1678889707537](https://user-images.githubusercontent.com/7695325/225336083-4ab30814-1f47-4047-ae91-d774686c5e3c.png)

在 MetaMask 里对交易进行签名后

查看交易：

https://mumbai.polygonscan.com/tx/0xdf72e24cfbd48e29bc92db4192ae4ca848d0ca7fd6dc70b288df1daffb2ab8b5

![1678889833466](https://user-images.githubusercontent.com/7695325/225336714-0e871805-a480-4668-9518-53af0ea5dda8.png)

查看交易发射的事件：

https://mumbai.polygonscan.com/tx/0xdf72e24cfbd48e29bc92db4192ae4ca848d0ca7fd6dc70b288df1daffb2ab8b5#eventlog

![1678889899805](https://user-images.githubusercontent.com/7695325/225337067-e1ee9217-d8ce-41ff-95f1-dccec2c0bfb9.png)
