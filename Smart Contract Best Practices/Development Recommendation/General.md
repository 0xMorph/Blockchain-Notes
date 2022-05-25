## External Calls
All external calls should be treated as a potential security risk. Use the following methods to minimize the danger.

#### Mark untrusted Contracts
When interacting with external contracts, name your variables, methods and contract interfaces to show that interactinv with them might be unsafe. Applies to your own functions that call external contracts.

```sol
// bad
Bank.withdraw(100); // unclear

function makeWithdrawal(uint amount) {
	Bank.withdraw(amount); // unknown if safe or not
}

// good
UntrustedBank.withdraw(100); // untrusted external call
trustedBank.withdraw(100); // trusted bank despite external call

function makeUntrustedWithdrawal(uint amount) {
	UntrustedBank.withdraw(amount);
}
```

#### Avoid state changes after external calls
Whether using `raw calls` (of the form `someAddress.call()`) or `contract calls` (of the form `ExternalContract.someMethod()`), assume malicious code might execute even if not malicious as these code could be executed by any contracts it calls. 

Malicious code might `hijack` the control flow, leading to vulnerabilities due to `reentrancy`.

Avoid state changes after calling to an untrusted external contract. Also known as the [checks-effects-interactions pattern](http://solidity.readthedocs.io/en/develop/security-considerations.html?highlight=check%20effects#use-the-checks-effects-interactions-pattern).

#### Don't use `transfer()` or `send()`
`.transfer()` and `.send()` forward exactly 2,300 gas to the recipient. This was made to prevent `reentrancy` but only makes sense under the assumption that gas costs are constant.

It's recommended to stop using `.transfer()` and `.send()` and instead use `.call()`.

```sol
// bad
contract Vulnerable {
	function withdraw(uint256 amount) external {
		// This forwards 2300 gas, not enough 
		// if recipient is a contract and gas costs change.
		msg.sender.transfer(amount);
	}
}

// good
contract Fixed {
	function withdraw(uint256 amount) {
		// This forwards all available gas. Check the return value!
		(bool success, ) = msg.sender.call.value(amount)("");
		require(success, "Transfer failed.");
	}
}
```
`.call()` actually does nothing to mitigate reentrancy attacks, other precautions must be taken.
To prevent reentrancy attacks, it is recommended that you use the [checks-effects-interactions pattern](https://solidity.readthedocs.io/en/develop/security-considerations.html?highlight=check%20effects#use-the-checks-effects-interactions-pattern).

#### Handle errors in external calls
Solidity has low-level call methods that work on raw addresses: `address.call()`, `address.callcode()`, `address.delegatecall()` and `address.send()`. These methods never throw an exception, and will return `false` when an exception is encountered. While `contract calls` (e.g. `ExternalContract.doSomething()`) automatically propagates a throw, if `doSomething()` throws.

If low-level call methods are used, consider the possibility that the call will fail, by checking the return value.

```sol
// bad
someAddress.send(55);
someAddress.call.value(55)(""); //very dangerous, will forward ALL remaining gas and not check for results
someAddress.call.value(100)(bytes4(sha3("deposit()"))); // if deposit throws an exception, the raw call() returns false and transaction will NOT be reverted

// good
(bool success, ) = someAddress.call.value(55)("");
if(!success) {
	//handle failure code
}

ExternalContract(someAddress).deposit.value(100)();
```

#### Favor pull over push for external calls
External calls can fail. Better to isolate each external call to its own transaction that can be initiated by the recipient of the call.

Payments especially, better to allow users to withdraw funds than push funds to them automatically, also reduces gas limits problems. Avoid combining multiple ether transfers in a single transaction.

```sol
// bad
contract auction {
	address highestBidder;
	uint highestbid;

	function bid() payable {
		require(msg.value >= highestBid);

		if (highestBidder != address(0)) {
			(bool success, ) = highestBidder.call.value(highestBid)("");
			require(success); // if this call consistently fails, no one can bid
		}

		highestBidder = msg.sender;
		highestBid = msg.value;
	}
}

// good
contract auction {
	address highestBidder;
	uint highestBid;
	mapping(address => uint) refunds;

	function bid() payable external {
		require(msg.value >= highestbid);

		if (highestBidder != address(0)) {
			refunds[highestBidder] += highestBid; // record the refund that this user can claim
		}

		highestBidder = msg.sender;
		highestBid = msg.value;
	}

	function withdrawRefund() external {
		uint refund = refunds[msg.sender];
		refunds[msg.sender] = 0;
		(bool success, ) = msg.sender.call.value(refund)("");
		require(success);
	}
}
```

#### Don't delegatecall to untrusted code
The `delegatecall` function is used to call functions from other contracts as if they belong to the caller contract. Thus the callee may chage the state of the calling address. This may be insecure, as here is an example showing how `delegatecall` can lead to destruction of the contract and loss of its balance.

```sol
contract Destructor {
	function doWork() external {
		selfdestruct(0);
	}
}

contract Worker {
	function doWork(address _internalWorker) public {
		//unsafe
		_internalWorker.delegatecall(bytes4(keccak256("doWork()")));
	}
}
```
If `Worker.doWork()` is called with the address of the deployed `Destructor` contract as an argument, the `Worker` contract will self-destruct

## Force-feeding Ether
Caution on coding an invariant that strictly checks the balance of a contract.

An attacker and force to send ether to any account and it cannot be prevented, not even via a fallback function that does a `revert()`.

Attacker creates a contract, funding it with 1 wei, then invoking `selfdestruct(victimAddress)`. No code is invoked in `victimAddress`, so it cannot be prevented. This is also true of block reward which is sent to the address of the miner, which can be any arbitrary address. 

Since the contract address can be precomputed, ether can eb sent to an address before the contract is deployed.

## Public on-chain Data
Avoid requiring users to publish info too early. Use commitment schemes with separate phases

I will stop taking notes here for these few weeks, need to focus on more exploitation stuff over development stuff.

https://consensys.github.io/smart-contract-best-practices/development-recommendations/general/public-data/