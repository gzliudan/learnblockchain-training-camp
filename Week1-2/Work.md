# 第 1 周第 2 课作业

文件 foundry.toml:

```toml
[profile.default]
src = "src"
out = "output"
libs = ['lib']
solc_version = "0.8.19"
fs_permissions = [{ access = "read-write", path = "./deploy" }]

# See more config options https://github.com/foundry-rs/foundry/tree/master/config

[fmt]
tab_width = 4
line_length = 120
wrap_comments = true
bracket_spacing = true
override_spacing = true
int_types = 'long'
quote_style = 'double'
number_underscore = 'thousands'
multiline_func_header = 'attributes_first'
ignore = []

[rpc_endpoints]
goerli = "${GOERLI_RPC_URL}"
local = "http://127.0.0.1:8545"

[etherscan]
goerli = { key = "${ETHERSCAN_API_KEY}" }
```

## 第 1 题：修改 Counter 合约，仅有部署者 可以调用 count()

文件 src/Counter.sol：

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.13;

error Unauthorized();

contract Counter {
    uint256 public counter;
    address public owner;

    constructor(uint256 value) {
        counter = value;
        owner = msg.sender;
    }

    function count() public returns (uint256) {
        if (msg.sender != owner) {
            revert Unauthorized();
        }

        uint256 result = counter + 1;
        counter = result;

        return result;
    }
}
```

## 第 2 题：使用 foundry 部署修改后的 Counter

文件 script/Counter.s.sol:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;

import "forge-std/Script.sol";
import "../src/Counter.sol";

contract CounterScript is Script {
    address internal deployer;
    string internal mnemonic;

    function saveContract(string memory network, string memory name, address addr) public {
        string memory filename = string.concat(string.concat("deploy/", network), ".json");

        string memory childObj = "child";
        vm.serializeString(childObj, "name", name);
        string memory childJson = vm.serializeAddress(childObj, "address", addr);

        string memory rootObj = "root";
        vm.serializeString(rootObj, "network", network);
        string memory rootJson = vm.serializeString(rootObj, name, childJson);

        vm.writeJson(rootJson, filename);
    }

    function run() public {
        uint256 deployerPrivateKey = vm.envUint("PRIVATE_KEY");
        vm.startBroadcast(deployerPrivateKey);

        Counter c = new Counter(0);
        console.log("Counter deployed on %s", address(c));
        vm.stopBroadcast();

        saveContract("goerli", "Counter", address(c));
    }
}
```

命令如下：

```shell
source .env
forge build
forge script script/Counter.s.sol --rpc-url $GOERLI_RPC_URL --broadcast
```

截图如下：

![1678532207488](https://user-images.githubusercontent.com/7695325/224480395-f6698ec9-4c04-47bd-8228-0b6a8499e141.png)

## 第 3 题：使用 foundry 测试 Counter:

文件 test/Counter.t.sol:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;

import "forge-std/Test.sol";
import "../src/Counter.sol";

contract CounterTest is Test {
    Counter public counter;

    function setUp() public {
        counter = new Counter(0);
    }

    function test_SetUpState() public {
        assertEq(counter.counter(), 0, "counter should be 0 after setup");
    }

    function testCount() public {
        uint256 result = counter.count();
        assertEq(result, 1, "function count() should return 1");
    }

    function testCountAsOwner() public {
        counter.count();
        assertEq(counter.counter(), 1, "counter should be 1 after call count()");
    }

    function testFailCountAsNotOwner() public {
        vm.prank(address(0));
        counter.count();
    }

    function test_RevertWhen_CountNotOwner() public {
        vm.expectRevert(Unauthorized.selector);
        vm.prank(address(0));
        counter.count();
    }
}
```

### 3.1 Case l: 部署者成功调用 count()

命令如下：

```shell
# 查看部署者账号
cast call --rpc-url $GOERLI_RPC_URL 0x7672d06738f9ce913cf4ef96624bbd0b8c0f1fd9 "owner()"
# 获取计数器当前值
cast call --rpc-url $GOERLI_RPC_URL 0x7672d06738f9ce913cf4ef96624bbd0b8c0f1fd9 "counter()"
# 更新计数器
cast send --rpc-url $GOERLI_RPC_URL --private-key $PRIVATE_KEY 0x7672d06738f9ce913cf4ef96624bbd0b8c0f1fd9 "count()"
# 再次获取计数器当前值
cast call --rpc-url $GOERLI_RPC_URL 0x7672d06738f9ce913cf4ef96624bbd0b8c0f1fd9 "counter()"
```

截图如下：

![1678532943206](https://user-images.githubusercontent.com/7695325/224480861-6dd28504-7217-4bb6-870e-eff3530ee815.png)

### 3.2 Case 2: 其他地址调用 count() 失败

```shell
# 获取计数器当前值
cast call --rpc-url $GOERLI_RPC_URL 0x7672d06738f9ce913cf4ef96624bbd0b8c0f1fd9 "counter()"
# 更换私钥
PRIVATE_KEY=<另外一个私钥>
# 更新计数器
cast send --rpc-url $GOERLI_RPC_URL --private-key $PRIVATE_KEY 0x7672d06738f9ce913cf4ef96624bbd0b8c0f1fd9 "count()"
# 再次获取计数器当前值
cast call --rpc-url $GOERLI_RPC_URL 0x7672d06738f9ce913cf4ef96624bbd0b8c0f1fd9 "counter()"
```

截图如下：

![image](https://user-images.githubusercontent.com/7695325/224481873-10d3eb60-363b-4015-8ff2-4dd327e87043.png)

## 第 4 题：代码开源到区块浏览器，写上合约地址

命令如下：

```shell
forge verify-contract --watch --chain 5 \
    --compiler-version v0.8.19+commit.7dd6d404 -e $ETHERSCAN_API_KEY \
    --constructor-args $(cast abi-encode "constructor(uint256)" 0) \
    0x7672d06738f9ce913cf4ef96624bbd0b8c0f1fd9 src/Counter.sol:Counter
```

截图如下：

![1678536557085](https://user-images.githubusercontent.com/7695325/224483583-174f9d12-99c3-440e-8f17-1bbbd2b8b1df.png)

区块链浏览器地址： https://goerli.etherscan.io/address/0x7672d06738f9ce913cf4ef96624bbd0b8c0f1fd9#code

![1678536636221](https://user-images.githubusercontent.com/7695325/224483650-d06e2484-4296-4a30-a9a1-bb8c493e1472.png)
