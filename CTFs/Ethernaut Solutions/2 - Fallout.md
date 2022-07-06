## Overview
Misnamed function. In this Solidity version, there is no constructors, so the contract when deployed can only initialize with the function sharing the same name with the contract.

Sometimes a contract could change their name, but forget to rename it's function, there's where the real life situation of the exploit of this contract happened.

Here the function `Fal1out` is mispelled, calling it makes the `msg.sender` become the owner. Rest of the contract can be ignored to solve the level.

## Foundry POC:
