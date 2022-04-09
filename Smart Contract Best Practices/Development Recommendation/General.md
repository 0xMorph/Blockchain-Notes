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