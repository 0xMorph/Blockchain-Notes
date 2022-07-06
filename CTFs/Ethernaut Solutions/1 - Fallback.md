## Overview
There are a constructor, a modifier, 3 functions and a fallback function.

The constructor sets the address who deployed the contract as the "owner", as well as setting the contributions of it to be 1000 ether.

The modifier `onlyOwner` is as what it says, any function with this modifier is only callable by the owner.

The function `contribute` is payable and requires the message value to be less than 0.001 ether, then adds the value to the contributions of the `msg.sender`. This function also could change the owner, under the condition that the `msg.sender`'s contribution is larger than the current owner, so who contributes most owns the contract.

`getContribution` returns a uint256, which returns the contributions of `msg.sender`.

`withdraw` has the `onlyOwner` modifier, allowing only the owner to transfer the whole balance of the contract to the owner.

Lastly the `fallback` function is the other only payable function, requiring the `msg.sender` to already contribute more than 0 and the `msg.value` to be more than 0. If satisfied then sets the `msg.sender` as the owner. 

The fallback function is where the exploit would work, so we need to satisfy it's requirements first by contributing less than 0.001 ether to the `contribute({value: 1})` function then using `contract.sendTransaction({value: 1})`, then we are now the owner. Lastly, call `withdraw` and we completed the level.

## Foundry POC:
