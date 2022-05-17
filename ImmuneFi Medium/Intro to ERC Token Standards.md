## ERC20 
Fungible tokens, each token has the same value, uses gas to cover transaction fees in the native asset.

Core functionality of ERC20 is `transfer`

```sol
function transfer(address _to, uint _amount) public {
	uint256 senderBalance = _balances[msg.sender];
	require(senderBalance >= _amount, "ERC20: Transfer amount exceeds balance");
	balances[msg.sender] = senderBalance - _value;
	balances[_to] = balances[_to] + _value;

	emit Transfer(msg.sender, _to, _value);
	
}
```