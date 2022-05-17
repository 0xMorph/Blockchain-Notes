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

Calling `transfer(bob, 50)` sends bob 50 tokens.

## Drawbacks of ERC20
1. Impossible of handling incoming token transactions

No actual method to handle incoming token transactions and no way to reject non-supported tokens. Thus, no way for the receiver to know about the credit and act upon the credited amount.

2. Transfer process is inefficient

Calling `transfer(bob.address, 10)` is suitable for transfering to EOAs only. If the recipient is a contract, it has to be compatible with the ERC20 standard interface. 
Transfering to a non-ERC20 compatible contract will result in loss of tokens, as they are stuck forever in the receiver contract since it doesn't have to functionality to move the tokens from the contract. 
To transfer funds to a smart contract, Alice needs to use `approve` and `transferFrom` combination.

a. User sets allowance for the `receiverContract` with an amount: `approve(receiverContract, 100)`
b. User makes call from `receiverContract` to move user funds from ERC20 token contract through:
`transferFrom(Alice, receiverContract, 100)`
c. This pattern is also gas-innefficient, requiring 2 seperate transactions to move the funds from the user to the receiving contract.

## ERC223
Fix ERC20 issues like save token losses from accidental transfers and gives receivers ability to accept or decline tokens arriving at their contract address. 

No longer follows `(approve + transferFrom)` ERC20 multistep standards to transfer the tokens to the contract. Gas efficient solution requiring only 1 transaction to transfer tokens to a contract.

ERC223 introduces a callback function for the receiving contract to handle incoming tokens via the `tokenReceived` function. Users can add custom logic to accept or reject tokens arriving at their contract address with that `tokenReceived` function. If they don't have that function, the transaction reverts, preventing accidental transfers to non-supported standard contracts.

This is how `transfer()` function looks for ERC223.

