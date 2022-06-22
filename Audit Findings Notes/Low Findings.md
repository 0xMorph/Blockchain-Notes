## Unused `receive()` function locks Ether in contract
If the intention is for the Ether to be used, the function should call another function, else it should revert

```sol
receive() external payable {}
```
