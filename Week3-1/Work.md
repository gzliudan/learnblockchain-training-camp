# 第 3 周第 1 课作业

## 第 1 题：发行一个 ERC20Token (用自己的名字) ，发行 100000 token

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.13;

import { ERC20 } from "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import { ERC20Permit } from "@openzeppelin/contracts/token/ERC20/extensions/draft-ERC20Permit.sol";

contract MyTestToken is ERC20, ERC20Permit {
    constructor() ERC20("My Test Token", "MTT") ERC20Permit("MyToken") {
        // 发行 100000 token
        _mint(msg.sender, 100000 ether);
    }
}
```

## 第 2 题：编写一个金库 Vault 合约:

- 编写 deposit 方法，实现 ERC20 存入 Vault，并记录每个用户存款金额 (approve/transferFrom)
- 编写 withdraw 方法，提取用户自己的存款

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.13;

import { IERC20 } from "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import { IERC20Permit } from "@openzeppelin/contracts/token/ERC20/extensions/draft-IERC20Permit.sol";
import { SafeERC20 } from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

contract Vault {
    using SafeERC20 for IERC20;

    IERC20 public token;
    mapping(address => uint256) public balances;

    event Deposit(address indexed from, uint256 amount, uint256 oldBalance, uint256 newBalance);
    event Withdraw(address indexed to, uint256 amount, uint256 oldBalance, uint256 newBalance);

    error BigAmount(address who, uint256 balance, uint256 amount);
    error UnsupportedToken();

    constructor(address token_) {
        token = IERC20(token_);
    }

    // 编写 deposit 方法，实现 ERC20 存入 Vault，并记录每个用户存款金额 (approve/transferFrom)
    function deposit(uint256 amount, uint256 deadline, uint8 v, bytes32 r, bytes32 s) public {
        IERC20Permit(address(token)).permit(msg.sender, address(this), amount, deadline, v, r, s);

        uint256 oldBalance = token.balanceOf(address(this));
        token.safeTransferFrom(msg.sender, address(this), amount);
        uint256 newBalance = token.balanceOf(address(this));

        if (oldBalance + amount != newBalance) {
            revert UnsupportedToken();
        }

        oldBalance = balances[msg.sender];
        newBalance = oldBalance + amount;
        balances[msg.sender] = newBalance;

        emit Deposit(msg.sender, amount, oldBalance, newBalance);
    }

    // 编写 withdraw 方法，提取用户自己的存款
    function withdraw(uint256 amount) public {
        uint256 oldBalance = balances[msg.sender];
        if (oldBalance < amount) {
            revert BigAmount(msg.sender, oldBalance, amount);
        }

        uint256 newBalance = oldBalance - amount;
        balances[msg.sender] = newBalance;

        token.safeTransfer(msg.sender, amount);

        emit Withdraw(msg.sender, amount, oldBalance, newBalance);
    }
}
```

## 第 3 题：进阶练习:

使用 ERC2612 标准 Token，使用签名的方式 deposit

## 第 4 题：发行一个 ERC721 Token (用自己的名字)

- 铸造一个 NFT，在测试网上发行，在 OpenSea 上查看

### 4.1 代码

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.13;

import { Counters } from "@openzeppelin/contracts/utils/Counters.sol";
import { ERC721 } from "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import { ERC721URIStorage } from "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import { Ownable } from "@openzeppelin/contracts/access/Ownable.sol";

contract MyTestNft is ERC721URIStorage, Ownable {
    using Counters for Counters.Counter;

    Counters.Counter private _nextTokenId;

    constructor() ERC721("My Test NFT", "MTNFT") {
        _nextTokenId.increment();
    }

    function mint(address to, string memory tokenURI) public onlyOwner returns (uint256) {
        uint256 currentTokenId = _nextTokenId.current();
        _nextTokenId.increment();
        _safeMint(to, currentTokenId);
        _setTokenURI(currentTokenId, tokenURI);

        return currentTokenId;
    }

    function nextTokenId() public view returns (uint256) {
        return _nextTokenId.current();
    }
}
```

### 4.2 部署

https://mumbai.polygonscan.com/address/0x9317883a8d279c779735598e720d4f7dc691c0d8#code

### 4.3 铸造

元数据：ipfs://QmWZYAJ2hxBvnicVKwLtWwVyUfZudkdzZScQCNFURbdyac

```json
{
  "name": "X",
  "description": "Letter X",
  "image": "ipfs://QmeRq7pabiJE2n1xU3Y5Mb4TZSX9kQ74x7a3P2Z4PqcMRX",
  "attributes": [
    {
      "trait_type": "NO",
      "value": 24
    },
    {
      "trait_type": "uppercase",
      "value": "X"
    },
    {
      "trait_type": "lowercase",
      "value": "x"
    },
    {
      "trait_type": "previous",
      "value": "W"
    },
    {
      "trait_type": "next",
      "value": "Y"
    }
  ]
}
```

![267bfdba12fab039836bc7f52c93cf9](https://user-images.githubusercontent.com/7695325/227482250-2412ac33-64d2-4bac-9b70-6badb8eb668d.png)

### 4.4 在 OpenSea 上查看

https://testnets.opensea.io/assets/mumbai/0x9317883a8d279c779735598e720d4f7dc691c0d8/1

![1679650662271](https://user-images.githubusercontent.com/7695325/227482866-173ba1d5-0f95-4232-a02c-5f3e0e42db84.png)

## 第 5 题：编写一个合约:使用自己发行的 ERC20Token 来买卖 NFT

- NFT 持有者可上架 NFT (设置价格 多少个 TOKEN 购买 NFT )
- 编写购买 NFT 方法，转入对应的 TOKEN，获取对应的 NFT

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.13;

import { Counters } from "@openzeppelin/contracts/utils/Counters.sol";
import { ERC721 } from "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import { IERC20 } from "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import { Ownable } from "@openzeppelin/contracts/access/Ownable.sol";
import { SafeERC20 } from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

contract TradableNft is ERC721, Ownable {
    using SafeERC20 for IERC20;
    using Counters for Counters.Counter;

    IERC20 public token;
    Counters.Counter private _nextTokenId;

    // tokenId => price
    mapping(uint256 => uint256) prices;

    event List(uint256 indexed tokenId, uint256 price, address indexed owner);
    event Delist(uint256 indexed tokenId, uint256 price, address indexed owner);
    event Buy(uint256 indexed tokenId, uint256 price, address indexed seller, address indexed buyer);

    error InvalidPrice(uint256 tokenId, address owner);
    error NotListed(uint256 tokenId, address caller);
    error NotOwner(uint256 tokenId, address who);

    constructor(address token_) ERC721("Tradable NFT", "TNFT") {
        token = IERC20(token_);
        _nextTokenId.increment();
    }

    function mint(address to) public onlyOwner returns (uint256) {
        uint256 currentTokenId = _nextTokenId.current();
        _nextTokenId.increment();
        _safeMint(to, currentTokenId);

        return currentTokenId;
    }

    function nextTokenId() public view returns (uint256) {
        return _nextTokenId.current();
    }

    // NFT 持有者可上架 NFT (设置价格 多少个 TOKEN 购买 NFT )
    function list(uint256 tokenId, uint256 price) external {
        if (price == 0) {
            revert InvalidPrice(tokenId, msg.sender);
        }

        if (msg.sender != ownerOf(tokenId)) {
            revert NotOwner(tokenId, msg.sender);
        }

        prices[tokenId] = price;

        emit List(tokenId, price, msg.sender);
    }

    // 下架
    function delist(uint256 tokenId) external {
        if (msg.sender != ownerOf(tokenId)) {
            revert NotOwner(tokenId, msg.sender);
        }

        if (prices[tokenId] == 0) {
            revert NotListed(tokenId, msg.sender);
        }

        uint256 price = prices[tokenId];
        delete prices[tokenId];

        emit Delist(tokenId, price, msg.sender);
    }

    // 购买 NFT，转入对应的 TOKEN，获取对应的 NFT
    function buy(uint256 tokenId) external {
        address seller = ownerOf(tokenId);

        uint256 price = prices[tokenId];
        if (price == 0) {
            revert NotListed(tokenId, msg.sender);
        }

        token.safeTransferFrom(msg.sender, seller, price);
        _safeTransfer(seller, msg.sender, tokenId, "");

        delete prices[tokenId];

        emit Buy(tokenId, price, seller, msg.sender);
    }
}
```
