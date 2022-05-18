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

```sol
function transfer(address _to, uint _amount, bytes calldata _data) public override returns (bool success) {
	balances[msg.sender] = balances[msg.sender] - _amount;
	if(Address.isContract(_to)) {
		IERC223Recipient(_to).tokenReceived(msg.sender, _amount, _data);
	}
	emit Transfer(msg.sender, _to, _amount);
	emit TransferData(_data);
	return true;
}
```

`transfer()` takes another optional argument `data`, used to make a call to the recipient contract with the data supplied by the caller. The recipient contract can use this data to do something. This behavior also introduces the concept of transfer + call. This transfers the tokens and instructs for something to be done on the recipient contract.

If recipient address is a contract, the ERC223 token makes a call to the recipient contract on `tokenReceived` function.

```sol
function tokenReceived(address _from, uint _amount, bytes memory _data) public {
	// handling incoming token logic
	require(msg.sender == address(token));
	balanceOf[_from] += _amount;
	// do something with the data supplied
	doSomething(_data);
}
```

The contract can define its own logic to accept the incoming token transfer or to reject and many more.

Drawback of ERC223 is that it overrides the design of `transfer(address, uint256, bytes)` function since it changes the standard behavior of ERC20's `transfer(address, uint256)`. Not backward compatible with existing ERC20 compatible contracts, since `tokenReceived()` is required on the receiving contract else the transfer would just revert.

## ERC677
ERC677 provides useful functionality without colliding with ERC20 standard. ERC677 adds a new function `transferAndCall()` to the existing ERC20 standard, unlike ERC223, which overrides the existing `transfer()` function.

`transferAndCall()` is called to transfer tokens to a contract and then call the recipient contract with the additional data supplied from the sender. ERC677 tokens can be stored in any ERC20-compatible wallet. Receiving contracts would have the function `onTokenTransfer(address, uint256, bytes)` to write custom logic for the incoming call.

Even if the recipient contract does not have `onTokenTransfer()` we can still use the regular ERC20 function `transfer(address, uint256)` to transfer the tokens. 

Famouse ERC677 tokens include Chainlink (LINK)
```sol
function transferAndCall(address _to, uint _value, bytes _data) public returns (bool success) {
	super.transfer(_to, _value); // STANDARD ERC20 TRANSFER
	Transfer(msg.sender, _to, _value, _data);
	if (isContract(_to)) { // CONDITION
		contractFallback(_to, _value, _data);
	}
	return true;
}

function contractFallback(address _to, uint _value, bytes _data) private {
	ERC677Receiver receiver = ERC677Receiver(_to);
	receiver.onTokenTransfer(msg.sender, _value, _data);
}
```

To do an API call or Chainlink VRF request requires a function call on a smart contract. This needs LINK Tokens, With the regular ERC20 standard, we need to do this in multiple steps (approve + call) but with ERC677, you can do both in a single transaction `transferAndCall(to, value, data)`. It first transfers the tokens to the receiver, then checks if its a contract, then makes a call on the recipient's `onTokenTransfer()` along with the data included.

```sol
modifier onlyLINK() {
	require(msg.sender == address(LINK), "Must use LINK token");
	_;
	
}

function onTokenTransfer(address sender, uint256 fee, bytes memory _data) public onlyLINK {
	(bytes32 keyHash, uint256 seed) = abi.decode(_data, (bytes32, uint256));
	emit RandomnessRequest(sender, keyHash, seed);
}
```
Receiver contract checks if the incoming token call `msg.sender` is coming from the LINK address, then it decodes the data supplied to the transfer and emits the `RandomnessRequest` event.

## ERC1363
Similar to ERC677 but with great extensions, in ERC677 we have `transferAndCall()` function but EIP1363 adds 2 more functions as extentions to the ERC677 and ERC20 standard compatibility.

* `transferFromAndcall(..)`: Similar to `transferAndCall()`, receiver contract can now transfer the tokens from the spender to the receiver within the specified allowance limit and do something with the call.
* `approveAndCall(..)`: Sets the allowance amount to the spender and do something with a call

![[Pasted image 20220518124056.png]]

Using `approveAndCall()` now the token can make a callback to the recipient contract after approving the tokens to the recipient to instruct them to do somehting. By so, contracts can accept ERC20 payments to create a token payable crowdsale, selling services for tokens etc.