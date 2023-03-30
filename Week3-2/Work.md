# 第 3 周第 2 课作业

## 第 1 题：部署一个可升级的 ERC20 Token

- 第一版本
- 第二版本，加入方法: function transferWithCallback(address recipient, uint256amount) external returns (bool)

### 1.1 第一版

#### 1.1.1 智能合约 contracts/TestTokenV1.sol：

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.13;

// ==================== External Imports ====================

import { ERC20Upgradeable } from "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";
import { Initializable } from "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

contract TestTokenV1 is ERC20Upgradeable {
    function initialize() public initializer {
        __ERC20_init("Test Token", "TTK");
        _mint(msg.sender, 1000 ether);
    }

    function version() public pure returns (string memory) {
        return "v1.0";
    }

    function mint(address to, uint256 amount) external {
        _mint(to, amount);
    }
}
```

#### 1.1.2 部署和校验脚本 scripts/deployTestTokenV1.js：

```javascript
/* global run */

const { ethers, upgrades } = require("hardhat");
const { getImplementationAddress } = require("@openzeppelin/upgrades-core");

async function main() {
  console.log("Deploy TestTokenV1 ...");
  const TestTokenV1 = await ethers.getContractFactory("TestTokenV1");
  const proxy = await upgrades.deployProxy(TestTokenV1);
  await proxy.deployed();
  console.log(`proxy = ${proxy.address}`);

  const implementationAddress = await getImplementationAddress(
    ethers.provider,
    proxy.address
  );
  console.log(`implementation = ${implementationAddress}\n`);

  // Wait for Etherscan to catch up (5 blocks), verify, and publish source
  await ethers.provider.waitForTransaction(proxy.deployTransaction.hash, 5);
  await run("verify:verify", { address: proxy.address });
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

#### 1.1.3 部署和校验

执行命令：

```shell
npx hardhat run --network mumbai ./scripts/deployTestTokenV1.js
```

执行结果：

```text
Deploy TestTokenV1 ...
proxy = 0x8Cb088fd4e81DF79672f23c8795371B209eb9b2e
implementation = 0xc3A59A031F8cD157A97858081020925e361fEE13

Verifying implementation: 0xc3A59A031F8cD157A97858081020925e361fEE13
The contract 0xc3A59A031F8cD157A97858081020925e361fEE13 has already been verified
Verifying proxy: 0x8Cb088fd4e81DF79672f23c8795371B209eb9b2e
Contract at 0x8Cb088fd4e81DF79672f23c8795371B209eb9b2e already verified.
Linking proxy 0x8Cb088fd4e81DF79672f23c8795371B209eb9b2e with implementation
Successfully linked proxy to implementation.
Verifying proxy admin: 0x6d77BF667D6f692F79B3D271f8C2f8b4a5f0AaD0
Contract at 0x6d77BF667D6f692F79B3D271f8C2f8b4a5f0AaD0 already verified.

Proxy fully verified.
```

#### 1.1.4 验证

https://mumbai.polygonscan.com/address/0x8cb088fd4e81df79672f23c8795371b209eb9b2e#readProxyContract

![1679691686727](https://user-images.githubusercontent.com/7695325/227639791-723e85e2-02e8-4dc0-b2c1-94d269363227.png)

https://mumbai.polygonscan.com/address/0xc3a59a031f8cd157a97858081020925e361fee13#readContract

![1679692202846](https://user-images.githubusercontent.com/7695325/227642280-d1b5ea7a-6552-45f0-b12a-254e9a04c7ae.png)

### 1.2 第二版

#### 1.2.1 智能合约 contracts/TestTokenV2.sol：

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.13;

import { AddressUpgradeable } from "@openzeppelin/contracts-upgradeable/utils/AddressUpgradeable.sol";
import { ERC20Upgradeable } from "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";

interface TokenRecipient {
    function tokensReceived(address sender, uint amount) external returns (bool);
}

contract TestTokenV2 is ERC20Upgradeable {
    using AddressUpgradeable for address;

    function initialize() public initializer {
        __ERC20_init("Test Token", "TTK");
        _mint(msg.sender, 1000 ether);
    }

    function version() public pure returns (string memory) {
        return "v2.0";
    }

    function mint(address to, uint256 amount) external {
        _mint(to, amount);
    }

    function transferWithCallback(address recipient, uint256 amount) external returns (bool) {
        _transfer(msg.sender, recipient, amount);

        if (recipient.isContract()) {
            bool rv = TokenRecipient(recipient).tokensReceived(msg.sender, amount);
            require(rv, "No tokensReceived");
        }

        return true;
    }
}
```

#### 1.2.2 部署和校验脚本 scripts/deployTestTokenV2.js：

```javascript
/* global run */

const { ethers, upgrades } = require("hardhat");
const { getImplementationAddress } = require("@openzeppelin/upgrades-core");

const proxyAddress = "0x8Cb088fd4e81DF79672f23c8795371B209eb9b2e";

async function main() {
  console.log(`Upgrade TestTokenV1 to TestTokenV2 ...`);
  console.log(`proxy = ${proxyAddress}`);
  const TestTokenV2 = await ethers.getContractFactory("TestTokenV2");
  const proxy = await upgrades.upgradeProxy(proxyAddress, TestTokenV2);
  await proxy.deployed();
  console.log(`Upgrade OK`);

  const implementationAddress = await getImplementationAddress(
    ethers.provider,
    proxy.address
  );
  console.log(`implementation address = ${implementationAddress}\n`);

  // Wait for Etherscan to catch up (5 blocks), verify, and publish source
  await ethers.provider.waitForTransaction(proxy.deployTransaction.hash, 5);
  await run("verify:verify", { address: implementationAddress });
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

#### 1.2.3 部署和校验

执行命令：

```shell
npx hardhat run --network mumbai ./scripts/deployTestTokenV2.js
```

执行结果：

```text
Upgrade TestTokenV1 to TestTokenV2 ...
proxy = 0x8Cb088fd4e81DF79672f23c8795371B209eb9b2e
Upgrade OK
implementation address = 0xDDd40CB1b1229F3e944DCbBD2542b6a7d79E7DDC

The contract 0xDDd40CB1b1229F3e944DCbBD2542b6a7d79E7DDC has already been verified
```

#### 1.2.4 验证

https://mumbai.polygonscan.com/address/0x8cb088fd4e81df79672f23c8795371b209eb9b2e#readProxyContract

![1679692007081](https://user-images.githubusercontent.com/7695325/227641544-cb491b37-3047-436d-9e2f-3858b76ea83a.png)

https://mumbai.polygonscan.com/address/0xddd40cb1b1229f3e944dcbbd2542b6a7d79e7ddc#writeContract

![1679692074391](https://user-images.githubusercontent.com/7695325/227641839-8e5fadae-3eff-4f18-a530-2de91516501f.png)
