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
Solidity has low-level call methods that work on raw addresses: `address.call()`, `address.callcode()`, `address.delegatecall()` and `address.send()`.