1.  **Unhandled return values of** _**transfer**_ **and** _**transferFrom**_: ERC20 implementations are not always consistent. Some implementations of _transfer_ and _transferFrom_ could return ‘false’ on failure instead of reverting. It is safer to wrap such calls into _require()_ statements to these failures.
    
    1.  Recommendation: Check the return value and revert on 0/false or use OpenZeppelin’s _SafeERC20_ wrapper functions
        
    2.  Medium severity finding from [Consensys Diligence Audit of Aave Protocol V2](https://consensys.net/diligence/audits/2020/09/aave-protocol-v2/#unhandled-return-values-of-transfer-and-transferfrom)
        
2.  **Random task execution**: In a scenario where a user takes a flash loan, __parseFLAndExecute()_ gives the flash loan wrapper contract (_FLAaveV2_, _FLDyDx_) the permission to execute functions on behalf of the user’s _DSProxy_. This execution permission is revoked only after the entire recipe execution is finished, which means that in case that any of the external calls along the recipe execution is malicious, it might call _executeAction()_ back, i.e. Reentrancy Attack, and inject any task it wishes (e.g. take user’s funds out, drain approved tokens, etc)
    
    1.  Recommendation: A reentrancy guard (mutex) should be used to prevent such attack
        
    2.  Critical severity finding from [Consensys Diligence Audit of Defi Saver](https://consensys.net/diligence/audits/2021/03/defi-saver/#random-task-execution)
        
3.  **Tokens with more than 18 decimal points will cause issues**: It is assumed that the maximum number of decimals for each token is 18. However uncommon, it is possible to have tokens with more than 18 decimals, as an example YAMv2 has 24 decimals. This can result in broken code flow and unpredictable outcomes
    
    1.  Recommendation: Make sure the code won’t fail in case the token’s decimals is more than 18
        
    2.  Major severity finding from [Consensys Diligence Audit of Defi Saver](https://consensys.net/diligence/audits/2021/03/defi-saver/#tokens-with-more-than-18-decimal-points-will-cause-issues)
        
4.  **Error codes of Compound’s** _**Comptroller.enterMarket**_**,** _**Comptroller.exitMarket**_ **are not checked**: Compound’s _enterMarket_/_exitMarket_ functions return an error code instead of reverting in case of failure. DeFi Saver smart contracts never check for the error codes returned from Compound smart contracts.
    
    1.  Recommendation: Caller contract should revert in case the error code is not 0
        
    2.  Major severity finding from [Consensys Diligence Audit of Defi Saver](https://consensys.net/diligence/audits/2021/03/defi-saver/#error-codes-of-compound-s-comptroller-entermarket-comptroller-exitmarket-are-not-checked)
        
5.  **Reversed order of parameters in allowance function call**: the parameters that are used for the allowance function call are not in the same order that is used later in the call to _safeTransferFrom_.
    
    1.  Recommendation: Reverse the order of parameters in allowance function call to fit the order that is in the safeTransferFrom function call.
        
    2.  Medium severity finding from [Consensys Diligence Audit of Defi Saver](https://consensys.net/diligence/audits/2021/03/defi-saver/#reversed-order-of-parameters-in-allowance-function-call)
        
6.  **Token approvals can be stolen in** _**DAOfiV1Router01.addLiquidity()**_: _DAOfiV1Router01.addLiquidity()_ creates the desired pair contract if it does not already exist, then transfers tokens into the pair and calls _DAOfiV1Pair.deposit()_. There is no validation of the address to transfer tokens from, so an attacker could pass in any address with nonzero token approvals to _DAOfiV1Router_. This could be used to add liquidity to a pair contract for which the attacker is the _pairOwner_, allowing the stolen funds to be retrieved using DAOfiV1Pair.withdraw().
    
    1.  Recommendation: Transfer tokens from msg.sender instead of lp.sender
        
    2.  Critical severity finding from [Consensys Diligence Audit of DAOfi](https://consensys.net/diligence/audits/2021/02/daofi/#token-approvals-can-be-stolen-in-daofiv1router01-addliquidity)
        
7.  _**swapExactTokensForETH**_ **checks the wrong return value**: Instead of checking that the amount of tokens received from a swap is greater than the minimum amount expected from this swap, it calculates the difference between the initial receiver’s balance and the balance of the router
    
    1.  Recommendation: Check the intended values
        
    2.  Major severity finding from [Consensys Diligence Audit of DAOfi](https://consensys.net/diligence/audits/2021/02/daofi/#the-swapexacttokensforeth-checks-the-wrong-return-value)
        
8.  _**DAOfiV1Pair.deposit()**_ **accepts deposits of zero, blocking the pool**: _DAOfiV1Pair.deposit()_ is used to deposit liquidity into the pool. Only a single deposit can be made, so no liquidity can ever be added to a pool where deposited == true. The deposit() function does not check for a nonzero deposit amount in either token, so a malicious user that does not hold any of the _baseToken_ or _quoteToken_ can lock the pool by calling deposit() without first transferring any funds to the pool.
    
    1.  Recommendation: Require a minimum deposit amount with non-zero checks
        
    2.  Medium severity finding from [Consensys Diligence Audit of DAOfi](https://consensys.net/diligence/audits/2021/02/daofi/#daofiv1pair-deposit-accepts-deposits-of-zero-blocking-the-pool)
        
9.  _**GenesisGroup.commit**_ **overwrites previously-committed values**: The amount stored in the recipient’s _committedFGEN_ balance overwrites any previously-committed value. Additionally, this also allows anyone to commit an amount of “0” to any account, deleting their commitment entirely.
    
    1.  Recommendation: Ensure the committed amount is added to the existing commitment.
        
    2.  Critical severity finding from [Consensys Diligence Audit of Fei Protocol](https://consensys.net/diligence/audits/2021/01/fei-protocol/#genesisgroup-commit-overwrites-previously-committed-values)
        
10.  **Purchasing and committing still possible after launch**: Even after _GenesisGroup.launch_ has successfully been executed, it is still possible to invoke _GenesisGroup.purchase_ and _GenesisGroup.commit_.
    
    1.  Recommendation: Consider adding validation in _GenesisGroup.purchase_ and _GenesisGroup.commit_ to make sure that these functions cannot be called after the launch.
        
    2.  Critical severity finding from [Consensys Diligence Audit of Fei Protocol](https://consensys.net/diligence/audits/2021/01/fei-protocol/#purchasing-and-committing-still-possible-after-launch)
        
11.  _**UniswapIncentive**_ **overflow on pre-transfer hooks**: Before a token transfer is performed, Fei performs some combination of mint/burn operations via _UniswapIncentive.incentivize_. Both _incentivizeBuy_ and _incentivizeSell_ calculate buy/sell incentives using overflow-prone math, then mint / burn from the target according to the results. This may have unintended consequences, like allowing a caller to mint tokens before transferring them, or burn tokens from their recipient.
    
    1.  Recommendation: Ensure casts in _getBuyIncentive_ and _getSellPenalty_ do not overflow
        
    2.  Major severity finding from [Consensys Diligence Audit of Fei Protocol](https://consensys.net/diligence/audits/2021/01/fei-protocol/#uniswapincentive-overflow-on-pre-transfer-hooks)
        
12.  _**BondingCurve**_ **allows users to acquire FEI before launch**: allocate can be called before genesis launch, as long as the contract holds some nonzero PCV. By force-sending the contract 1 wei, anyone can bypass the majority of checks and actions in allocate, and mint themselves FEI each time the timer expires.
    
    1.  Recommendation: Prevent allocate from being called before genesis launch
        
    2.  Medium severity finding from [Consensys Diligence Audit of Fei Protocol](https://consensys.net/diligence/audits/2021/01/fei-protocol/#bondingcurve-allows-users-to-acquire-fei-before-launch)
        
13.  _**Timed.isTimeEnded**_ **returns true if the timer has not been initialized**: _Timed_ initialization is a 2-step process: 1) _Timed.duration_ is set in the constructor 2) _Timed.startTime_ is set when the method __initTimed_ is called. Before this second method is called, _isTimeEnded_() calculates remaining time using a _startTime_ of 0, resulting in the method returning true for most values, even though the timer has not technically been started.
    
    1.  Recommendation: If Timed has not been initialized, _isTimeEnded_() should return false, or revert
        
    2.  Medium severity finding from [Consensys Diligence Audit of Fei Protocol](https://consensys.net/diligence/audits/2021/01/fei-protocol/#timed-istimeended-returns-true-if-the-timer-has-not-been-initialized)
        
14.  **Overflow/underflow protection**: Having overflow/underflow vulnerabilities is very common for smart contracts. It is usually mitigated by using SafeMath or using solidity version ^0.8 (after solidity 0.8 arithmetical operations already have default overflow/underflow protection). In this code, many arithmetical operations are used without the ‘safe’ version. The reasoning behind it is that all the values are derived from the actual ETH values, so they can’t overflow.
    
    1.  Recommendation: In our opinion, it is still safer to have these operations in a safe mode. So we recommend using SafeMath or solidity version ^0.8 compiler.
        
    2.  Medium severity finding from [Consensys Diligence Audit of Fei Protocol](https://consensys.net/diligence/audits/2021/01/fei-protocol/#overflow-underflow-protection)
        
15.  **Unchecked return value for** _**IWETH.transfer**_ **call**: In _EthUniswapPCVController_, there is a call to _IWETH.transfer_ that does not check the return value. It is usually good to add a require-statement that checks the return value or to use something like _safeTransfer_; unless one is sure the given token reverts in case of a failure.
    
    1.  Recommendation: Consider adding a require-statement or using _safeTransfer_
        
    2.  Medium severity finding from [Consensys Diligence Audit of Fei Protocol](https://consensys.net/diligence/audits/2021/01/fei-protocol/#unchecked-return-value-for-iweth-transfer-call)
        
16.  _**GenesisGroup.emergencyExit**_ **remains functional after launch**: _emergencyExit_ is intended as an escape mechanism for users in the event the genesis launch method fails or is frozen. _emergencyExit_ becomes callable 3 days after launch is callable. These two methods are intended to be mutually-exclusive, but are not: either method remains callable after a successful call to the other. This may result in accounting edge cases.
    
    1.  Recommendation: 1) Ensure launch cannot be called if _emergencyExit_ has been called 2) Ensure _emergencyExit_ cannot be called if launch has been called
        
    2.  Medium severity finding from [Consensys Diligence Audit of Fei Protocol](https://consensys.net/diligence/audits/2021/01/fei-protocol/#genesisgroup-emergencyexit-remains-functional-after-launch)
        
17.  **ERC20 tokens with no return value will fail to transfer**: Although the ERC20 standard suggests that a transfer should return true on success, many tokens are non-compliant in this regard. In that case, the .transfer() call here will revert even if the transfer is successful, because solidity will check that the _RETURNDATASIZE_ matches the ERC20 interface.
    
    1.  Recommendation: Consider using OpenZeppelin’s SafeERC20
        
    2.  Major severity finding from [Consensys Diligence Audit of bitbank](https://consensys.net/diligence/audits/2020/11/bitbank/#erc20-tokens-with-no-return-value-will-fail-to-transfer)
        
18.  **Reentrancy vulnerability in** _**MetaSwap.swap()**_: If an attacker is able to reenter _swap()_, they can execute their own trade using the same tokens and get all the tokens for themselves.
    
    1.  Recommendation: Use a simple reentrancy guard, such as OpenZeppelin’s ReentrancyGuard to prevent reentrancy in _MetaSwap.swap()_
        
    2.  Major severity finding from [Consensys Diligence Audit of MetaSwap](https://consensys.net/diligence/audits/2020/08/metaswap/#reentrancy-vulnerability-in-metaswap-swap)
        
19.  **A new malicious adapter can access users’ tokens**: The purpose of the _MetaSwap_ contract is to save users gas costs when dealing with a number of different aggregators. They can just approve() their tokens to be spent by _MetaSwap_ (or in a later architecture, the Spender contract). They can then perform trades with all supported aggregators without having to reapprove anything. A downside to this design is that a malicious (or buggy) adapter has access to a large collection of valuable assets. Even a user who has diligently checked all existing adapter code before interacting with _MetaSwap_ runs the risk of having their funds intercepted by a new malicious adapter that’s added later.
    
    1.  Recommendation: Make _MetaSwap_ contract the only contract that receives token approval. It then moves tokens to the Spender contract before that contract _DELEGATECALLs_ to the appropriate adapter. In this model, newly added adapters shouldn’t be able to access users’ funds.
        
    2.  Medium severity finding from [Consensys Diligence Audit of MetaSwap](https://consensys.net/diligence/audits/2020/08/metaswap/#a-new-malicious-adapter-can-access-users-tokens)
        
20.  **Owner can front-run traders by updating adapters**: _MetaSwap_ owners can front-run users to swap an adapter implementation. This could be used by a malicious or compromised owner to steal from users. Because adapters are _DELEGATECALL’ed_, they can modify storage. This means any adapter can overwrite the logic of another adapter, regardless of what policies are put in place at the contract level. Users must fully trust every adapter because just one malicious adapter could change the logic of all other adapters.
    
    1.  Recommendation: At a minimum, disallow modification of existing adapters. Instead, simply add new adapters and disable the old ones.
        
    2.  Medium severity finding from [Consensys Diligence Audit of MetaSwap](https://consensys.net/diligence/audits/2020/08/metaswap/#owner-can-front-run-traders-by-updating-adapters)
        
21.  **Users can collect interest from** _**SavingsContract**_ **by only staking mTokens momentarily**: The _SAVE_ contract allows users to deposit _mAssets_ in return for lending yield and swap fees. When depositing _mAsset_, users receive a “credit” tokens at the momentary credit/_mAsset_ exchange rate which is updated at every deposit. However, the smart contract enforces a minimum timeframe of 30 minutes in which the interest rate will not be updated. A user who deposits shortly before the end of the timeframe will receive credits at the stale interest rate and can immediately trigger an update of the rate and withdraw at the updated (more favorable) rate after the 30 minutes window. As a result, it would be possible for users to benefit from interest payouts by only staking mAssets momentarily and using them for other purposes the rest of the time.
    
    1.  Recommendation: Remove the 30 minutes window such that every deposit also updates the exchange rate between credits and tokens.
        
    2.  Medium severity finding from [Consensys Diligence Audit of mstable-1.1](https://consensys.net/diligence/audits/2020/07/mstable-1.1/#users-can-collect-interest-from-savingscontract-by-only-staking-mtokens-momentarily)
        
22.  **Oracle updates can be manipulated to perform atomic front-running attack**: It is possible to atomically arbitrage rate changes in a risk-free way by “sandwiching” the Oracle update between two transactions. The attacker would send the following 2 transactions at the moment the Oracle update appears in the mempool: 1) The first transaction, which is sent with a higher gas price than the Oracle update transaction, converts a very small amount. This “locks in” the conversion weights for the block since _handleExternalRateChange()_ only updates weights once per block. By doing this, the arbitrageur ensures that the stale Oracle price is initially used when doing the first conversion in the following transaction. The second transaction, which is sent at a slightly lower gas price than the transaction that updates the Oracle, performs a large conversion at the old weight, adds a small amount of Liquidity to trigger rebalancing and converts back at the new rate. The attacker can obtain liquidity for step 2 using a flash loan. The attack will deplete the reserves of the pool.
    
    1.  Recommendation: Do not allow users to trade at a stale Oracle rate and trigger an Oracle price update in the same transaction.
        
    2.  Critical severity finding from [Consensys Diligence Audit of Bancor v2 AMM](https://consensys.net/diligence/audits/2020/06/bancor-v2-amm-security-audit/#oracle-updates-can-be-manipulated-to-perform-atomic-front-running-attack)
        
23.  **Certain functions lack input validation routines**: The functions should first check if the passed arguments are valid first. These checks should include, but not be limited to: 1) uint should be larger than 0 when 0 is considered invalid 2) uint should be within constraints 3) int should be positive in some cases 4) length of arrays should match if more arrays are sent as arguments 5) addresses should not be 0x0
    
    1.  Recommendation: Add tests that check if all of the arguments have been validated. Consider checking arguments as an important part of writing code and developing the system.
        
    2.  Major severity finding from [Consensys Diligence Audit of Shell Protocol](https://consensys.net/diligence/audits/2020/06/shell-protocol/#certain-functions-lack-input-validation-routines)
        
24.  **Remove** _**Loihi**_ **methods that can be used as backdoors by the administrator**: There are several functions in _Loihi_ that give extreme powers to the shell administrator. The most dangerous set of those is the ones granting the capability to add assimilators. Since assimilators are essentially a proxy architecture to delegate code to several different implementations of the same interface, the administrator could, intentionally or unintentionally, deploy malicious or faulty code in the implementation of an assimilator. This means that the administrator is essentially totally trusted to not run code that, for example, drains the whole pool or locks up the users’ and LPs’ tokens. In addition to these, the function _safeApprove_ allows the administrator to move any of the tokens the contract holds to any address regardless of the balances any of the users have. This can also be used by the owner as a backdoor to completely drain the contract.
    
    1.  Recommendation: Remove the _safeApprove_ function and, instead, use a trustless escape-hatch mechanism. For the assimilator addition functions, our recommendation is that they are made completely internal, only callable in the constructor, at deploy time. Even though this is not a big structural change (in fact, it reduces the attack surface), it is, indeed, a feature loss. However, this is the only way to make each shell a time-invariant system. This would not only increase Shell’s security but also would greatly improve the trust the users have in the protocol since, after deployment, the code is now static and auditable.
        
    2.  Major severity finding from [Consensys Diligence Audit of Shell Protocol](https://consensys.net/diligence/audits/2020/06/shell-protocol/#remove-loihi-methods-that-can-be-used-as-backdoors-by-the-administrator)
        
25.  **A reverting fallback function will lock up all payouts**: In _BoxExchange.sol_, the internal function __transferEth_() reverts if the transfer does not succeed. The __payment_() function processes a list of transfers to settle the transactions in an _ExchangeBox_. If any of the recipients of an ETH transfer is a smart contract that reverts, then the entire payout will fail and will be unrecoverable.
    
    1.  Recommendation: 1) Implement a queuing mechanism to allow buyers/sellers to initiate the withdrawal on their own using a ‘pull-over-push pattern.’ 2) Ignore a failed transfer and leave the responsibility up to users to receive them properly.
        
    2.  Critical severity finding from [Consensys Diligence Audit of Lien Protocol](https://consensys.net/diligence/audits/2020/05/lien-protocol/#a-reverting-fallback-function-will-lock-up-all-payouts)
        
26.  _**Saferagequit**_ **makes you lose funds**: _safeRagequit_ and _ragequit_ functions are used for withdrawing funds from the LAO. The difference between them is that ragequit function tries to withdraw all the allowed tokens and _safeRagequit_ function withdraws only some subset of these tokens, defined by the user. It’s needed in case the user or _GuildBank_ is blacklisted in some of the tokens and the transfer reverts. The problem is that even though you can quit in that case, you’ll lose the tokens that you exclude from the list. To be precise, the tokens are not completely lost, they will belong to the LAO and can still potentially be transferred to the user who quit. But that requires a lot of trust, coordination, time and anyone can steal some part of these tokens.
    
    1.  Recommendation: Implementing pull pattern for token withdrawals should solve the issue. Users will be able to quit the LAO and burn their shares but still keep their tokens in the LAO’s contract for some time if they can’t withdraw them right now.
        
    2.  Critical severity finding from [Consensys Diligence Audit of The Lao](https://consensys.net/diligence/audits/2020/01/the-lao)
        
27.  **Creating proposal is not trustless**: Usually, if someone submits a proposal and transfers some amount of tribute tokens, these tokens are transferred back if the proposal is rejected. But if the proposal is not processed before the emergency processing, these tokens will not be transferred back to the proposer. This might happen if a tribute token or a deposit token transfers are blocked. Tokens are not completely lost in that case, they now belong to the LAO shareholders and they might try to return that money back. But that requires a lot of coordination and time and everyone who ragequits during that time will take a part of that tokens with them.
    
    1.  Recommendation: Pull pattern for token transfers would solve the issue
        
    2.  Critical severity finding from [Consensys Diligence Audit of The Lao](https://consensys.net/diligence/audits/2020/01/the-lao)
        
28.  **Emergency processing can be blocked**: The main reason for the emergency processing mechanism is that there is a chance that some token transfers might be blocked. For example, a sender or a receiver is in the USDC blacklist. Emergency processing saves from this problem by not transferring tribute token back to the user (if there is some) and rejecting the proposal. The problem is that there is still a deposit transfer back to the sponsor and it could be potentially blocked too. If that happens, proposal can’t be processed and the LAO is blocked.
    
    1.  Recommendation: Pull pattern for token transfers would solve the issue
        
    2.  Critical severity finding from [Consensys Diligence Audit of The Lao](https://consensys.net/diligence/audits/2020/01/the-lao)
        
29.  **Token Overflow might result in system halt or loss of funds**: If a token overflows, some functionality such as _processProposal_, _cancelProposal_ will break due to SafeMath reverts. The overflow could happen because the supply of the token was artificially inflated to oblivion.
    
    1.  Recommendation: We recommend to allow overflow for broken or malicious tokens. This is to prevent system halt or loss of funds. It should be noted that in case an overflow occurs, the balance of the token will be incorrect for all token holders in the system
        
    2.  Major severity finding from [Consensys Diligence Audit of The Lao](https://consensys.net/diligence/audits/2020/01/the-lao)
        
30.  **Whitelisted tokens limit**: __ragequit_ function is iterating over all whitelisted tokens. If the number of tokens is too big, a transaction can run out of gas and all funds will be blocked forever.
    
    1.  Recommendation: A simple solution would be just limiting the number of whitelisted tokens. If the intention is to invest in many new tokens over time, and it’s not an option to limit the number of whitelisted tokens, it’s possible to add a function that removes tokens from the whitelist. For example, it’s possible to add a new type of proposal that is used to vote on token removal if the balance of this token is zero. Before voting for that, shareholders should sell all the balance of that token.
        
    2.  Major severity finding from [Consensys Diligence Audit of The Lao](https://consensys.net/diligence/audits/2020/01/the-lao)
        
31.  **Summoner can steal funds using bailout**: The _bailout_ function allows anyone to transfer kicked user’s funds to the summoner if the user does not call safeRagequit (which forces the user to lose some funds). The intention is for the summoner to transfer these funds to the kicked member afterwards. The issue here is that it requires a lot of trust to the summoner on the one hand, and requires more time to kick the member out of the LAO.
    
    1.  Recommendation: By implementing pull pattern for token transfers, kicked member won’t be able to block the ragekick and the LAO members would be able to kick anyone much quicker. There is no need to keep the _bailout_ function.
        
    2.  Major severity finding from [Consensys Diligence Audit of The Lao](https://consensys.net/diligence/audits/2020/01/the-lao)
        
32.  **Sponsorship front-running**: If proposal submission and sponsorship are done in 2 different transactions, it’s possible to front-run the _sponsorProposal_ function by any member. The incentive to do that is to be able to block the proposal afterwards.
    
    1.  Recommendation: Pull pattern for token transfers will solve the issue. Front-running will still be possible but it doesn’t affect anything.
        
    2.  Major severity finding from [Consensys Diligence Audit of The Lao](https://consensys.net/diligence/audits/2020/01/the-lao)
        
33.  **Delegate assignment front-running**: Any member can front-run another member’s _delegateKey_ assignment. If you try to submit an address as your _delegateKey_, someone else can try to assign your delegate address to themselves. While incentive of this action is unclear, it’s possible to block some address from being a delegate forever.
    
    1.  Recommendation: Make it possible for a _delegateKey_ to approve _delegateKey_ assignment or cancel the current delegation. Commit-reveal methods can also be used to mitigate this attack.
        
    2.  Medium severity finding from [Consensys Diligence Audit of The Lao](https://consensys.net/diligence/audits/2020/01/the-lao)
        
34.  **Queued transactions cannot be canceled**: The Governor contract contains special functions to set it as the admin of the _Timelock_. Only the admin can call _Timelock.cancelTransaction_. There are no functions in Governor that call _Timelock.cancelTransaction_. This makes it impossible for _Timelock.cancelTransaction_ to ever be called.
    
    1.  Recommendation: Short term, add a function to the Governor that calls _Timelock.cancelTransaction_. It is unclear who should be able to call it, and what other restrictions there should be around cancelling a transaction. Long term, consider letting Governor inherit from _Timelock_. This would allow a lot of functions and code to be removed and significantly lower the complexity of these two contracts.
        
    2.  High Risk severity finding from [ToB’s Audit of Origin Dollar](https://github.com/trailofbits/publications/blob/master/reviews/OriginDollar.pdf)
        
35.  **Proposal transactions can be executed separately and block** _**Proposal.execute**_ **call**: Missing access controls in the _Timelock.executeTransaction_ function allow Proposal transactions to be executed separately, circumventing the _Governor.execute_ function.
    
    1.  Recommendation: Short term, only allow the admin to call _Timelock.executeTransaction_
        
    2.  High Risk severity finding from [ToB’s Audit of Origin Dollar](https://github.com/trailofbits/publications/blob/master/reviews/OriginDollar.pdf)
        
36.  **Proposals could allow** _**Timelock.admin**_ **takeover**: The Governor contract contains special functions to let the guardian queue a transaction to change the _Timelock.admin_. However, a regular Proposal is also allowed to contain a transaction to change the _Timelock.admin_. This poses an unnecessary risk in that an attacker could create a Proposal to change the _Timelock.admin_.
    
    1.  Recommendation: Short term, add a check that prevents _setPendingAdmin_ to be included in a Proposal
        
    2.  High Risk severity finding from [ToB’s Audit of Origin Dollar](https://github.com/trailofbits/publications/blob/master/reviews/OriginDollar.pdf)
        
37.  **Reentrancy and untrusted contract call in** _**mintMultiple**_: Missing checks and no reentrancy prevention allow untrusted contracts to be called from _mintMultiple_. This could be used by an attacker to drain the contracts.
    
    1.  Recommendation: Short term, add checks that cause _mintMultiple_ to revert if the amount is zero or the asset is not supported. Add a reentrancy guard to the _mint_, _mintMultiple_, _redeem_, and _redeemAll_ functions. Long term, make use of Slither which will flag the reentrancy. Or even better, use Crytic and incorporate static analysis checks into your CI/CD pipeline. Add reentrancy guards to all non-view functions callable by anyone. Make sure to always revert a transaction if an input is incorrect. Disallow calling untrusted contracts.
        
    2.  High Risk severity finding from [ToB’s Audit of Origin Dollar](https://github.com/trailofbits/publications/blob/master/reviews/OriginDollar.pdf)
        
38.  **Lack of return value checks can lead to unexpected results**: Several function calls do not check the return value. Without a return value check, the code is error-prone, which may lead to unexpected results.
    
    1.  Recommendation: Short term, check the return value of all calls mentioned above. Long term, subscribe to Crytic.io to catch missing return checks. Crytic identifies this bug type automatically.
        
    2.  High Risk severity finding from [ToB’s Audit of Origin Dollar](https://github.com/trailofbits/publications/blob/master/reviews/OriginDollar.pdf)
        
39.  **External calls in loop can lead to denial of service**: Several function calls are made in unbounded loops. This pattern is error-prone as it can trap the contracts due to the gas limitations or failed transactions.
    
    1.  Recommendation: Short term, review all the loops mentioned above and either: 1) allow iteration over part of the loop, or 2) remove elements. Long term, subscribe to Crytic.io to review external calls in loops. Crytic catches bugs of this type.
        
    2.  High Risk severity finding from [ToB’s Audit of Origin Dollar](https://github.com/trailofbits/publications/blob/master/reviews/OriginDollar.pdf)
        
40.  **OUSD allows users to transfer more tokens than expected**: Under certain circumstances, the OUSD contract allows users to transfer more tokens than the ones they have in their balance. This issue seems to be caused by a rounding issue when the _creditsDeducted_ is calculated and subtracted.
    
    1.  Recommendation: Short term, make sure the balance is correctly checked before performing all the arithmetic operations. This will make sure it does not allow to transfer more than expected. Long term, use Echidna to write properties that ensure ERC20 transfers are transferring the expected amount.
        
    2.  High Risk severity finding from [ToB’s Audit of Origin Dollar](https://github.com/trailofbits/publications/blob/master/reviews/OriginDollar.pdf)
        
41.  **OUSD total supply can be arbitrary, even smaller than user balances**: The OUSD token contract allows users to opt out of rebasing effects. At that point, their exchange rate is “fixed”, and further rebases will not have an impact on token balances (until the user opts in).
    
    1.  Recommendation: Short term, we would advise making clear all common invariant violations for users and other stakeholders. Long term, we would recommend designing the system in such a way to preserve as many commonplace invariants as possible.
        
    2.  High Risk severity finding from [ToB’s Audit of Origin Dollar](https://github.com/trailofbits/publications/blob/master/reviews/OriginDollar.pdf)
        
42.  **Flash minting can be used to redeem** _**fyDAI**_: The flash-minting feature from the _fyDAI_ token can be used to redeem an arbitrary amount of funds from a mature token.
    
    1.  Recommendation: Short term, disallow calls to redeem in the _YDai_ and Unwind contracts during flash minting. Long term, do not include operations that allow any user to manipulate an arbitrary amount of funds, even if it is in a single transaction. This will prevent attackers from gaining leverage to manipulate the market and break internal invariants.
        
    2.  Medium Risk severity finding from [ToB’s Audit of Yield Protocol](https://github.com/trailofbits/publications/blob/master/reviews/YieldProtocol.pdf)
        
43.  **Lack of** _**chainID**_ **validation allows signatures to be re-used across forks**: _YDai_ implements the draft ERC 2612 via the _ERC20Permit_ contract it inherits from. This allows a third party to transmit a signature from a token holder that modifies the ERC20 allowance for a particular user. These signatures used in calls to permit in _ERC20Permit_ do not account for chain splits. The _chainID_ is included in the domain separator. However, it is not updatable and not included in the signed data as part of the permit call. As a result, if the chain forks after deployment, the signed message may be considered valid on both forks.
    
    1.  Recommendation: Short term, include the _chainID_ opcode in the permit schema. This will make replay attacks impossible in the event of a post-deployment hard fork. Long term, document and carefully review any signature schemas, including their robustness to replay on different wallets, contracts, and blockchains. Make sure users are aware of signing best practices and the danger of signing messages from untrusted sources.
        
    2.  High Risk severity finding from [ToB’s Audit of Yield Protocol](https://github.com/trailofbits/publications/blob/master/reviews/YieldProtocol.pdf)
        
44.  **Lack of a contract existence check allows token theft**: Since there’s no existence check for contracts that interact with external tokens, an attacker can steal funds by registering a token that’s not yet deployed. __safeTransferFrom_ will return success even if the token is not yet deployed, or was self-destructed. An attacker that knows the address of a future token can register the token in Hermez, and deposit any amount prior to the token deployment. Once the contract is deployed and tokens have been deposited in Hermez, the attacker can steal the funds. The address of a contract to be deployed can be determined by knowing the address of its deployer.
    
    1.  Recommendation: Short term, check for contract existence in _ _safeTransferFrom_. Add a similar check for any low-level calls, including in _WithdrawalDelayer_. This will prevent an attacker from listing and depositing tokens in a contract that is not yet deployed. Long term, carefully review the Solidity documentation, especially the Warnings section. The Solidity documentation warns: The low-level call, _delegatecall_ and _callcode_ will return success if the called account is non-existent, as part of the design of EVM. Existence must be checked prior to calling if desired.
        
    2.  High Risk severity finding from [ToB’s Audit of Hermez](https://github.com/trailofbits/publications/blob/master/reviews/hermez.pdf)
        
45.  **No incentive for bidders to vote earlier**: Hermez relies on a voting system that allows anyone to vote with any weight at the last minute. As a result, anyone with a large fund can manipulate the vote. Hermez’s voting mechanism relies on bidding. There is no incentive for users to bid tokens well before the voting ends. Users can bid a large amount of tokens just before voting ends, and anyone with a large fund can decide the outcome of the vote. As all the votes are public, users bidding earlier will be penalized, because their bids will be known by the other participants. An attacker can know exactly how much currency will be necessary to change the outcome of the voting just before it ends.
    
    1.  Recommendation: Short term, explore ways to incentivize users to vote earlier. Consider a weighted bid, with a weight decreasing over time. While it won’t prevent users with unlimited resources from manipulating the vote at the last minute, it will make the attack more expensive and reduce the chance of vote manipulation. Long term, stay up to date with the latest research on blockchain-based online voting and bidding. Blockchain-based online voting is a known challenge. No perfect solution has been found yet.
        
    2.  Medium Risk severity finding from [ToB’s Audit of Hermez](https://github.com/trailofbits/publications/blob/master/reviews/hermez.pdf)
        
46.  **Lack of access control separation is risky**: The system uses the same account to change both frequently updated parameters and those that require less frequent updates. This architecture is error-prone and increases the severity of any privileged account compromises.
    
    1.  Recommendation: Short term, use a separate account to handle updating the tokens/USD ratio. Using the same account for the critical operations and update the tokens/USD ratio increases underlying risks. Long term, document the access controls and set up a proper authorization architecture. Consider the risks associated with each access point and their frequency of usage to evaluate the proper design.
        
    2.  High Risk severity finding from [ToB’s Audit of Hermez](https://github.com/trailofbits/publications/blob/master/reviews/hermez.pdf)
        
47.  **Lack of two-step procedure for critical operations leaves them error-prone**: Several critical operations are done in one function call. This schema is error-prone and can lead to irrevocable mistakes. For example, the setter for the whitehack group address sets the address to the provided argument. If the address is incorrect, the new address will take on the functionality of the new role immediately. However, a two-step process is similar to the approve-transferFrom functionality: The contract approves the new address for a new role, and the new address acquires the role by calling the contract.
    
    1.  Recommendation: Short term, use a two-step procedure for all non-recoverable critical operations to prevent irrecoverable mistakes. Long term, identify and document all possible actions and their associated risks for privileged accounts. Identifying the risks will assist codebase review and prevent future mistakes.
        
    2.  High Risk severity finding from [ToB’s Audit of Hermez](https://github.com/trailofbits/publications/blob/master/reviews/hermez.pdf)
        
48.  **Initialization functions can be front-run**: _Hermez_, _HermezAuctionProtocol_, and _WithdrawalDelayer_ have initialization functions that can be front-run, allowing an attacker to incorrectly initialize the contracts. Due to the use of the _delegatecall_ proxy pattern, _Hermez_, _HermezAuctionProtocol_, and _WithdrawalDelayer_ cannot be initialized with a constructor, and have initializer functions. All these functions can be front-run by an attacker, allowing them to initialize the contracts with malicious values.
    
    1.  Recommendation: Short term, either: 1) Use a factory pattern that will prevent front-running of the initialization, or 2) Ensure the deployment scripts are robust in case of a front-running attack. Carefully review the Solidity documentation, especially the Warnings section. Carefully review the pitfalls of using delegatecall proxy pattern.
        
    2.  High Risk severity finding from [ToB’s Audit of Hermez](https://github.com/trailofbits/publications/blob/master/reviews/hermez.pdf)
        
49.  **Missing validation of** _**_owner**_ **argument could indefinitely lock owner role**: A lack of input validation of the __owner_ argument in both the constructor and _setOwner_ functions could permanently lock the owner role, requiring a costly redeploy. To resolve an incorrect owner issue, Uniswap would need to redeploy the factory contract and re-add pairs and liquidity. Users might not be happy to learn of these actions, which could lead to reputational damage. Certain users could also decide to continue using the original factory and pair contracts, in which owner functions cannot be called. This could lead to the concurrent use of two versions of Uniswap, one with the original factory contract and no valid owner and another in which the owner was set correctly. Trail of Bits identified four distinct cases in which an incorrect owner is set: 1) Passing address(0) to the constructor 2) Passing address(0) to the _setOwner_ function 3) Passing an incorrect address to the constructor 4)  Passing an incorrect address to the _setOwner_ function.
    
    1.  Recommendation: Several improvements could prevent the four above mentioned cases: 1) Designate _msg.sender_ as the initial owner, and transfer ownership to the chosen owner after deployment. 2) Implement a two-step ownership-change process through which the new owner needs to accept ownership. 3) If it needs to be possible to set the owner to address(0), implement a _renounceOwnership_ function.
        
    2.  Medium Risk severity finding from [ToB’s Audit of Uniswap V3](https://github.com/Uniswap/uniswap-v3-core/blob/main/audits/tob/audit.pdf)
        
50.  **Incorrect comparison enables swapping and token draining at no cost**: An incorrect comparison in the swap function allows the swap to succeed even if no tokens are paid. This issue could be used to drain any pool of all of its tokens at no cost. The swap function calculates how many tokens the initiator (_msg.sender_) needs to pay (_amountIn_) to receive the requested amount of tokens (_amountOut_). It then calls the _uniswapV3SwapCallback_ function on the initiator’s account, passing in the amount of tokens to be paid. The callback function should then transfer at least the requested amount of tokens to the pool contract. Afterward, a require inside the swap function verifies that the correct amount of tokens (amountIn) has been transferred to the pool. However, the check inside the require is incorrect. The operand used is >= instead of <=.
    
    1.  Recommendation: Replace >= with <= in the require statement.
        
    2.  High Risk severity finding from [ToB’s Audit of Uniswap V3](https://github.com/Uniswap/uniswap-v3-core/blob/main/audits/tob/audit.pdf)
        
51.  **Unbound loop enables denial of service**: The swap function relies on an unbounded loop. An attacker could disrupt swap operations by forcing the loop to go through too many operations, potentially trapping the swap due to a lack of gas.
    
    1.  Recommendation: Bound the loops and document the bounds.
        
    2.  Medium Risk severity finding from [ToB’s Audit of Uniswap V3](https://github.com/Uniswap/uniswap-v3-core/blob/main/audits/tob/audit.pdf)
        
52.  **Front-running pool’s initialization can lead to draining of liquidity provider’s initial deposits**: A front-run on _UniswapV3Pool.initialize_ allows an attacker to set an unfair price and to drain assets from the first deposits. There are no access controls on the initialize function, so anyone could call it on a deployed pool. Initializing a pool with an incorrect price allows an attacker to generate profits from the initial liquidity provider’s deposits.
    
    1.  Recommendation: 1) moving the price operations from initialize to the constructor, 2) adding access controls to initialize, or 3) ensuring that the documentation clearly warns users about incorrect initialization.
        
    2.  Medium Risk severity finding from [ToB’s Audit of Uniswap V3](https://github.com/Uniswap/uniswap-v3-core/blob/main/audits/tob/audit.pdf)
        
53.  **Swapping on zero liquidity allows for control of the pool’s price**: Swapping on a tick with zero liquidity enables a user to adjust the price of 1 wei of tokens in any direction. As a result, an attacker could set an arbitrary price at the pool’s initialization or if the liquidity providers withdraw all of the liquidity for a short time.
    
    1.  Recommendation: No straightforward way to prevent the issue. Ensure pools don’t end up in unexpected states. Warn users of potential risks.
        
    2.  Medium Risk severity finding from [ToB’s Audit of Uniswap V3](https://github.com/Uniswap/uniswap-v3-core/blob/main/audits/tob/audit.pdf)
        
54.  **Failed transfer may be overlooked due to lack of contract existence check**: Because the pool fails to check that a contract exists, the pool may assume that failed transactions involving destructed tokens are successful. _TransferHelper.safeTransfer_ performs a transfer with a low-level call without confirming the contract’s existence. As a result, if the tokens have not yet been deployed or have been destroyed, safeTransfer will return success even though no transfer was executed.
    
    1.  Recommendation: Short term, check the contract’s existence prior to the low-level call in _TransferHelper.safeTransfer_. Long term, avoid low-level calls.
        
    2.  High Risk severity finding from [ToB’s Audit of Uniswap V3](https://github.com/Uniswap/uniswap-v3-core/blob/main/audits/tob/audit.pdf)
        
55.  **Use of undefined behavior in equality check**: On the left-hand side of the equality check, there is an assignment of the variable _outputAmt__. The right-hand side uses the same variable. The Solidity 0.7.3. documentation states that “The evaluation order of expressions is not specified (more formally, the order in which the children of one node in the expression tree are evaluated is not specified, but they are of course evaluated before the node itself). It is only guaranteed that statements are executed in order and short-circuiting for boolean expressions is done” which means that this check constitutes an instance of undefined behavior. As such, the behavior of this code is not specified and could change in a future release of Solidity.
    
    1.  Recommendation: Short term, rewrite the if statement such that it does not use and assign the same variable in an equality check. Long term, ensure that the codebase does not contain undefined Solidity or EVM behavior.
        
    2.  High Risk severity finding from [ToB’s Audit of DFX Finance](https://github.com/dfx-finance/protocol/blob/main/audits/2021-05-03-Trail_of_Bits.pdf)
        
56.  **Assimilators’ balance functions return raw values**: The system converts raw values to numeraire values for its internal arithmetic. However, in one instance it uses raw values alongside numeraire values. Interchanging raw and numeraire values will produce unwanted results and may result in loss of funds for liquidity provider.
    
    1.  Recommendation: Short term, change the semantics of the three functions listed above in the CADC, XSGD, and EURS assimilators to return the numeraire balance. Long term, use unit tests and fuzzing to ensure that all calculations return the expected values. Additionally, ensure that changes to the Shell Protocol do not introduce bugs such as this one.
        
    2.  High Risk severity finding from [ToB’s Audit of DFX Finance](https://github.com/dfx-finance/protocol/blob/main/audits/2021-05-03-Trail_of_Bits.pdf)
        
57.  **System always assumes USDC is equivalent to USD**: Throughout the system, assimilators are used to facilitate the processing of various stablecoins. However, the _UsdcToUsdAssimilator_’s implementation of the _getRate_ method does not use the USDC-USD oracle provided by Chainlink; instead, it assumes 1 USDC is always worth 1 USD. A deviation in the exchange rate of 1 USDC = 1 USD could result in exchange errors.
    
    1.  Recommendation: Short term, replace the hard-coded integer literal in the _UsdcToUsdAssimilator_’s _getRate_ method with a call to the relevant Chainlink oracle, as is done in other assimilator contracts. Long term, ensure that the system is robust against a decrease in the price of any stablecoin.
        
    2.  Medium Risk severity finding from [ToB’s Audit of DFX Finance](https://github.com/dfx-finance/protocol/blob/main/audits/2021-05-03-Trail_of_Bits.pdf)
        
58.  **Assimilators use a deprecated Chainlink API**: The old version of the Chainlink price feed API (_AggregatorInterface_) is used throughout the contracts and tests. For example, the deprecated function _latestAnswer_ is used. This function is not present in the latest API reference (_AggregatorInterfaceV3_). However, it is present in the deprecated API reference. In the worst-case scenario, the deprecated contract could cease to report the latest values, which would very likely cause liquidity providers to incur losses.
    
    1.  Recommendation: Use the latest stable versions of any external libraries or contracts leveraged by the codebase
        
    2.  Undetermined Risk severity finding from [ToB’s Audit of DFX Finance](https://github.com/dfx-finance/protocol/blob/main/audits/2021-05-03-Trail_of_Bits.pdf)
        
59.  _**cancelOrdersUpTo**_ **can be used to permanently block future orders**: Users can cancel an arbitrary number of future orders, and this operation is not reversible. The _cancelOrdersUpTo_ function (Figure 3.1) can cancel an arbitrary number of orders in a single, fixed-size transaction. This function uses a parameter to discard any order with salt less than the input value. However, _cancelOrdersUpTo_ can cancel future orders if it is called with a very large value (e.g., _MAX_UINT256_ - 1). This operation will cancel future orders, except for the one with salt equal to _MAX_UINT256_.
    
    1.  Recommendation: Properly document this behavior to warn users about the permanent effects of _cancelOrderUpTo_ on future orders. Alternatively, disallow the cancelation of future orders.
        
    2.  High Risk severity finding from [ToB’s Audit of 0x Protocol](https://github.com/trailofbits/publications/blob/master/reviews/0x-protocol.pdf)
        
60.  **Specification-Code mismatch for** _**AssetProxyOwner**_ **timelock period**: The specification for _AssetProxyOwner_ says: "The _AssetProxyOwner_ is a time-locked multi-signature wallet that has permission to perform administrative functions within the protocol. Submitted transactions must pass a 2 week timelock before they are executed." The _MultiSigWalletWithTimeLock_._sol_ and _AssetProxyOwner_._sol_ contracts' timelock-period implementation/usage does not enforce the two-week period, but is instead configurable by the wallet owner without any range checks. Either the specification is outdated (most likely), or this is a serious flaw.
    
    1.  Recommendation: Short term, implement the necessary range checks to enforce the timelock described in the specification. Otherwise correct the specification to match the intended behavior. Long term, make sure implementation and specification are in sync. Use Echidna or Manticore to test that your code properly implements the specification.
        
    2.  High Risk severity finding from [ToB’s Audit of 0x Protocol](https://github.com/trailofbits/publications/blob/master/reviews/0x-protocol.pdf)
        
61.  **Unclear documentation on how order filling can fail**: The 0x documentation is unclear about how to determine whether orders are fillable or not. Even some fillable orders cannot be completely filled. The 0x specification does not state clearly enough how fillable orders are determined.
    
    1.  Recommendation: Define a proper procedure to determine if an order is fillable and document it in the protocol specification. If necessary, warn the user about potential constraints on the orders.
        
    2.  High Risk severity finding from [ToB’s Audit of 0x Protocol](https://github.com/trailofbits/publications/blob/master/reviews/0x-protocol.pdf)
        
62.  **Market makers have a reduced cost for performing front-running attacks**: Market makers receive a portion of the protocol fee for each order filled, and the protocol fee is based on the transaction gas price. Therefore market makers are able to specify a higher gas price for a reduced overall transaction rate, using the refund they will receive upon disbursement of protocol fee pools.
    
    1.  Recommendation: Short term, properly document this issue to make sure users are aware of this risk. Establish a reasonable cap for the protocolFeeMultiplier to mitigate this issue. Long term, consider using an alternative fee that does not depend on the tx.gasprice to avoid reducing the cost of performing front-running attacks.
        
    2.  Medium Risk severity finding from [ToB’s Audit of 0x Protocol](https://github.com/trailofbits/publications/blob/master/reviews/0x-protocol.pdf)
        
63.  _**setSignatureValidatorApproval**_ **race condition may be exploitable**: If a validator is compromised, a race condition in the signature validator approval logic becomes exploitable. The _setSignatureValidatorApproval_ function (Figure 4.1) allows users to delegate the signature validation to a contract. However, if the validator is compromised, a race condition in this function could allow an attacker to validate any amount of malicious transactions.
    
    1.  Recommendation: Short term, document this behavior to make sure users are aware of the inherent risks of using validators in case of a compromise. Long term, consider monitoring the blockchain using the _SignatureValidatorApproval_ events to catch front-running attacks.
        
    2.  Medium Risk severity finding from [ToB’s Audit of 0x Protocol](https://github.com/trailofbits/publications/blob/master/reviews/0x-protocol.pdf)
        
64.  **Batch processing of transaction execution and order matching may lead to exchange griefing**: Batch processing of transaction execution and order matching will iteratively process every transaction and order, which all involve filling. If the asset being filled does not have enough allowance, the asset’s transferFrom will fail, causing _AssetProxyDispatcher_ to revert. NoThrow variants of batch processing, which are available for filling orders, are not available for transaction execution and order matching. So if one transaction or order fails this way, the entire batch will revert and will have to be re-submitted after the reverting transaction is removed.
    
    1.  Recommendation: Short term, implement NoThrow variants for batch processing of transaction execution and order matching. Long term, take into consideration the effect of malicious inputs when implementing functions that perform a batch of operations.
        
    2.  Medium Risk severity finding from [ToB’s Audit of 0x Protocol](https://github.com/trailofbits/publications/blob/master/reviews/0x-protocol.pdf)
        
65.  **Zero fee orders are possible if a user performs transactions with a zero gas price**: Users can submit valid orders and avoid paying fees if they use a zero gas price. The computation of fees for each transaction is performed in the _calculateFillResults_ function. It uses the gas price selected by the user and the _protocolFeeMultiplier_ coefficient. Since the user completely controls the gas price of their transaction and the price could even be zero, the user could feasibly avoid paying fees.
    
    1.  Recommendation: Short term, select a reasonable minimum value for the protocol fee for each order or transaction. Long term, consider not depending on the gas price for the computation of protocol fees. This will avoid giving miners an economic advantage in the system.
        
    2.  Medium Risk severity finding from [ToB’s Audit of 0x Protocol](https://github.com/trailofbits/publications/blob/master/reviews/0x-protocol.pdf)
        
66.  **Calls to** _**setParams**_ **may set invalid values and produce unexpected behavior in the staking contracts**: Certain parameters of the contracts can be configured to invalid values, causing a variety of issues and breaking expected interactions between contracts. _setParams_ allows the owner of the staking contracts to reparameterize critical parameters. However, reparameterization lacks sanity/threshold/limit checks on all parameters.
    
    1.  Recommendation: Add proper validation checks on all parameters in _setParams_. If the validation procedure is unclear or too complex to implement on-chain, document the potential issues that could produce invalid values.
        
    2.  Medium Risk severity finding from [ToB’s Audit of 0x Protocol](https://github.com/trailofbits/publications/blob/master/reviews/0x-protocol.pdf)
        
67.  **Improper Supply Cap Limitation Enforcement**: The _openLoan()_ function does not check if the loan to be issued will result in the supply cap being exceeded. It only enforces that the supply cap is not reached before the loan is opened. As a result, any account can create a loan that exceeds the maximum amount of sETH that can be issued by the _EtherCollateral_ contract.
    
    1.  Recommendation: Introduce a require statement in the _openLoan()_ function to prevent the total cap from being exceeded by the loan to be opened.
        
    2.  High Risk severity finding from [Sigma Prime's Audit of Synthetix EtherCollateral](https://github.com/sigp/public-audits/blob/master/synthetix/ethercollateral/review.pdf)
        
68.  **Improper Storage Management of Open Loan Accounts**: When loans are open, the associated account address gets added to the _accountsWithOpenLoans_ array regardless of whether the account already has a loan/is already included in the array. Additionally, it is possible for a malicious actor to create a denial of service condition exploiting the unbound storage array in _accountsSynthLoans_. 
    
    1.  Recommendation: 1) Consider changing the _storeLoan_ function to only push the account to the _accountsWithOpenLoans_ array if the loan to be stored is the first one for that particular account ; 2) Introduce a limit to the number of loans each account can have.
        
    2.  High Risk severity finding from [Sigma Prime's Audit of Synthetix EtherCollateral](https://github.com/sigp/public-audits/blob/master/synthetix/ethercollateral/review.pdf)
        
69.  **Contract Owner Can Arbitrarily Change Minting Fees and Interest Rates**: The _issueFeeRate_ and _interestRate_ variables can both be changed by the _EtherCollateral_ contract owner after loans have been opened. As a result, the owner can control fees such as they equal/exceed the collateral for any given loan.
    
    1.  Recommendation: While "dynamic" interest rates are common, we recommend considering the minting fee ( _issueFeeRate_ ) to be a constant that cannot be changed by the owner.
        
    2.  Medium Risk severity finding from [Sigma Prime's Audit of Synthetix EtherCollateral](https://github.com/sigp/public-audits/blob/master/synthetix/ethercollateral/review.pdf)
        
70.  **Inadequate Proxy Implementation Preventing Contract Upgrades**: The _TokenImpl_ smart contract requires Owner , name , symbol and decimals of _TokenImpl_ to be set by the _TokenImpl_ constructor. Consider two smart contracts, contract A and contract B . If contract A performs a delegatecall on contract B , the state/storage variables of contract B are not accessible by contract A . Therefore, when _TokenProxy_ targets an implementation of _TokenImpl_ and interacts with it via a _DELEGATECALL_ , it will not be able to access any of the state variables of the _TokenImpl_ contract. Instead, the _TokenProxy_ will access its local storage, which does not contain the variables set in the constructor of the _TokenImpl_ implementation. When the _TokenProxy_ contract is constructed it will only initialize and set two storage slots: • The proxy admin address ( __setAdmin_ internal function) • The token implementation address ( __setImplementation_ private function) Hence when a proxy call to the implementation is made, variables such as _Owner_ will be uninitialised (effectively set to their default value). This is equivalent to the owner being the 0x0 address. Without access to the implementation state variables, the proxy contract is rendered unusable.
    
    1.  Recommendation: 1) Set fixed constant parameters as Solidity constants. The solidity compiler replaces all occurrences of a constant in the code and thus does not reserve state for them. Thus if the correct getters exist for the ERC20 interface, the proxy contract doesn’t need to initialise anything. 2) Create a constructor-like function that can only be called once within _TokenImpl_ . This can be used to set the state variables as is currently done in the constructor, however if called by the proxy after deployment, the proxy will set its state variables. 3) Create getter and setter functions that can only be called by the owner . Note that this strategy allows the owner to change various parameters of the contract after deployment. 4) Predetermine the slots used by the required variables and set them in the constructor of the proxy. The storage slots used by a contract are deterministic and can be computed. Hence the variables Owner , name , symbol and decimals can be set directly by their slot in the proxy constructor.
        
    2.  Critical Risk severity finding from [Sigma Prime's Audit of InfiniGold](https://github.com/sigp/public-audits/raw/master/infinigold/review.pdf)
        
71.  **Blacklisting Bypass via** _**transferFrom()**_ **Function**: The _transferFrom()_ function in the _TokenImpl_ contract does not verify that the sender (i.e. the from address) is not blacklisted. As such, it is possible for a user to allow an account to spend a certain allowance regardless of their blacklisting status.
    
    1.  Recommendation: At present the function _transferFrom()_ uses the _notBlacklisted(address)_ modifier twice, on the msg.sender and to addresses. The _notBlacklisted(address)_ modifier should be used a third time against the from address.
        
    2.  High Risk severity finding from [Sigma Prime's Audit of InfiniGold](https://github.com/sigp/public-audits/raw/master/infinigold/review.pdf)
        
72.  **Wrong Order of Operations Leads to Exponentiation of** _**rewardPerTokenStored**_: _rewardPerTokenStored_ is mistakenly used in the numerator of a fraction instead of being added to the fraction. The result is that _rewardPerTokenStored_ will grow exponentially thereby severely overstating each individual's rewards earned. Individuals will therefore either be able to withdraw more funds than should be allocated to them or they will not be able to withdraw their funds at all as the contract has insufficient SNX balance. This vulnerability makes the Unipool contract unusable.
    
    1.  Recommendation: Adjust the function _rewardPerToken()_ to represent the original functionality.
        
    2.  Critical Risk severity finding from [Sigma Prime's Audit of Synthetix Unipool](https://github.com/sigp/public-audits/blob/master/synthetix/unipool/review.pdf)
        
73.  **Staking Before Initial notifyRewardAmount Can Lead to Disproportionate Rewards**: If a user successfully stakes an amount of UNI tokens before the function _notifyRewardAmount()_ is called for the first time, their initial _userRewardPerTokenPaid_ will be set to zero. The staker would be paid out funds greater than their share of the SNX rewards.
    
    1.  Recommendation: We recommend preventing _stake()_ from being called before _notifyRewardAmount()_ is called for the first time.
        
    2.  High Risk severity finding from [Sigma Prime's Audit of Synthetix Unipool](https://github.com/sigp/public-audits/blob/master/synthetix/unipool/review.pdf)
        
74.  **External Call Reverts if Period Has Not Elapsed**: The function _notifyRewardAmount()_ will revert if _block.timestamp >= periodFinish_. However this function is called indirectly via the _Synthetix.mint()_ function. A revert here would cause the external call to fail and thereby halt the mint process. _Synthetix.mint()_ cannot be successfully called until enough time has elapsed for the period to finish.
    
    1.  Recommendation: Consider handling the case where the reward period has not elapsed without reverting the call.
        
    2.  High Risk severity finding from [Sigma Prime's Audit of Synthetix Unipool](https://github.com/sigp/public-audits/blob/master/synthetix/unipool/review.pdf)
        
75.  **Gap Between Periods Can Lead to Erroneous Rewards**: The SNX rewards are earned each period based on reward and duration as specified in the _notifyRewardAmount()_ function. The contract will output more rewards than it receives. Therefore if all stakers call _getReward()_ the contract will not have enough SNX balance to transfer out all the rewards and some stakers may not receive any rewards.
    
    1.  Recommendation: We recommend enforcing each period start exactly at the end of the previous period.
        
    2.  Medium Risk severity finding from [Sigma Prime's Audit of Synthetix Unipool](https://github.com/sigp/public-audits/blob/master/synthetix/unipool/review.pdf)
        
76.  **Malicious Users Can DOS/Hijack Requests From Chainlinked Contracts**: Malicious users can hijack or perform Denial of Service (DOS) attacks on requests of Chainlinked contracts by replicating or front-running legitimate requests. The Chainlinked (_Chainlinked.sol_) contract contains the _checkChainlinkFulfillment_() modifier. This modifier is demonstrated in the examples that come with the repository. In these examples this modifier is used within the functions which contracts implement that will be called by the Oracle when fulfilling requests. It requires that the caller of the function be the Oracle that corresponds to the request that is being fulfilled. Thus, requests from Chainlinked contracts are expected to only be fulfilled by the Oracle that they have requested. However, because a request can specify an arbitrary callback address, a malicious user can also place a request where the callback address is a target Chainlinked contract. If this malicious request gets fulfilled first (which can ask for incorrect or malicious results), the Oracle will call the legitimate contract and fulfil it with incorrect or malicious results. Because the known requests of a Chainlinked contract gets deleted, the legitimate request will fail. It could be such that the Oracle fulfils requests in the order in which they are received. In such cases, the malicious user could simply front-run the requests to be higher in the queue.
    
    1.  Recommendation: This issue arises due to the fact that any request can specify its own arbitrary callback address. A restrictive solution would be where callback addresses are localised to the requester themselves.
        
    2.  High Risk severity finding from [Sigma Prime's Audit of Chainlink](https://github.com/sigp/public-audits/blob/master/chainlink-1/review.pdf)
        
77.  **Lack of event emission after sensitive actions**: The __getLatestFundingRate_ function of the _FundingRateApplier_ contract does not emit relevant events after executing the sensitive actions of setting the _fundingRate_, _updateTime_ and _proposalTime_, and transferring the rewards.
    
    1.  Recommendation: Consider emitting events after sensitive changes take place, to facilitate tracking and notify off-chain clients following the contract’s activity.
        
    2.  Medium Risk severity finding from [OpenZeppelin’s Audit of UMA Phase 4](https://blog.openzeppelin.com/uma-audit-phase-4/)
        
78.  **Functions with unexpected side-effects**: Some functions have side-effects. For example, the __getLatestFundingRate_ function of the _FundingRateApplier_ contract might also update the funding rate and send rewards. The _getPrice_ function of the OptimisticOracle contract might also settle a price request. These side-effect actions are not clear in the name of the functions and are thus unexpected, which could lead to mistakes when the code is modified by new developers not experienced in all the implementation details of the project.
    
    1.  Recommendation: Consider splitting these functions in separate getters and setters. Alternatively, consider renaming the functions to describe all the actions that they perform.
        
    2.  Medium Risk severity finding from [OpenZeppelin’s Audit of UMA Phase 4](https://blog.openzeppelin.com/uma-audit-phase-4/)
        
79.  **Mooniswap pairs cannot be unpaused**: The _MooniswapFactoryGovernance_ contract has a shutdown function that can be used to pause the contract and prevent any future swaps. However there is no function to unpause the contract. There is also no way for the factory contract to redeploy a Mooniswap instance for a given pair of tokens. Therefore, if a Mooniswap contract is ever shutdown/paused, it will not be possible for that pair of tokens to ever be traded on the Mooniswap platform again, unless a new factory contract is deployed.
    
    1.  Recommendation: Consider providing a way for Mooniswap contracts to be unpaused.
        
    2.  Medium Risk severity finding from [OpenZeppelin’s Audit of 1inch Liquidity Protocol Audit](https://blog.openzeppelin.com/1inch-liquidity-protocol-audit/)
        
80.  **Attackers can prevent honest users from performing an instant withdraw from the Wallet contract**: An attacker who sees an honest user’s call to _MessageProcessor.instantWithdraw_ in the mempool can grab the _oracleMessage_ and _oracleSignature_ parameters from the user’s transaction, then submit their own transaction to _instantWithdraw_ using the same parameters, a higher gas price (so as to frontrun the honest user’s transaction), and carefully choosing the gas limit for their transactions such that the internal call to the _callInstantWithdraw_ will fail on line 785 with an out-of-gas error, but will successfully execute the _if(!success)_ block. The result is that the attacker’s instant withdraw will fail (so the user will not receive their funds), but the _userInteractionNumber_ will be successfully reserved by the _ReplayTracker_. As a result, the honest user’s transaction will revert because it will be attempting to use a _userInteractionNumber_ that is no longer valid.
    
    1.  Recommendation: Consider adding an access control mechanism to restrict who can submit _oracleMessages_ on behalf of the user.
        
    2.  High Risk severity finding from [OpenZeppelin’s Audit of Futureswap V2](https://blog.openzeppelin.com/futureswap-v2-audit/)
        
81.  **Not using upgrade safe contracts in** _**FsToken**_ **inheritance**: The _FsToken_ contract is intended to be an upgradeable contract, used behind a proxy (namely, the _FsTokenProxy_ contract). However, the contracts _ERC20Snapshot_, _ERC20Mintable_ and _ERC20Burnable_ in the inheritance chain of FsToken are not imported from the upgrade safe library _@openzeppelin/contracts-ethereum-package_ but instead from _@openzeppelin/contracts_.
    
    1.  Recommendation: Use the upgrades safe library in this case will ensure the inheritance from Initializable and the other contracts is always linearized as expected by the compiler.
        
    2.  Medium Risk severity finding from [OpenZeppelin’s Audit of Futureswap V2](https://blog.openzeppelin.com/futureswap-v2-audit/)
        
82.  **Unchecked output of the ECDSA recover function**: The _ECDSA.recover_ function (in version 2.5.1) returns address(0) if the signature provided is invalid. This function is used twice in the Futureswap code: Once to recover an _oracleAddress_ from an _oracleSignature_, and again to recover the user’s address from their signature. If the oracle signature was invalid, the _oracleAddress_ is set to address(0). Similarly, if the user’s signature is invalid, then the _userMessage.signer_ or the withDrawer is set to address(0). This can result in unintended behavior.
    
    1.  Recommendation: Consider reverting if the output of the _ECDSA.recover_ is ever address(0)
        
    2.  Medium Risk severity finding from [OpenZeppelin’s Audit of Futureswap V2](https://blog.openzeppelin.com/futureswap-v2-audit/)
        
83.  **Adding new variables to multi-level inherited upgradeable contracts may break storage layout**: The Notional protocol uses the OpenZeppelin/SDK contracts to manage upgradeability in the system, which follows the unstructured storage pattern. When using this upgradeability approach, and when working with multi-level inheritance, if a new variable is introduced in a parent contract, that addition can potentially overwrite the beginning of the storage layout of the child contract, causing critical misbehaviors in the system.
    
    1.  Recommendation: consider preventing these scenarios by defining a storage gap in each upgradeable parent contract at the end of all the storage variable definitions as follows: _uint256[50] __gap; // gap to reserve storage in the contract for future variable additions._ In such an implementation, the size of the gap would be intentionally decreased each time a new variable was introduced, thereby avoiding overwriting preexisting storage values.
        
    2.  Medium Risk severity finding from [OpenZeppelin’s Audit of Notional Protocol](https://blog.openzeppelin.com/notional-audit/)
        
84.  **Unsafe division in** _**rdivide**_ **and** _**wdivide**_ **functions**: The function _rdivide_ on line 227 and the function _wdivide_ on line 230 of the _GlobalSettlement_ contract, accept the divisor y as an input parameter. However, these functions do not check if the value of y is 0. If that is the case, the call will revert due to the division by zero error.
    
    1.  Recommendation: consider adding a _require_ statement in the functions to ensure _y > 0_, or consider using the _div_ functions provided in OpenZeppelin’s SafeMath libraries
        
    2.  Medium Risk severity finding from [OpenZeppelin’s Audit of GEB Protocol](https://blog.openzeppelin.com/geb-protocol-audit/)
        
85.  **Incorrect** _**safeApprove**_ **usage**: The _safeApprove_ function of the OpenZeppelin SafeERC20 library prevents changing an allowance between non-zero values to mitigate a possible front-running attack. Instead, the _safeIncreaseAllowance_ and _safeDecreaseAllowance_ functions should be used. However, the _UniERC20_ library simply bypasses this restriction by first setting the allowance to zero. This reintroduces the front-running attack and undermines the value of the _safeApprove_ function. Consider introducing an _increaseAllowance_ function to handle this case.
    
    1.  Recommendation: _safeIncreaseAllowance_ and _safeDecreaseAllowance_ functions should be used
        
    2.  Medium Risk severity finding from [OpenZeppelin’s Audit of 1inch Exchange Audit](https://blog.openzeppelin.com/1inch-exchange-audit/)
        
86.  **ETH could get trapped in the protocol**: The Controller contract allows users to send arbitrary actions such as possible flash loans through the __call_ internal function. Among other features, it allows sending ETH with the action to then perform a call to a _CalleeInterface_ type of contract. To do so, it saves the original _msg.value_ sent with the operate function call in the _ethLeft_ variable and it updates the remaining ETH left after each one of those calls to revert in case that it is not enough. Nevertheless, if the user sends more than the necessary ETH for the batch of actions, the remaining ETH (stored in the ethLeft variable after the last iteration) will not be returned to the user and will be locked in the contract due to the lack of a _withdrawEth_ function.
    
    1.  Recommendation: Consider either returning all the remaining ETH to the user or creating a function that allows the user to collect the remaining ETH after performing a Call action type, taking into account that sending ETH with a push method may trigger the fallback function on the caller’s address.
        
    2.  High Risk severity finding from [OpenZeppelin’s Audit of Opyn Gamma Protocol](https://blog.openzeppelin.com/opyn-gamma-protocol-audit/)
        
87.  **Use of transfer might render ETH impossible to withdraw**: When withdrawing ETH deposits, the _PayableProxyController_ contract uses Solidity’s _transfer_ function. This has some notable shortcomings when the withdrawer is a smart contract, which can render ETH deposits impossible to withdraw. Specifically, the withdrawal will inevitably fail when: 1) The withdrawer smart contract does not implement a payable fallback function. 2) The withdrawer smart contract implements a payable fallback function which uses more than 2300 gas units. 3) The withdrawer smart contract implements a payable fallback function which needs less than 2300 gas units but is called through a proxy that raises the call’s gas usage above 2300.
    
    1.  Recommendation:  _sendValue_ function available in OpenZeppelin Contract’s Address library can be used to transfer the withdrawn Ether without being limited to 2300 gas units. Risks of reentrancy stemming from the use of this function can be mitigated by tightly following the “Check-effects-interactions” pattern and using OpenZeppelin Contract’s _ReentrancyGuard_ contract.
        
    2.  Medium Risk severity finding from [OpenZeppelin’s Audit of Opyn Gamma Protocol](https://blog.openzeppelin.com/opyn-gamma-protocol-audit/)
        
88.  **Not following the Checks-Effects-Interactions pattern**: The _finalizeGrant_ function of the Fund contract is setting the _grant.complete_ storage variable after a token transfer. Solidity recommends the usage of the Check-Effects-Interaction Pattern to avoid potential security issues, such as reentrancy. The _finalizeGrant_ function can be used to conduct a reentrancy attack, where the token transfer in line 129 can call back again the same function, sending to the admin multiple times an amount of fee, before setting the grant as completed. In this way the grant.recipient can receive less than expected and the contract funds can be drained unexpectedly leading to an unwanted loss of funds.
    
    1.  Recommendation: Consider always following the “Check-Effects-Interactions” pattern, thus modifying the contract’s state before making any external call to other contracts.
        
    2.  High Risk severity finding from [OpenZeppelin’s Audit of Endaoment](https://blog.openzeppelin.com/endaoment-audit/)
        
89.  **Updating the Governance registry and Guardian addresses emits no events**: In the Governance contract the _registryAddress_ and the _guardianAddress_ are highly sensitive accounts. The first one holds the contracts that can be proposal targets, and the second one is a superuser account that can execute proposals without voting. These variables can be updated by calling _setRegistryAddress_ and _transferGuardianship_, respectively. Note that these two functions update these sensitive addresses without logging any events. Stakers who monitor the Audius system would have to inspect all transactions to notice that one address they trust is replaced with an untrusted one.
    
    1.  Recommendation: Consider emitting events when these addresses are updated. This will be more transparent, and it will make it easier for clients to subscribe to the events when they want to keep track of the status of the system.
        
    2.  High Risk severity finding from [OpenZeppelin’s Audit of Audius](https://blog.openzeppelin.com/audius-contracts-audit/#high)
        
90.  **The quorum requirement can be trivially bypassed with sybil accounts**: While the final vote on a proposal is determined via a token-weighted vote, the quorum check in the _evaluateProposalOutcome_ function can be trivially bypassed by splitting one’s tokens over multiple accounts and voting with each of the accounts. Each of these sybil votes increases the _proposals[_proposalId].numVotes_ variable. This means anyone can make the quorum check pass.
    
    1.  Recommendation: Consider measuring quorum size by the percentage of existing tokens that have voted, rather than the number of unique accounts that have voted.
        
    2.  High Risk severity finding from [OpenZeppelin’s Audit of Audius](https://blog.openzeppelin.com/audius-contracts-audit/#high)
        
91.  **Inconsistently checking initialization**: When a contract is initialized, its _isInitialized_ state variable is set to true. Since interacting with uninitialized contracts would cause problems, the __requireIsInitialized_ function is available to make this check. However, this check is not used consistently. For example, it is used in the _getVotingQuorum_ function of the Governance contract, but it is not used in the _getRegistryAddress_ function of the same contract. There is no obvious difference between the functions to explain this difference, and it could be misleading and cause uninitialized contracts to be called.
    
    1.  Recommendation: Consider calling __requireIsInitialized_ consistently in all the functions of the _InitializableV2_ contracts. If there is a reason to not call it in some functions, consider documenting it. Alternatively, consider removing this check altogether and preparing a good deployment script that will ensure that all contracts are initialized in the same transaction that they are deployed. In this alternative, it would be required to check that contracts resulting from new proposals are also initialized before they are put in production.
        
    2.  Medium Risk severity finding from [OpenZeppelin’s Audit of Audius](https://blog.openzeppelin.com/audius-contracts-audit/#medium)
        
92.  **Voting period and quorum can be set to zero**: When the Governance contract is initialized, the values of _votingPeriod_ and _votingQuorum_ are checked to make sure that they are greater than 0. However, the corresponding setter functions _setVotingPeriod_ and setVotingQuorum allow these variables to be reset to 0. Setting the _votingPeriod_ to zero would cause spurious proposals that cannot be voted. Setting the quorum to zero is worse because it would allow proposals with 0 votes to be executed.
    
    1.  Recommendation: Consider adding the validation to the setter functions
        
    2.  Medium Risk severity finding from [OpenZeppelin’s Audit of Audius](https://blog.openzeppelin.com/audius-contracts-audit/#medium)
        
93.  **Some state variables are not set during initialize**: The Audius contracts can be upgraded using the unstructured storage proxy pattern. This pattern requires the use of an initializer instead of the constructor to set the initial values of the state variables. In some of the contracts, the initializer is not initializing all of the state variables.
    
    1.  Recommendation: Consider setting all the required variables in the initializer. If there is a reason for leaving them uninitialized, consider documenting it, and adding checks on the functions that use those variables to ensure that they are not called before initialization.
        
    2.  Medium Risk severity finding from [OpenZeppelin’s Audit of Audius](https://blog.openzeppelin.com/audius-contracts-audit/#medium)
        
94.  **Expired and/or paused options can still be traded**: Option tokens can still be freely transferred when the Option contract is either paused or expired (or both). This would allow malicious option holders to sell paused / expired options that cannot be exercised in the open market to exchanges and users who do not take the necessary precautions before buying an option minted by the Primitive protocol.
    
    1.  Recommendation: Should this be the system’s expected behavior, consider clearly documenting it in user-friendly documentation so as to raise awareness in option sellers and buyers. Alternatively, if the described behavior is not intended, consider implementing the necessary logic in the Option contract to prevent transfers of tokens during pause and after expiration.
        
    2.  Medium Risk severity finding from [OpenZeppelin’s Audit of Primitive](https://blog.openzeppelin.com/primitive-audit/)
        
95.  **ERC20 transfers can misbehave**: The __transferFromERC20_ function is used throughout _ACOToken.sol_ to handle transferring funds into the contract from a user. It is called within mint, within _mintTo_, and within __validateAndBurn_. In each case, the destination is the ACOToken contract. Such transfers may behave unexpectedly if the token contract charges fees. As an example, the popular USDT token does not presently charge any fees upon transfer, but it has the potential to do so. In this case the amount received would be less than the amount sent. Such tokens have the potential to lead to protocol insolvency when they are used to mint new ACOTokens. In the case of __transferERC20_, similar issues can occur, and could cause users to receive less than expected when collateral is transferred or when exercise assets are transferred.
    
    1.  Recommendation: Consider thoroughly vetting each token used within an ACO options pair, ensuring that failing _transferFrom_ and _transfer_ calls will cause reverts within _ACOToken.sol_. Additionally, consider implementing some sort of sanity check which enforces that the balance of the ACOToken contract increases by the desired amount when calling __transferFromERC20_. 
        
    2.  Medium Risk severity finding from [OpenZeppelin’s Audit of ACO Protocol](https://blog.openzeppelin.com/aco-protocol-audit/)
        
96.  **Incorrect event emission**: The _UniswapWindowUpdate_ event of the _UniswapAnchoredView_ contract is currently being emitted in the _pokeWindowValues_ function using incorrect values. In particular, as it is being emitted before relevant state changes are applied to the _oldObservation_ and _newObservation_ variables, the data logged by the event will be outdated.
    
    1.  Recommendation: Consider emitting the _UniswapWindowUpdate_ event after changes are applied so that all logged data is up-to-date.
        
    2.  Medium Risk severity finding from [OpenZeppelin’s Audit of Compound Open Price Feed – Uniswap Integration](https://blog.openzeppelin.com/compound-open-price-feed-uniswap-integration-audit/)
        
97.  **Anyone can liquidate on behalf of another account**: The Perpetual contract has a public liquidateFrom function that bypasses the checks in the liquidate function. This means that it can be called to liquidate a position when the contract is in the _SETTLED_ state. Additionally, any user can set an arbitrary from address, causing a third-party user to confiscate the under-collateralized trader’s position. This means that any trader can unilaterally rearrange another account’s position. They could also liquidate on behalf of the Perpetual Proxy, which could break some of the Automated Market Maker invariants, such as the condition that it only holds LONG positions.
    
    1.  Recommendation: Consider restricting _liquidateFrom_ to internal visibility
        
    2.  Critical Risk severity finding from [OpenZeppelin’s Audit of MCDEX Mai Protocol](https://blog.openzeppelin.com/mcdex-mai-protocol-audit/)
        
98.  **Orders cannot be cancelled**: When a user or broker calls _cancelOrder_, the cancelled mapping is updated, but this has no subsequent effects. In particular, _validateOrderParam_ does not check if the order has been cancelled.
    
    1.  Recommendation: Consider adding this check to the order validation to ensure cancelled orders cannot be filled.
        
    2.  Critical Risk severity finding from [OpenZeppelin’s Audit of MCDEX Mai Protocol](https://blog.openzeppelin.com/mcdex-mai-protocol-audit/)
        
99.  **Re-entrancy possibilities**: There are several examples of interactions preceding effects: 1) In the deposit function of the Collateral contract, collateral is retrieved before the user balance is updated and an event is emitted. 2) In the __withdraw_ function of the Collateral contract, collateral is sent before the event is emitted 3) The same pattern occurs in the _depositToInsuranceFund_, _depositEtherToInsuranceFund_ and _withdrawFromInsuranceFund_ functions of the Perpetual contract. It should be noted that even when a correctly implemented ERC20 contract is used for collateral, incoming and outgoing transfers could execute arbitrary code if the contract is also ERC777 compliant. These re-entrancy opportunities are unlikely to corrupt the internal state of the system, but they would affect the order and contents of emitted events, which could confuse external clients about the state of the system. 
    
    1.  Recommendation: Consider always following the “Check-Effects-Interactions” pattern or use _ReentrancyGuard_ contract is now used to protect those functions
        
    2.  Medium Risk severity finding from [OpenZeppelin’s Audit of MCDEX Mai Protocol](https://blog.openzeppelin.com/mcdex-mai-protocol-audit/)
        
100.  **Governance parameter changes should not be instant**: Many sensitive changes can be made by any account with the _WhitelistAdmin_ role via the functions _setGovernanceParameter_ within the _AMMGovernance_ and _PerpetualGovernance_ contracts. For example, the _WhitelistAdmin_ can change the fee schedule, the initial and maintenance margin rates, or the lot size parameters, and these new parameters instantly take effect in the protocol with important effects. For example, raising the maintenance margin rate could cause _isSafe_ to return False when it would have previously returned True. This would allow the user’s position to be liquidated. By changing _tradingLotSize_, trades may revert when being matched, where they would not have before the change. These are only examples; the complexity of the protocol, combined with unpredictable market conditions and user actions means that many other negative effects likely exist as well.
    
    1.  Recommendation: Since these changes are occasionally needed, but can create risk for the users of the protocol, consider implementing a time-lock mechanism for such changes to take place. By having a delay between the signal of intent and the actual change, users will have time to remove their funds or close trades that would otherwise be at risk if the change happened instantly. 
        
    2.  Medium Risk severity finding from [OpenZeppelin’s Audit of MCDEX Mai Protocol](https://blog.openzeppelin.com/mcdex-mai-protocol-audit/)
        
101.  **Votes can be duplicated**: The Data Verification Mechanism uses a commit-reveal scheme to hide votes during the voting period. The intention is to prevent voters from simply voting with the majority. However, the current design allows voters to blindly copy each other’s submissions, which undermines this goal. In particular, each commitment is a masked hash of the claimed price, but is not cryptographically tied to the voter. This means that anyone can copy the commitment of a target voter (for instance, someone with a large balance) and submit it as their own. When the target voter reveals their salt and price, the copycat can “reveal” the same values. Moreover, if another voter recognizes this has occurred during the commitment phase, they can also change their commitment to the same value, which may become an alternate Schelling point.
    
    1.  Recommendation: Consider including the voter address within the commitment to prevent votes from being duplicated. Additionally, as a matter of good practice, consider including the relevant timestamp, price identifier and round ID as well to limit the applicability (and reusability) of a commitment.
        
    2.  High Risk severity finding from [OpenZeppelin’s Audit of UMA Phase 1](https://blog.openzeppelin.com/uma-audit-phase-1/)

102.  **Document potential edge cases for hook receiver contracts**: The functions _withdrawTokenAndCall_() and _withdrawTokenAndCallOnBehalf_() make a call to a hook contract designated by the owner of the withdrawing stealth address. There are very few constraints on the parameters to these calls in the Umbra contract itself. Anyone can force a call to a hook contract by transferring a small amount of tokens to an address that they control and withdrawing these tokens, passing the target address as the hook receiver. 
    
    1.  Recommendation: Developers of these _UmbraHookReceiver_ contracts should be sure to validate both the caller of the _tokensWithdrawn_() function and the function parameters.
        
    2.  [ConsenSys's Audit of Umbra](https://consensys.net/diligence/audits/2021/03/umbra-smart-contracts/#document-potential-edge-cases-for-hook-receiver-contracts)
        
103.  **Document token behavior restrictions**: As with any protocol that interacts with arbitrary ERC20 tokens, it is important to clearly document which tokens are supported. Often this is best done by providing a specification for the behavior of the expected ERC20 tokens and only relaxing this specification after careful review of a particular class of tokens and their interactions with the protocol. 
    
    1.  Recommendation: Known deviations from “normal” ERC20 behavior should be explicitly noted as NOT supported by the Umbra Protocol: 1) Deflationary or fee-on-transfer tokens: These are tokens in which the balance of the recipient of a transfer may not be increased by the amount of the transfer. There may also be some alternative mechanism by which balances are unexpectedly decreased. While these tokens can be successfully sent via the _sendToken()_ function, the internal accounting of the Umbra contract will be out of sync with the balance as recorded in the token contract, resulting in loss of funds. 2) Inflationary tokens: The opposite of deflationary tokens. The Umbra contract provides no mechanism for claiming positive balance adjustments. 3) Rebasing tokens: A combination of the above cases, these are tokens in which an account’s balance increases or decreases along with expansions or contractions in supply. The contract provides no mechanism to update its internal accounting in response to these unexpected balance adjustments, and funds may be lost as a result.
        
    2.  [ConsenSys's Audit of Umbra](https://consensys.net/diligence/audits/2021/03/umbra-smart-contracts/#document-token-behavior-restrictions)
        
104.  **Full test suite is recommended**: The test suite at this stage is not complete and many of the tests fail to execute. For complicated systems such as DeFi Saver, which uses many different modules and interacts with different DeFi protocols, it is crucial to have a full test coverage that includes the edge cases and failed scenarios. Especially this helps with safer future development and upgrading each module. As we’ve seen in some smart contract incidents, a complete test suite can prevent issues that might be hard to find with manual reviews.
    
    1.  Recommendation: Add a full coverage test suite.
        
    2.  [ConsenSys's Audit of DeFi Saver](https://consensys.net/diligence/audits/2021/03/defi-saver/#full-test-suite-is-recommended)
        
105.  **Kyber getRates code is unclear**: Function names don’t reflect their true functionalities, and the code uses some undocumented assumptions.
    
    1.  Recommendation: Refactor the code to separate getting rate functionality with getSellRate and getBuyRate. Explicitly document any assumptions in the code ( slippage, etc).
        
    2.  [ConsenSys's Audit of DeFi Saver](https://consensys.net/diligence/audits/2021/03/defi-saver/#kyber-getrates-code-is-unclear)
        
106.  **Return value is not used for** _**TokenUtils.withdrawTokens**_: The return value of _TokenUtils.withdrawTokens_ which represents the actual amount of tokens that were transferred is never used throughout the repository. This might cause discrepancy in the case where the original value of __amount_ was _type(uint256).max_.
    
    1.  Recommendation: The return value can be used to validate the withdrawal or used in the event emitted
        
    2.  [ConsenSys's Audit of DeFi Saver](https://consensys.net/diligence/audits/2021/03/defi-saver/#return-value-is-not-used-for-tokenutils-withdrawtokens)
        
107.  **Missing access control for** _**DefiSaverLogger.Log**_: _DefiSaverLogger_ is used as a logging aggregator within the entire dapp, but anyone can create logs.
    
    1.  Recommendation: Add access control to all functions appropriately
        
    2.  [Consensys Audit of DeFi Saver](https://consensys.net/diligence/audits/2021/03/defi-saver/#missing-access-control-for-defisaverlogger-log)
        
108.  **Remove stale comments**: Remove inline comments that suggest the two uint256 values _DAOfiV1Pair.reserveBase_ and _DAOfiV1Pair.reserveQuote_ are stored in the same storage slot. This is likely a carryover from the _UniswapV2Pair_ contract, in which _reserve0_, _reserve1_, and _blockTimestampLast_ are packed into a single storage slot.
    
    1.  Recommendation: Remove stale comments
        
    2.  [ConsenSys's Audit of DAOfi](https://consensys.net/diligence/audits/2021/02/daofi/#remove-stale-comments)
        
109.  **Discrepancy between code and comments**: There is a mismatch between what the code implements and what the corresponding comment describes that code implements.
    
    1.  Recommendation: Update the code or the comment to be consistent
        
    2.  [ConsenSys's Audit of mstable-1.1](https://consensys.net/diligence/audits/2020/07/mstable-1.1/#discrepancy-between-code-and-comments)
        
110.  **Remove unnecessary call to** _**DAOfiV1Factory.formula**_**()**: The _DAOfiV1Pair_ functions _initialize_(), _getBaseOut_(), and _getQuoteOut_() all use the private function __getFormula_(), which makes a call to the factory to retrieve the address of the _BancorFormula_ contract. The formula address in the factory is set in the constructor and cannot be changed, so these calls can be replaced with an immutable value in the pair contract that is set in its constructor.
    
    1.  Recommendation: Remove unnecessary calls
        
    2.  [ConsenSys's Audit of DAOfi](https://consensys.net/diligence/audits/2021/02/daofi/#remove-unnecessary-call-to-daofiv1factory-formula) 
        
111.  **Deeper validation of curve math**: Increased testing of edge cases in complex mathematical operations could have identified at least one issue raised in this report. Additional unit tests are recommended, as well as fuzzing or property-based testing of curve-related operations. Improperly validated interactions with the _BancorFormula_ contract are seen to fail in unanticipated and potentially dangerous ways, so care should be taken to validate inputs and prevent pathological curve parameters.
    
    1.  Recommendation: More validation of mathematical operations
        
    2.  [ConsenSys's Audit of DAOfi](https://consensys.net/diligence/audits/2021/02/daofi/#deeper-validation-of-curve-math) 
        
112.  _**GovernorAlpha**_ **proposals may be canceled by the proposer, even after they have been accepted and queued**: _GovernorAlpha_ allows proposals to be canceled via cancel. A proposer may cancel proposals in any of these states: Pending, Active, Canceled, Defeated, Succeeded, Queued, Expired. 
    
    1.  Recommendation: Prevent proposals from being canceled unless they are in the Pending or Active states.
        
    2.  [ConsenSys's Audit of Fei Protocol](https://consensys.net/diligence/audits/2021/01/fei-protocol/#governoralpha-proposals-may-be-canceled-by-the-proposer-even-after-they-have-been-accepted-and-queued) 
        
113.  **Require a delay period before granting** _**KYC_ADMIN_ROLE**_  **Acknowledged**: The KYC Admin has the ability to freeze the funds of any user at any time by revoking the _KYC_MEMBER_ROLE_. The trust requirements from users can be decreased slightly by implementing a delay on granting this ability to new addresses. While the management of private keys and admin access is outside the scope of this review, the addition of a time delay can also help protect the development team and the system itself in the event of private key compromise.
    
    1.  Recommendation: Use a _TimelockController_ as the _KYC_DEFAULT_ADMIN_ of the eRLC contract
        
    2.  [ConsenSys's Audit of eRLC](https://consensys.net/diligence/audits/2021/01/erlc-iexec/#erlc-require-a-delay-period-before-granting-kyc-admin-role)
        
114.  **Improve inline documentation and test coverage**: The source-units hardly contain any inline documentation which makes it hard to reason about methods and how they are supposed to be used. Additionally, test-coverage seems to be limited. Especially for a public-facing exchange contract system test-coverage should be extensive, covering all methods and functions that can directly be accessed including potential security-relevant and edge-cases. This would have helped in detecting some of the findings raised with this report.
    
    1.  Recommendation: Consider adding natspec-format compliant inline code documentation, describe functions, what they are used for, and who is supposed to interact with them. Document function or source-unit specific assumptions. Increase test coverage.
        
    2.  [ConsenSys's Audit of 1inch Liquidity Protocol](https://consensys.net/diligence/audits/2020/12/1inch-liquidity-protocol/#improve-inline-documentation-and-test-coverage)
        
115.  **Unspecific compiler version pragma**: For most source-units the compiler version pragma is very unspecific _^0.6.0_. While this often makes sense for libraries to allow them to be included with multiple different versions of an application, it may be a security risk for the actual application implementation itself. A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up actually checking a different evm compilation that is ultimately deployed on the blockchain.
    
    1.  Recommendation: Avoid floating pragmas. We highly recommend pinning a concrete compiler version (latest without security issues) in at least the top-level “deployed” contracts to make it unambiguous which compiler version is being used. Rule of thumb: a flattened source-unit should have at least one non-floating concrete solidity compiler version pragma.
        
    2.  [ConsenSys's Audit of 1inch Liquidity Protocol](https://consensys.net/diligence/audits/2020/12/1inch-liquidity-protocol/#unspecific-compiler-version-pragma)
        
116.  **Use of hardcoded gas limits can be problematic**: Hardcoded gas limits can be problematic as the past has shown that gas economics in ethereum have changed, and may change again potentially rendering the contract system unusable in the future.
    
    1.  Recommendation: Be conscious about this potential limitation and prepare for the case where gas prices might change in a way that negatively affects the contract system.
        
    2.  [ConsenSys's Audit of 1inch Liquidity Protocol](https://consensys.net/diligence/audits/2020/12/1inch-liquidity-protocol/#use-of-hardcoded-gas-limits-can-be-problematic)
        
117.  **Anyone can steal all the funds that belong to** _**ReferralFeeReceiver**_: The _ReferralFeeReceiver_ receives pool shares when users _swap()_ tokens in the pool. A _ReferralFeeReceiver_ may be used with multiple pools and, therefore, be a lucrative target as it is holding pool shares. Any token or ETH that belongs to the _ReferralFeeReceiver_ is at risk and can be drained by any user by providing a custom mooniswap pool contract that references existing token holdings. It should be noted that none of the functions in _ReferralFeeReceiver_ verify that the user-provided mooniswap pool address was actually deployed by the linked MooniswapFactory.
    
    1.  Recommendation: Enforce that the user-provided mooniswap contract was actually deployed by the linked factory. Other contracts cannot be trusted. Consider implementing token sorting and de-duplication (_tokenA!=tokenB_) in the pool contract constructor as well. Consider employing a reentrancy guard to safeguard the contract from reentrancy attacks. Improve testing. The methods mentioned here are not covered at all. Improve documentation and provide a specification that outlines how this contract is supposed to be used.
        
    2.  Critical finding in [ConsenSys's Audit of 1inch Liquidity Protocol](https://consensys.net/diligence/audits/2020/12/1inch-liquidity-protocol/#out-of-scope-referralfeereceiver-anyone-can-steal-all-the-funds-that-belong-to-referralfeereceiver)
        
118.  **Unpredictable behavior for users due to admin front running or general bad timing**: In a number of cases, administrators of contracts can update or upgrade things in the system without warning. This has the potential to violate a security goal of the system. Specifically, privileged roles could use front running to make malicious changes just ahead of incoming transactions, or purely accidental negative effects could occur due to the unfortunate timing of changes. In general users of the system should have assurances about the behavior of the action they’re about to take.
    
    1.  Recommendation: We recommend giving the user advance notice of changes with a time lock. For example, make all system-parameter and upgrades require two steps with a mandatory time window between them. The first step merely broadcasts to users that a particular change is coming, and the second step commits that change after a suitable waiting period. This allows users that do not accept the change to withdraw immediately.
        
    2.  [ConsenSys's Audit of 1inch Liquidity Protocol](https://consensys.net/diligence/audits/2020/12/1inch-liquidity-protocol/#unpredictable-behavior-for-users-due-to-admin-front-running-or-general-bad-timing)
        
119.  **Improve system documentation and create a complete technical specification**: A system’s design specification and supporting documentation should be almost as important as the system’s implementation itself. Users rely on high-level documentation to understand the big picture of how a system works. Without spending time and effort to create palatable documentation, a user’s only resource is the code itself, something the vast majority of users cannot understand. Security assessments depend on a complete technical specification to understand the specifics of how a system works. When a behavior is not specified (or is specified incorrectly), security assessments must base their knowledge in assumptions, leading to less effective review. Maintaining and updating code relies on supporting documentation to know why the system is implemented in a specific way. If code maintainers cannot reference documentation, they must rely on memory or assistance to make high-quality changes. Currently, the only documentation for Growth DeFi is a single README file, as well as code comments.
    
    1.  Recommendation: Improve system documentation and create a complete technical specification
        
    2.  [ConsenSys's Audit of Growth DeFi](https://consensys.net/diligence/audits/2020/12/growth-defi-v1/#improve-system-documentation-and-create-a-complete-technical-specification)
        
120.  **Ensure system states, roles, and permissions are sufficiently restrictive**: Smart contract code should strive to be strict. Strict code behaves predictably, is easier to maintain, and increases a system’s ability to handle nonideal conditions. Our assessment of Growth DeFi found that many of its states, roles, and permissions are loosely defined.
    
    1.  Recommendation: Document the use of administrator permissions. Monitor the usage of administrator permissions. Specify strict operation requirements for each contract.
        
    2.  [ConsenSys's Audit of Growth DeFi](https://consensys.net/diligence/audits/2020/12/growth-defi-v1/#ensure-system-states-roles-and-permissions-are-sufficiently-restrictive)
        
121.  **Evaluate all tokens prior to inclusion in the system**: Review current and future tokens in the system for non-standard behavior. Particularly dangerous functionality to look for includes a callback (ie. ERC777) which would enable an attacker to execute potentially arbitrary code during the transaction, fees on transfers, or inflationary/deflationary tokens.
    
    1.  Recommendation: Evaluate all tokens prior to inclusion in the system
        
    2.  [ConsenSys's Audit of Growth DeFi](https://consensys.net/diligence/audits/2020/12/growth-defi-v1/#evaluate-all-tokens-prior-to-inclusion-in-the-system)
        
122.  **Use descriptive names for contracts and libraries**: The code base makes use of many different contracts, abstract contracts, interfaces, and libraries for inheritance and code reuse. In principle, this can be a good practice to avoid repeated use of similar code. However, with no descriptive naming conventions to signal which files would contain meaningful logic, codebase becomes difficult to navigate.
    
    1.  Recommendation: Use descriptive names for contracts and libraries
        
    2.  [ConsenSys's Audit of Growth DeFi](https://consensys.net/diligence/audits/2020/12/growth-defi-v1/#use-descriptive-names-for-contracts-and-libraries)
        
123.  **Prevent contracts from being used before they are entirely initialized**: Many contracts allow users to deposit / withdraw assets before the contracts are entirely initialized, or while they are in a semi-configured state. Because these contracts allow interaction on semi-configured states, the number of configurations possible when interacting with the system makes it incredibly difficult to determine whether the contracts behave as expected in every scenario, or even what behavior is expected in the first place.
    
    1.  Recommendation: Prevent contracts from being used before they are entirely initialized
        
    2.  [ConsenSys's Audit of Growth DeFi](https://consensys.net/diligence/audits/2020/12/growth-defi-v1/#prevent-contracts-from-being-used-before-they-are-entirely-initialized)
        
124.  **Potential resource exhaustion by external calls performed within an unbounded loop**: _DydxFlashLoanAbstraction_.__requestFlashLoan_ performs external calls in a potentially-unbounded loop. Depending on changes made to DyDx’s _SoloMargin_, this may render this flash loan provider prohibitively expensive. In the worst case, changes to _SoloMargin_ could make it impossible to execute this code due to the block gas limit.
    
    1.  Recommendation: Reconsider or bound the loop
        
    2.  [ConsenSys's Audit of Growth DeFi](https://consensys.net/diligence/audits/2020/12/growth-defi-v1/#potential-resource-exhaustion-by-external-calls-performed-within-an-unbounded-loop)
        
125.  **Owners can never be removed**: The intention of _setOwners_() is to replace the current set of owners with a new set of owners. However, the _isOwner_ mapping is never updated, which means any address that was ever considered an owner is permanently considered an owner for purposes of signing transactions.
    
    1.  Recommendation: In _setOwners__(), before adding new owners, loop through the current set of owners and clear their _isOwner_ booleans
        
    2.  Critical finding in [ConsenSys's Audit of Paxos](https://consensys.net/diligence/audits/2020/11/paxos/#owners-can-never-be-removed)
        
126.  **Potential manipulation of stable interest rates using flash loans**: Flash loans allow users to borrow large amounts of liquidity from the protocol. It is possible to adjust the stable rate up or down by momentarily removing or adding large amounts of liquidity to reserves. 
    
    1.  Recommendation: This type of manipulation is difficult to prevent especially when flash loans are available. Aave should monitor the protocol at all times to make sure that interest rates are being rebalanced to sane values.
        
    2.  [ConsenSys's Audit of Aave Protocol V2](https://consensys.net/diligence/audits/2020/09/aave-protocol-v2/#potential-manipulation-of-stable-interest-rates-using-flash-loans)
        
127.  **Only whitelist validated assets**: Because some of the functionality relies on correct token behavior, any whitelisted token should be audited in the context of this system. Problems can arise if a malicious token is whitelisted because it can block people from voting with that specific token or gain unfair advantage if the balance can be manipulated.
    
    1.  Recommendation: Make sure to audit any new whitelisted asset.
        
    2.  [ConsenSys's Audit of Aave Governance DAO](https://consensys.net/diligence/audits/2020/08/aave-governance-dao/#only-whitelist-validated-assets)
        
128.  **Underflow if** _**TOKEN_DECIMALS**_ **are greater than 18**: In _latestAnswer_(), the assumption is made that _TOKEN_DECIMALS_ is less than 18.
    
    1.  Recommendation: Add a simple check to the constructor to ensure the added token has 18 decimals or less
        
    2.  [ConsenSys's Audit of Aave CPM Price Provider](https://consensys.net/diligence/audits/2020/05/aave-cpm-price-provider/#underflow-if-token-decimals-are-greater-than-18)
        
129.  **Chainlink’s performance at times of price volatility**: In order to understand the risk of the Chainlink oracle deviating significantly, it would be helpful to compare historical prices on Chainlink when prices are moving rapidly, and see what the largest historical delta is compared to the live price on a large exchange.
    
    1.  Recommendation: Review Chainlink’s performance at times of price volatility
        
    2.  [ConsenSys's Audit of Aave CPM Price Provider](https://consensys.net/diligence/audits/2020/05/aave-cpm-price-provider/#review-chainlink-s-performance-at-times-of-price-volatility) 
        
130.  **Consider an iterative approach to launching. Be aware of and prepare for worst-case scenarios**: The system has many components with complex functionality and no apparent upgrade path. 
    
    1.  Recommendation: We recommend identifying which components are crucial for a minimum viable system, then focusing efforts on ensuring the security of those components first, and then moving on to the others. During the early life of the system, have a method for pausing and upgrading the system. 
        
    2.  [ConsenSys's Audit of Lien Protocol](https://consensys.net/diligence/audits/2020/05/lien-protocol/#consider-an-iterative-approach-to-launching)
        
131.  **Use of modifiers for repeated checks**: It is recommended to use modifiers for common checks within different functions. This will result in less code duplication in the given smart contract and adds significant readability into the code base.
    
    1.  Recommendation: Use of modifiers for repeated checks
        
    2.  [ConsenSys's Audit of Balancer Finance](https://consensys.net/diligence/audits/2020/05/balancer-finance/#use-of-modifiers-for-repeated-checks)
        
132.  **Switch modifier order**: _BPool_ functions often use modifiers in the following order: __logs__, __lock__. Because __lock__ is a reentrancy guard, it should take precedence over __logs_._
    
    1.  Recommendation: Place __lock__ before other modifiers; ensuring it is the very first and very last thing to run when a function is called.
        
    2.  [ConsenSys's Audit of Balancer Finance](https://consensys.net/diligence/audits/2020/05/balancer-finance/#switch-modifier-order-in-bpool)
        
133.  **Address codebase fragility**: Software is considered “fragile” when issues or changes in one part of the system can have side-effects in conceptually unrelated parts of the codebase. Fragile software tends to break easily and may be challenging to maintain.
    
    1.  Recommendation: Building an anti-fragile system requires careful thought and consideration outside of the scope of this review. In general, prioritize the following concepts: 1) Follow the single-responsibility principle of functions 2) Reduce reliance on external systems
        
    2.  [ConsenSys's Audit of MCDEX Mai Protocol V2](https://consensys.net/diligence/audits/2020/05/mcdex-mai-protocol-v2/#address-codebase-fragility)
        
134.  **Reentrancy could lead to incorrect order of emitted events**: The order of operations in the __moveTokensAndETHfromAdjustment_ function in the _BorrowOperations_ contract may allow an attacker to cause events to be emitted out of order. In the event that the borrower is a contract, this could trigger a callback into _BorrowerOperations_, executing the _ _adjustTrove_ flow above again. As the __moveTokensAndETHfromAdjustment_ call is the final operation in the function the state of the system on-chain cannot be manipulated. However, there are events that are emitted after this call. In the event of a reentrant call, these events would be emitted in the incorrect order. The event for the second operation i s emitted first, followed by the event for the first operation. Any off-chain monitoring tools may now have an inconsistent view of on-chain state.
    
    1.  Recommendation: Apply the checks-effects-interactions pattern and move the event emissions above the call to _ _moveTokensAndETHfromAdjustment_ to avoid the potential reentrancy.
        
    2.  [ToB's Audit of Liquidity](https://github.com/trailofbits/publications/blob/master/reviews/Liquity.pdf)
        
135.  **Variable shadowing from OUSD to ERC20**: OUSD inherits from ERC20, but redefines the _ _allowances_ and _ _totalSupply_ state variables. As a result, access to these variables can lead to returning different values.
    
    1.  Recommendation: Remove the shadowed variables (_ _allowances_ and _ _totalSupply_) in OUSD.
        
    2.  [ToB's Audit of Origin Dollar](https://github.com/trailofbits/publications/blob/master/reviews/OriginDollar.pdf)
        
136.  **VaultCore.rebase functions have no return statements**: _VaultCore.rebase()_ and _VaultCore.rebase(bool)_ return a _uint_ but lack a return statement. As a result these functions will always return the default value, and are likely to cause issues for their callers. Both _VaultCore.rebase()_ and _VaultCore.rebase(bool)_ are expected to return a _uint256_. _rebase()_ does not have a return statement. _rebase(bool)_ has one return statement in one branch (return 0), but lacks a return statement for the other paths. So both functions will always return zero. As a result, a third-party code relying on the return value might not work as intended.
    
    1.  Recommendation: Add the missing return statement(s) or remove the return type in _VaultCore.rebase()_ and _VaultCore.rebase(bool)_. Properly adjust the documentation as necessary.
        
    2.  [ToB's Audit of Origin Dollar](https://github.com/trailofbits/publications/blob/master/reviews/OriginDollar.pdf)
        
137.  **Multiple contracts are missing inheritances**: Multiple contracts are the implementation of their interfaces, but do not inherit from them. This behavior is error-prone and might lead the implementation to not follow the interface if the code is updated.
    
    1.  Recommendation: Ensure contracts inherit from their interfaces
        
    2.  [ToB's Audit of Origin Dollar](https://github.com/trailofbits/publications/blob/master/reviews/OriginDollar.pdf)
        
138.  **Solidity compiler optimizations can be dangerous**: Yield Protocol has enabled optional compiler optimizations in Solidity. There have been several bugs with security implications related to optimizations. Moreover, optimizations are actively being developed . Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them. Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past . A high-severity bug in the emscripten -generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG. Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6 .
    
    1.  Recommendation: Short term, measure the gas savings from optimizations, and carefully weigh them against the possibility of an optimization-related bug. Long term, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.
        
    2.  [ToB's Audit of Yield Protocol](https://github.com/trailofbits/publications/blob/master/reviews/YieldProtocol.pdf)
        
139.  **Permission-granting is too simplistic and not flexible enough**: The Yield Protocol contracts implement an oversimplified permission system that can be abused by the administrator. The Yield Protocol implements several contracts that need to call privileged functions from each other. However, there is no way to specify which operation can be called for every privileged user. All the authorized addresses can call any restricted function, and the owner can add any number of them. Also, the privileged addresses are supposed to be smart contracts; however, there is no check for that. Moreover, once an address is added, it cannot be deleted.
    
    1.  Recommendation: Rewrite the authorization system to allow only certain addresses to access certain functions
        
    2.  [ToB's Audit of Yield Protocol](https://github.com/trailofbits/publications/blob/master/reviews/YieldProtocol.pdf) 
        
140.  **Lack of validation when setting the maturity value:** When a _fyDAI_ contract is deployed, one of the deployment parameters is a maturity date, passed as a Unix timestamp. This is the date at which point _fyDAI_ tokens can be redeemed for the underlying Dai. Currently, the contract constructor performs no validation on this timestamp to ensure it is within an acceptable range. As a result, it is possible to mistakenly deploy a _YDai_ contract that has a maturity date in the past or many years in the future, which may not be immediately noticed.
    
    1.  Recommendation: Short term, add checks to the _YDai_ contract constructor to ensure maturity timestamps fall within an acceptable range. This will prevent maturity dates from being mistakenly set in the past or too far in the future. Long term, always perform validation of parameters passed to contract constructors. This will help detect and prevent errors during deployment.
        
    2.  [ToB's Audit of Yield Protocol](https://github.com/trailofbits/publications/blob/master/reviews/YieldProtocol.pdf) 
        
141.  **Delegates can be added or removed repeatedly to bloat logs**: Several contracts in the Yield Protocol system inherit the Delegable contract. This contract allows users to delegate the ability to perform certain operations on their behalf to other addresses. When a user adds or removes a delegate, a corresponding event is emitted to log this operation. However, there is no check to prevent a user from repeatedly adding or removing a delegation that is already enabled or revoked, which could allow redundant events to be emitted repeatedly.
    
    1.  Recommendation: Short term, add a _require_ statement to check that the delegate address is not already enabled or disabled for the user. This will ensure log messages are only emitted when a delegate is activated or deactivated. Long term, review all operations and avoid emitting events in repeated calls to idempotent operations. This will help prevent bloated logs.
        
    2.  [ToB's Audit of Yield Protocol](https://github.com/trailofbits/publications/blob/master/reviews/YieldProtocol.pdf)
        
142.  **Lack of events for critical operations**: Several critical operations do not trigger events, which will make it difficult to review the correct behavior of the contracts once deployed. Users and blockchain monitoring systems will not be able to easily detect suspicious behaviors without events.
    
    1.  Recommendation: Short term, add events where appropriate for all critical operations. Long term, consider using a blockchain monitoring system to track any suspicious behavior in the contracts. 
        
    2.  [ToB's Audit of 0x Protocol](https://github.com/trailofbits/publications/blob/master/reviews/0x-protocol.pdf)
        
143.   _**_assertStakingPoolExists**_ **never returns true**: The __assertStakingPoolExists_ should return a bool to determine if the staking pool exists or not; however, it only returns false or reverts. The __assertStakingPoolExists_ function checks that a staking pool exists given a pool id parameter. However, this function does not use a return statement and therefore, it will always return false or revert.
    
    1.  Recommendation: Add a return statement or remove the return type. Properly adjust the documentation, if needed.
        
    2.  [ToB's Audit of 0x Protocol](https://github.com/trailofbits/publications/blob/master/reviews/0x-protocol.pdf)
        
144.  **_min* and _max* have unorthodox semantics**: Throughout the Curve contract, __minTargetAmount_ and __maxOriginAmount_ are used as open ranges (i.e., ranges that exclude the value itself). This contravenes the standard meanings of the terms “minimum” and “maximum,” which are generally used to describe closed ranges.
    
    1.  Recommendation: Short term, unless they are intended to be strict, make the inequalities in the require statements non-strict. Alternatively, consider refactoring the variables or providing additional documentation to convey that they are meant to be exclusive bounds. Long term, ensure that mathematical terms such as “minimum,” “at least,” and “at most” are used in the typical way—that is, to describe values inclusive of minimums or maximums (as relevant).
        
    2.  [ToB's Audit of DFX Finance](https://github.com/dfx-finance/protocol/blob/main/audits/2021-05-03-Trail_of_Bits.pdf)
        
145.  _**CurveFactory.newCurve**_ **returns existing curves without provided arguments**: _CurveFactory.newCurve_ takes values and creates a Curve contract instance for each __baseCurrency_ and __quoteCurrency_ pair, populating the Curve with provided weights and assimilator contract references. However, if the pair already exists, the existing Curve will be returned without any indication that it is not a newly created Curve with the provided weights. If an operator attempts to create a new Curve for a base-and-quote-currency pair that already exists, _CurveFactory_ will return the existing Curve instance regardless of whether other creation parameters differ. A naive operator may overlook this issue.
    
    1.  Recommendation: Consider rewriting _newCurve_ such that it reverts in the event that a base-and-quote-currency pair already exists. A view function can be used to check for and retrieve existing Curves without any gas payment prior to an attempt at Curve creation.
        
    2.  [ToB's Audit of DFX Finance](https://github.com/dfx-finance/protocol/blob/main/audits/2021-05-03-Trail_of_Bits.pdf)
        
146.  **Missing zero-address checks in** _**Curve.transferOwnership**_ **and** _**Router.constructor**_: Like other similar functions, _Curve._transfer_ and _Orchestrator.includeAsset_ perform zero-address checks. However, _Curve.transferOwnership_ and the Router constructor do not. This may make sense for _Curve.transferOwnership_, because without zero-address checks, the function may serve as a means of burning ownership. However, popular contracts that define similar functions often consider this case, such as OpenZeppelin’s _Ownable_ contracts. Conversely, a zero-address check should be added to the Router constructor to prevent the deployment of an invalid Router, which would revert upon a call to the zero address.
    
    1.  Recommendation: Short term, consider adding zero-address checks to the Router’s constructor and Curve’s _transferOwnership_ function to prevent operator errors. Long term, review state variables which referencing contracts to ensure that the code that sets the state variables performs zero-address checks where necessary
        
    2.  [ToB's Audit of DFX Finance](https://github.com/dfx-finance/protocol/blob/main/audits/2021-05-03-Trail_of_Bits.pdf)
        
147.  _**safeApprove**_ **does not check return values for approve call**: Although the Router contract uses OpenZeppelin’s _SafeERC20_ library to perform safe calls to ERC20’s approve function, the Orchestrator library defines its own _safeApprove_ function. This function checks that a call to approve was successful but does not check returndata to verify whether the call returned true. In contrast, OpenZeppelin’s _safeApprove_ function checks return values appropriately. This issue may result in uncaught approve errors in successful Curve deployments, causing undefined behavior.
    
    1.  Recommendation: Short term, leverage OpenZeppelin’s _safeApprove_ function wherever possible. Long term, ensure that all low-level calls have accompanying contract existence checks and return value checks where appropriate.
        
    2.  [ToB's Audit of DFX Finance](https://github.com/dfx-finance/protocol/blob/main/audits/2021-05-03-Trail_of_Bits.pdf)
        
148.  **ERC20 token Curve does not implement symbol, name, or decimals**: _Curve.sol_ is an ERC20 token and implements all six required ERC20 methods: _balanceOf_, _totalSupply_, _allowance_, _transfer_, _approve_, and _transferFrom_. However, it does not implement the optional but extremely common view methods _symbol_, _name_, and _decimals_.
    
    1.  Recommendation: Short term, implement _symbol_, _name_, and _decimals_ on Curve contracts. Long term, ensure that contracts conform to all required and recommended industry standards.
        
    2.  [ToB's Audit of DFX Finance](https://github.com/dfx-finance/protocol/blob/main/audits/2021-05-03-Trail_of_Bits.pdf)
        
149.  **Insufficient use of** _**SafeMath**_: _CurveMath.calculateTrade_ is used to compute the output amount for a trade. However, although _SafeMath_ is used throughout the codebase to prevent underflows/overflows, it is not used in this calculation. Although we could not prove that the lack of _SafeMath_ would cause an arithmetic issue in practice, all such calculations would benefit from the use of _SafeMath_.
    
    1.  Recommendation: Review all critical arithmetic to ensure that it accounts for underflows, overflows, and the loss of precision. Consider using _SafeMath_ and the safe functions of _ABDKMath64x64_ where possible to prevent underflows and overflows.
        
    2.  [ToB's Audit of DFX Finance](https://github.com/dfx-finance/protocol/blob/main/audits/2021-05-03-Trail_of_Bits.pdf)
        
150.  _**setFrozen**_ **can be front-run to deny deposits/swaps**: Currently, a Curve contract owner can use the _setFrozen_ function to set the contract into a state that will block swaps and deposits. A contract owner could leverage this process to front-run transactions and freeze contracts before certain deposits or swaps are made; the contract owner could then unfreeze them at a later time.
    
    1.  Recommendation: Short term, consider rewriting _setFrozen_ such that any contract freeze will not last long enough for a malicious user to easily execute an attack. Alternatively, depending on the intended use of this function, consider implementing permanent freezes.
        
    2.  [ToB's Audit of DFX Finance](https://github.com/dfx-finance/protocol/blob/main/audits/2021-05-03-Trail_of_Bits.pdf)
        
151.  **Account creation spam:** Hermez has a limit of _2**MAX_NLEVELS_ accounts. There is no fee on account creation, so an attacker can spam the network with account creation to fill the tree. If _MAX_NLEVELS_ is below 32, an attacker can quickly reach the account limit. If _MAX_NLEVELS_ is above or equal to 32, the time required to fill the tree will depend on the number of transactions accepted per second, but will take at least a couple of months. Ethereum miners do not have to pay for account creation. Therefore, an Ethereum miner can spam the network with account creation by sending L1 user transactions. 
    
    1.  Recommendation: Short term, add a fee for account creation or ensure _MAX_NLEVELS_ is at least 32. Also, monitor account creation and alert the community if a malicious coordinator spams the system. This will prevent an attacker from spamming the system to prevent new accounts from being created. Long term, when designing spam mitigation, consider that L1 gas cost can be avoided by Ethereum miners.
        
    2.  [ToB's Audit of Hermez Network](https://github.com/trailofbits/publications/blob/master/reviews/hermez.pdf)
        
152.  **Using empty functions instead of interfaces leaves contract error-prone**: _WithdrawalDelayerInterface_ is a contract meant to be an interface. It contains functions with empty bodies instead of function signatures, which might lead to unexpected behavior. A contract inheriting from _WithdrawalDelayerInterface_ will not require an override of these functions and will not benefit from the compiler checks on its correct interface.
    
    1.  Recommendation: Short term, use an interface instead of a contract in _WithdrawalDelayerInterface_. This will make derived contracts follow the interface properly. Long term, properly document the inheritance schema of the contracts. Use Slither’s inheritance-graph printer to review the inheritance.
        
    2.  [ToB's Audit of Hermez Network](https://github.com/trailofbits/publications/blob/master/reviews/hermez.pdf)
        
153.  _**cancelTransaction**_ **can be called on non-queued transaction**: Without a transaction existence check in _cancelTransaction_, an attacker can confuse monitoring systems. _cancelTransaction_ emits an event without checking that the transaction to be canceled exists. This allows a malicious admin to confuse monitoring systems by generating malicious events.
    
    1.  Recommendation: Short term, check that the transaction to be canceled exists in _cancelTransaction_. This will ensure that monitoring tools can rely on emitted events. Long term, write a specification of each function and thoroughly test it with unit tests and fuzzing. Use symbolic execution for arithmetic invariants.
        
    2.  [ToB's Audit of Hermez Network](https://github.com/trailofbits/publications/blob/master/reviews/hermez.pdf)
        
154.  **Contracts used as dependencies do not track upstream changes**: Third-party contracts like __concatStorage_ are pasted into the Hermez repository. Moreover, the code documentation does not specify the exact revision used, or if it is modified. This makes updates and security fixes on these dependencies unreliable since they must be updated manually. __concatStorage_ is borrowed from the solidity-bytes-utils library, which provides helper functions for byte-related operations. Recently, a critical vulnerability was discovered in the library’s slice function which allows arbitrary writes for user-supplied inputs. 
    
    1.  Recommendation: Short term, review the codebase and document each dependency’s source and version. Include the third-party sources as submodules in your Git repository so internal path consistency can be maintained and dependencies are updated periodically. Long term, identify the areas in the code that are relying on external libraries and use an Ethereum development environment and NPM to manage packages as part of your project.
        
    2.  [ToB's Audit of Hermez Network](https://github.com/trailofbits/publications/blob/master/reviews/hermez.pdf)
        
155.  **Expected behavior regarding authorization for adding tokens is unclear**: _addToken_ allows anyone to list a new token on Hermez. This contradicts the online documentation, which implies that only the governance should have this authorization. It is unclear whether the implementation or the documentation is correct.
    
    1.  Recommendation: Short term, update either the implementation or the documentation to standardize the authorization specification for adding tokens. Long term, write a specification of each function and thoroughly test it with unit tests and fuzzing. Use symbolic execution for arithmetic invariants.
        
    2.  [ToB's Audit of Hermez Network](https://github.com/trailofbits/publications/blob/master/reviews/hermez.pdf)
        
156.  **Contract name duplication leaves codebase error-prone**: The codebase has multiple contracts that share the same name. This allows buidler-waffle to generate incorrect json artifacts, preventing third parties from using their tools. Buidler-waffle does not correctly support a codebase with duplicate contract names. The compilation overwrites compilation artifacts and prevents the use of third-party tools, such as Slither.
    
    1.  Recommendation: Short term, prevent the re-use of duplicate contract names or change the compilation framework. Long term, use Slither, which will help detect duplicate contract names.
        
    2.  [ToB's Audit of Hermez Network](https://github.com/trailofbits/publications/blob/master/reviews/hermez.pdf)
        
157.  **Use of hard-coded addresses may cause errors**: Each contract needs contract addresses in order to be integrated into other protocols and systems. These addresses are currently hard-coded, which may cause errors and result in the codebase’s deployment with an incorrect asset. Using hard-coded values instead of deployer-provided values makes these contracts incredibly difficult to test.
    
    1.  Recommendation: Short term, set addresses when contracts are created rather than using hard-coded values. This practice will facilitate testing. Long term, to ensure that contracts can be tested and reused across networks, avoid using hard-coded parameters.
        
    2.  [ToB's Audit of Advanced Blockchains](https://github.com/trailofbits/publications/blob/master/reviews/AdvancedBlockchain.pdf)
        
158.  **Borrow rate depends on approximation of blocks per year**: The borrow rate formula uses an approximation of the number of blocks mined annually. This number can change across different blockchains and years. The current value assumes that a new block is mined every 15 seconds, but on Ethereum mainnet, a new block is mined every ~13 seconds. To calculate the base rate, the formula determines the approximate borrow rate over the past year and divides that number by the estimated number of blocks mined per year. However, _blocksPerYear_ is an estimated value and may change depending on transaction throughput. Additionally, different blockchains may have different block-settling times, which could also alter this number.
    
    1.  Recommendation: Short term, analyze the effects of a deviation from the actual number of blocks mined annually in borrow rate calculations and document the associated risks. Long term, identify all variables that are affected by external factors, and document the risks associated with deviations from their true values.
        
    2.  [ToB's Audit of Advanced Blockchains](https://github.com/trailofbits/publications/blob/master/reviews/AdvancedBlockchain.pdf)
        
159.  **Flash loan rate lacks bounds and can be set arbitrarily**: There are no lower or upper bounds on the flash loan rate implemented in the contract. The Blacksmith team could therefore set an arbitrarily high flash loan rate to secure higher fees. The Blacksmith team sets the __flashLoanRate_ when the Vault is first initialized. The _blackSmithTeam_ address can then update this value by calling _updateFlashloanRate_. However, because there is no check on either setter function, the flash loan rate can be set arbitrarily. A very high rate could enable the Blacksmith team to steal vault deposits.
    
    1.  Recommendation: Short term, introduce lower and upper bounds for all configurable parameters in the system to limit privileged users’ abilities. Long term, identify all incoming parameters in the system as well as the financial implications of large and small corner-case values. Additionally, use Echidna or Manticore to ensure that system invariants hold.
        
    2.  [ToB's Audit of Advanced Blockchains](https://github.com/trailofbits/publications/blob/master/reviews/AdvancedBlockchain.pdf)
        
160.  **Logic duplicated across code**: The logic in the repositories provided to Trail of Bits contains a significant amount of duplicated code. This development practice increases the risk that new bugs will be introduced into the system, as bug fixes must be copied and pasted into files across the system.
    
    1.  Recommendation: Short term, use inheritance to allow code to be reused across contracts. Changes to one inherited contract will be applied to all files without requiring developers to copy and paste them. Long term, minimize the amount of manual copying and pasting required to apply changes made to one file to other files.
        
    2.  [ToB's Audit of Advanced Blockchains](https://github.com/trailofbits/publications/blob/master/reviews/AdvancedBlockchain.pdf)
        
161.  **Insufficient testing**: The repositories under review lack appropriate testing, which increases the likelihood of errors in the development process and makes the code more difficult to review.
    
    1.  Recommendation: Short term, ensure that the unit tests cover all public functions at least once, as well as all known corner cases. Long term, integrate coverage analysis tools into the development process and regularly review the coverage.
        
    2.  [ToB's Audit of Advanced Blockchains](https://github.com/trailofbits/publications/blob/master/reviews/AdvancedBlockchain.pdf)
        
162.  **Project dependencies contain vulnerabilities**: Although dependency scans did not yield a direct threat to the projects under review, yarn audit identified dependencies with known vulnerabilities. Due to the sensitivity of the deployment code and its environment, it is important to ensure dependencies are not malicious. Problems with dependencies in the JavaScript community could have a significant effect on the repositories under review.
    
    1.  Recommendation: Short term, ensure dependencies are up to date. Several node modules have been documented as malicious because they execute malicious code when installing dependencies to projects. Keep modules current and verify their integrity after installation. Long term, consider integrating automated dependency auditing into the development workflow. If dependencies cannot be updated when a vulnerability is disclosed, ensure that the codebase does not use and is not affected by the vulnerable functionality of the dependency.
        
    2.  [ToB's Audit of Advanced Blockchains](https://github.com/trailofbits/publications/blob/master/reviews/AdvancedBlockchain.pdf)
        
163.  **Lack of contract documentation makes codebase difficult to understand**: The codebase lacks code documentation, high-level descriptions, and examples, making the contracts difficult to review and increasing the likelihood of user mistakes. The documentation would benefit from more detail.
    
    1.  Recommendation: Short term, review and properly document the above mentioned aspects of the codebase. Long term, consider writing a formal specification of the protocol.
        
    2.  [ToB's Audit of Advanced Blockchains](https://github.com/trailofbits/publications/blob/master/reviews/AdvancedBlockchain.pdf)
        
164.  **ABIEncoderV2 is not production-ready**: The contracts use the new Solidity ABI encoder, _ABIEncoderV2_. This experimental encoder is not ready for production. More than 3% of all GitHub issues for the Solidity compiler are related to experimental features, primarily _ABIEncoderV2_. Several issues and bug reports are still open and unresolved. _ABIEncoderV2_ has been associated with more than [20 high-severity bugs](https://github.com/ethereum/solidity/issues?q=is:issue+abiencoderv2+label:%22bug+:bug:%22+sort:created-desc), some of which are so recent that they have not yet been included in a Solidity release. For example, in March 2019 a [severe bug](https://blog.ethereum.org/2019/03/26/solidity-optimizer-and-abiencoderv2-bug/) introduced in Solidity 0.5.5 was found in the encoder. 
    
    1.  Recommendation: Short term, use neither _ABIEncoderV2_ nor any other experimental Solidity feature. Refactor the code such that structs do not need to be passed to or returned from functions. Long term, integrate static analysis tools like Slither into your CI pipeline to detect unsafe pragmas.
        
    2.  [ToB's Audit of Advanced Blockchains](https://github.com/trailofbits/publications/blob/master/reviews/AdvancedBlockchain.pdf)
        
165.  **Contract owner has too many privileges**: The owner of the contracts has too many privileges relative to standard users. Users can lose all of their assets if a contract owner private key is compromised. The contract owner can do the following: 1) Upgrade the system’s implementation to steal funds 2) Upgrade the token’s implementation to act maliciously 3)  Increase the amount of _iTokens_ for reward distribution to such an extent that rewards cannot be disbursed 4) Arbitrarily update the interest model contracts The concentration of these privileges creates a single point of failure. It increases the likelihood that the owner will be targeted by an attacker, especially given the insufficient protection on sensitive owner private keys. Additionally, it incentivizes the owner to act maliciously.
    
    1.  Recommendation: Short term: 1) Clearly document the functions and implementations the owner can change. 2) Split privileges to ensure that no one address has excessive ownership of the system. Long term, document the risks associated with privileged users and single points of failure. Ensure that users are aware of all the risks associated with the system.
        
    2.  [ToB's Audit of dForce Lending](https://github.com/dforce-network/documents/blob/master/audit_report/Lending/dForceLending-Audit-Report-TrailofBits-Mar-2021.pdf)
        
166.  **Poor error-handling practices in test suite**: The test suite does not properly test expected behavior, as the contracts run in production. Additionally, certain components lack error-handling methods. These deficiencies can cause failed tests to be overlooked. In particular, the tests fail to properly check error messages. For example, errors are silenced with a try-catch statement. If this error is silenced, there will be no guarantee that a smart contract call has reverted for the right reason. As a result, if the test suite passes, it will provide no guarantee that the transaction call reverted correctly.
    
    1.  Recommendation: Short term, test these operations against a specific error message. Testing will ensure that errors are never silenced, and the test suite will check that a contract call has reverted for the right reason. Long term, follow standard testing practices for smart contracts to minimize the number of issues during development.
        
    2.  [ToB's Audit of dForce Lending](https://github.com/dforce-network/documents/blob/master/audit_report/Lending/dForceLending-Audit-Report-TrailofBits-Mar-2021.pdf)
        
167.  **Redundant and Unused Code**: The __recordLoanClosure()_ function returns a boolean ( _loanClosed_ ) which is never used by the calling function (see __closeLoan_() , line [312]). Furthermore, since the __recordLoanClosure_() function is only called via the __closeLoan_() function, this means that _synthLoan.timeClosed_ is always equal to zero (see _require_ statement on line [305]). Therefore, the if statement on line [357] is redundant and unnecessary.
    
    1.  Recommendation: 1) Using the return value of the __recordLoanClosure_() function or changing the function definition to stop returning _loanClosed_ 2) Removing the if statement in line [357]
        
    2.  [Sigma Prime's Audit of Synthetix EtherCollateral](https://github.com/sigp/public-audits/blob/master/synthetix/ethercollateral/review.pdf)
        
168.  **Single Account Can Capture All Supply**: The _EtherCollateral_ smart contract does not rely on a _maxLoanSize_ to limit the amount of ETH that can be locked for a loan. As a result, a single account can issue a loan that will reach the total minting supply.
    
    1.  Recommendation: Make sure this behaviour is understood and consider introducing and enforcing a cap ( _maxLoanSize_ ) on the size of the loans allowed to be opened.
        
    2.  [Sigma Prime's Audit of Synthetix EtherCollateral](https://github.com/sigp/public-audits/blob/master/synthetix/ethercollateral/review.pdf)
        
169.  **Insufficient Input Validation**: The constructor of the _EtherCollateral_ smart contract does not check the validity of the addresses provided as input parameters. It is possible to deploy an instance of the _EtherCollateral_ contract with the _synthProxy_ , _sUSDProxy_ and depot addresses set to zero. Similarly, the effective interest rate can be equal to zero if _interestRate_ is set to any value lesser than 31536000 ( _SECONDS_IN_A_YEAR_ ), as _interestPerSecond_ will be null.
    
    1.  Recommendation: Consider introducing require statements to perform adequate input validation.
        
    2.  [Sigma Prime's Audit of Synthetix EtherCollateral](https://github.com/sigp/public-audits/blob/master/synthetix/ethercollateral/review.pdf)
        
170.  **Unused Event Logs**: log events are declared but never emitted.
    
    1.  Recommendation: Remove these events from the EtherCollateral contract.
        
    2.  [Sigma Prime's Audit of Synthetix EtherCollateral](https://github.com/sigp/public-audits/blob/master/synthetix/ethercollateral/review.pdf)
        
171.  **Possible Unintended Token Burning in** _**transferFrom**_**() Function**: _InfiniGold_ allows users to convert/exchange their PMGT tokens to "gold certificates", which are digital artefacts effectively redeemable for actual gold. To do so, users are supposed to send their PMGT tokens to a specific burn address. The _transferFrom_() function does not check the to address against this burn address. Users may send tokens to the burn address, using the _transferFrom_() function, without triggering the emission of the _Burn(address indexed burner, uint256 value)_ event, which dictates how the gold certificates are created and distributed.
    
    1.  Recommendation: Prevent sending tokens to the burn address in the _transferFrom_() function. This can be achieved by adding a _require_ within _transferFrom_() which disallows the to address to be the _burnAddress_ .
        
    2.  [Sigma Prime's Audit of InfiniGold](https://github.com/sigp/public-audits/raw/master/infinigold/review.pdf)
        
172.  **Denial of Service Vector from Unbound List**: The _reset_() internal function (called by the _replaceAll_() function) resets the role linked list by deleting all the elements (i.e. nodes) part of the bearer mapping. The caller is bound by the number of elements that are being removed for a particular role . Calling the _reset_() function will exceed the current block gas limit (i.e. 8,000,0000) for more than 371 total elements in a role linked list. Similarly, the _size_() and _toArray_() functions also loop through the linked list. This essentially means that listers, unlisters, minters, pausers, unpausers and owners can perform denial of service attacks on the lists they administer. In a scenario where the Roles library is leveraged by other smart contracts, calling these two functions will also result in a potential denial of service after a certain number of elements have been included in the linked list (this number would depend on the gas cost of the Opcodes implemented by the calling functions).
    
    1.  Recommendation: One way to ensure that the current block gas limit is not exceeded would be to introduce a condition in the _add_() function to check that the linked list size is strictly lesser than 371 elements before adding a new element. This additional condition would significantly increase the gas cost associated with calling the _add_() function, as a call to the _size_() function would be required to fetch the exact number of nodes in the linked list. Alternatively, the _gasleft_() Solidity special function could be used to make sure that going through the linked list does not exceed the block gas limit. Finally, the _reset_() could be changed to allow for removing an arbitrary number of nodes (by taking this number as a function parameter).
        
    2.  [Sigma Prime's Audit of InfiniGold](https://github.com/sigp/public-audits/raw/master/infinigold/review.pdf)
        
173.  **ERC20 Implementation Vulnerable to Front-Running**: Front-running attacks involve users watching the blockchain for particular transactions and, upon observing such a transaction, submitting their own transactions with a greater gas price. This incentivises miners to prioritise the later transaction. The ERC20 implementation is known to be affected by a front-running vulnerability, in its _approve_() function.
    
    1.  Recommendation: Be aware of the front-running issues in _approve_() , potentially add extended approve functions which are not vulnerable to the front-running vulnerability for future third-party-applications. See the Open-Zeppelin [8] solution for an example. We note that modifying the ERC20 standard to address this issue may lead to backward incompatibilities with external third-party software.
        
    2.  [Sigma Prime's Audit of InfiniGold](https://github.com/sigp/public-audits/raw/master/infinigold/review.pdf)
        
174.  **Unnecessary** _**require**_ **Statement**: The following _require_ statement in _Blacklistable.sol_ can be removed: _require(to != address(0));_ Indeed, this check is implemented in the __transfer_() function in the _ERC20.sol_ smart contract.
    
    1.  Recommendation: Consider removing the require statement for gas saving purposes.
        
    2.  [Sigma Prime's Audit of InfiniGold](https://github.com/sigp/public-audits/raw/master/infinigold/review.pdf)
        
175.  **Rounding to Zero if Duration is Greater Than Reward**: The _rewardRate_ value is calculated as follows: _rewardRate = reward/duration_. Due to the integer representation of these variables, if duration is larger than reward the value of _rewardRate_ will round to zero. Thus, stakers will not receive any of the reward for their stakes. Furthermore, due to the integer rounding, the total rewards distributed may be rounded down by up to one less than duration . As a result, the Unipool contract may slowly accumulate SNX.
    
    1.  Recommendation: Beware of the rounding issues when calling the _notifyRewardAmount_() function. We also recommend some way of allowing the excess SNX reward from rounding to be claimed or withdrawn from the Unipool contract.
        
    2.  [Sigma Prime's Audit of Synthetix Unipool](https://github.com/sigp/public-audits/blob/master/synthetix/unipool/review.pdf)
        
176.  **Withdrawn Event Log Poisoning**: Calling the _withdraw_() function will emit the Withdrawn event. No UNI tokens are required as this function can be called with _amount_ = 0 . As a result a user could continually call this function, creating a potentially infinite amount of events. This can lead to an event log poisoning situation where malicious external users spam the Unipool contract to generate arbitrary Withdrawn events.
    
    1.  Recommendation: Consider adding a _require_ or _if_ statement preventing the _withdraw_() function from emitting the Withdrawn event when the amount variable is zero.
        
    2.  [Sigma Prime's Audit of Synthetix Unipool](https://github.com/sigp/public-audits/blob/master/synthetix/unipool/review.pdf)
        
177.  **Insufficient incentives to liquidator**: The liquidation process is a very important part of every DeFi project because it allows to extinguish the problem of having the whole system under-collateralized under critical conditions of the market, and it needs a design that incentivizes its speed of execution. The Holdefi contract implements the liquidation process for those accounts that may have an under-collateralized balance or that may have been inactive for a whole year without interacting with the project. The liquidator would end up paying for the expensive liquidation process, without receiving any benefit. Buying discounted collateral assets could be considered as an incentive to the liquidators
    
    1.  Recommendation: Consider improving the incentive design to give the liquidators higher incentives to execute the liquidation process
        
    2.  [OpenZeppelin's Audit of HoldeFi](https://blog.openzeppelin.com/holdefi-audit)
        
178.  **Markets can become insolvent**: When the value of all collateral is worth less than the value of all borrowed assets, we say a market is insolvent. The Holdefi codebase can do many things to reduce the risk of market insolvency, including: prudent selection of collateral-ratios, incentivizing third-party collateral liquidation, careful selection of which tokens are listed on the platform, etc. However, the risk of insolvency cannot be entirely eliminated, and there are numerous ways a market can become insolvent.
    
    1.  Recommendation: This risk is not unique to the Holdefi project. All collateralized loans (even non-blockchain loans) have a risk of insolvency. However, it is important to know that this risk does exist, and that it can be difficult to recover from even a small dip into insolvency. Consider adding more targeted tests for these scenarios to better understand the behavior of the protocol, and designing relevant mechanics to make sure the platform operates properly. Also consider communicating the potential risks to the users if needed.
        
    2.  [OpenZeppelin's Audit of HoldeFi](https://blog.openzeppelin.com/holdefi-audit)
        
179.  **Not using OpenZeppelin contracts**: OpenZeppelin maintains a library of standard, audited, community-reviewed, and battle-tested smart contracts. Instead of always importing these contracts, the Holdefi project reimplements them in some cases, while in other cases it just copies them. This increases the amount of code that the Holdefi team will have to maintain and misses all the improvements and bug fixes that the OpenZeppelin team is constantly implementing with the help of the community.
    
    1.  Recommendation: Consider importing the OpenZeppelin contracts instead of reimplementing or copying them. These contracts can be extended to add the extra functionalities required by Holdefi.
        
    2.  [OpenZeppelin's Audit of HoldeFi](https://blog.openzeppelin.com/holdefi-audit)
        
180.  **Lack of indexed parameters in events**: Throughout the Holdefi’s codebase, none of the parameters in the events defined in the contracts are indexed.
    
    1.  Recommendation: Consider indexing event parameters to avoid hindering the task of off-chain services searching and filtering for specific events.
        
    2.  [OpenZeppelin's Audit of HoldeFi](https://blog.openzeppelin.com/holdefi-audit)
        
181.  **Named return variables**: There is an inconsistent use of named return variables across the entire codebase. 
    
    1.  Recommendation: Consider removing all named return variables, explicitly declaring them as local variables in the body of the function, and adding the necessary explicit return statements where appropriate. This should favor both explicitness and readability of the project.
        
    2.  [OpenZeppelin's Audit of HoldeFi](https://blog.openzeppelin.com/holdefi-audit)
        
182.  **block.timestamp Unreliable**: Code uses the _block.timestamp_ as part of the calculations and time checks. Nevertheless, timestamps can be slightly altered by miners to favor them in contracts that have logics that depend strongly on them.
    
    1.  Recommendation: Consider taking into account this issue and warning the users that such a scenario could happen. If the alteration of timestamps cannot affect the protocol in any way, consider documenting the reasoning and writing tests enforcing that these guarantees will be preserved even if the code changes in the future.
        
    2.  [OpenZeppelin's Audit of HoldeFi](https://blog.openzeppelin.com/holdefi-audit)
        
183.  **Assignment in** _**require**_ **statement**: In the _YieldOracle_ contract, there is a _require_ statement that makes an assignment. This deviates from the standard usage and intention of _require_ statements and can easily lead to confusion.
    
    1.  Recommendation: Consider moving the assignment to its own line before the _require_ statement and then using the _require_ statement solely for condition checking.
        
    2.  [OpenZeppelin's Audit of BarnBrige Smart Yield Bonds](https://blog.openzeppelin.com/barnbridge-smart-yield-bonds-audit/)
        
184.  **Commented code**: Throughout the codebase there are lines of code that have been commented out with //. This can lead to confusion and is detrimental to overall code readability. 
    
    1.  Recommendation: Consider removing commented out lines of code that are no longer needed.
        
    2.  [OpenZeppelin's Audit of BarnBrige Smart Yield Bonds](https://blog.openzeppelin.com/barnbridge-smart-yield-bonds-audit/)
        
185.  **Misleading** _**revert**_ **messages**: Error messages are intended to notify users about failing conditions, and should provide enough information so that the appropriate corrections needed to interact with the system can be applied. Uninformative error messages greatly damage the overall user experience, thus lowering the system’s quality. 
    
    1.  Recommendation: Consider not only fixing the specific issues mentioned, but also reviewing the entire codebase to make sure every error message is informative and user-friendly enough. Furthermore, for consistency, consider reusing error messages when extremely similar conditions are checked.
        
    2.  [OpenZeppelin's Audit of Compound Governor Bravo](https://blog.openzeppelin.com/compound-governor-bravo-audit/)
        
186.  **Multiple outdated Solidity versions in use**: Outdated versions of Solidity are being used in all contracts. The compiler options in the truffle-config file specifies version 0.6.6, which was released on April 6, 2020. Throughout the codebase there are also different versions of Solidity being used. 
    
    1.  Recommendation: As Solidity is now under a fast release cycle, consider using a more recent version of the compiler, such as version 0.7.6. In addition, to avoid unexpected behavior, consider specifying explicit Solidity versions in pragma statements.
        
    2.  [OpenZeppelin's Audit of Fei Protocol](https://blog.openzeppelin.com/fei-protocol-audit/)
        
187.  **Test and production constants in the same codebase**: The _CoreOrchestrator_ contract defines the _TEST_MODE_ boolean variable which is used to define several constants in the system. This decreases legibility of production code, and makes the system’s integral values more error-prone. 
    
    1.  Recommendation: Consider having different environments for production and testing, with different contracts.
        
    2.  [OpenZeppelin's Audit of Fei Protocol](https://blog.openzeppelin.com/fei-protocol-audit/)
        
188.  **Unnecessarily small integer sizes**: In Solidity, using integers smaller than 256 bits tends to increase gas costs because the Ethereum Virtual Machine must perform additional operations to zero out the unused bits. This can be justified by savings in storage costs in some scenarios, however, that is not generally the case in this codebase.
    
    1.  Recommendation: Consider using integers of size 256 bits to improve gas efficiency and mitigate function reverts.
        
    2.  [OpenZeppelin's Audit of Fei Protocol](https://blog.openzeppelin.com/fei-protocol-audit/)
        
189.  **Use of** _**uint**_ **instead of** _**uint256**_: Across the codebase, there are hundreds of instances of _uint_, as opposed to _uint256_.
    
    1.  Recommendation: In favor of explicitness, consider replacing all instances of _uint_ with _uint256_.
        
    2.  [OpenZeppelin's Audit of Fei Protocol](https://blog.openzeppelin.com/fei-protocol-audit/)
        
190.  **Functions with unexpected side-effects**: Some functions have side-effects. For example, the __getLatestFundingRate_ function of the _FundingRateApplier_ contract might also update the funding rate and send rewards. The getPrice function of the _OptimisticOracle_ contract might also settle a price request. These side-effect actions are not clear in the name of the functions and are thus unexpected, which could lead to mistakes when the code is modified by new developers not experienced in all the implementation details of the project.
    
    1.  Recommendation: Consider splitting these functions in separate getters and setters. Alternatively, consider renaming the functions to describe all the actions that they perform.
        
    2.  [OpenZeppelin's Audit of Uma Phase 4](https://blog.openzeppelin.com/uma-audit-phase-4/)
        
191.  **Unsafe casting**: In line 554 of the _TaxCollector_ contract, the value of _coinBalance(receiver)_ is an _uint_. This is cast to an _int_ and then negated. However, since _uint_ can store higher values than _int_, it is possible that casting from _uint_ to _int_ may create an overflow.
    
    1.  Recommendation: Consider verifying that the value of _coinBalance(receiver)_ is within the acceptable range for negative _int_ values before casting and negating. Consider using OpenZeppelin’s _SafeCast_ contract, which provides functions for safely casting between types.
        
    2.  [OpenZeppelin's Audit of GEB Protocol](https://blog.openzeppelin.com/geb-protocol-audit/)
        
192.  **Missing error messages in** _**require**_ **statements**: There are many places where _require_ statements are correctly followed by their error messages, clarifying what was the triggered exception. However, there are places where _require_ statements are not followed by the corresponding error messages. If any of those _require_ statements were to fail the checked condition, the transaction  
    would revert silently without an informative error message.
    
    1.  Recommendation: Consider including specific and informative error messages in all _require_ statements.
        
    2.  [OpenZeppelin's Audit of GEB Protocol](https://blog.openzeppelin.com/geb-protocol-audit/)
        
193.  **Uncommented assembly block**: The _OracleRelayer_ contract includes an assembly block in the _rpower_() function. The same assembly block is repeated in the _TaxCollector_ and _CoinSavingsAccount_ contracts. While this does not pose a security risk per se, it is at the same time a complicated and critical part of the system. Moreover, as this is a low-level language that is harder to parse by readers, consider including extensive documentation regarding the rationale behind its use, clearly explaining what every single assembly instruction does. This will make it easier for users to trust the code, for reviewers to verify it, and for developers to build on top of it or update it. Note that the use of assembly discards several important safety features of Solidity, which may render the code unsafer and more error-prone.
    
    1.  Recommendation: Consider implementing thorough tests to cover all potential use cases of these functions to ensure they behave as expected.
        
    2.  [OpenZeppelin's Audit of GEB Protocol](https://blog.openzeppelin.com/geb-protocol-audit/)
        
194.  **Unnecessary** _**require**_ **statements**: There are several instances in the code base where the _require_ statements or conditional checks are unnecessary. For instance: In the _OracleRelayer_ contract, the _require_ statement in the _modifyParameters_ function at line 189 checks if the input parameter data > 0. This is unnecessary since the same condition is already checked in the _require_ statement at line 187.
    
    1.  Recommendation: To simplify the code and prevent wastage of gas, consider removing the unnecessary checks.
        
    2.  [OpenZeppelin's Audit of GEB Protocol](https://blog.openzeppelin.com/geb-protocol-audit/)
        
195.  **Unnecessary event emission**: The _popDebtFromQueue_ function of the _AccountingEngine_ contract is emitting a useless event whenever someone tries to call it with a _debtBlockTimestamp_ that has not been saved before.
    
    1.  Recommendation: To simplify the code and prevent wastage of gas, avoid emitting unnecessary events.
        
    2.  [OpenZeppelin's Audit of GEB Protocol](https://blog.openzeppelin.com/geb-protocol-audit/)
        
196.  _**oToken**_ **can be created with a non-whitelisted collateral asset**: A product consists of a set of assets and an option type. Each product has to be whitelisted by the admin using the _whitelistProduct_ function from the Whitelist contract.
    
    1.  Recommendation: Consider validating if the assets involved in a product have been already whitelisted before allowing the creation of _oTokens_.
        
    2.  [OpenZeppelin's Audit of Opyn Gamma Protocol](https://blog.openzeppelin.com/opyn-gamma-protocol-audit/)
        
197.  **Mismatches between contracts and interfaces**: Interfaces define the exposed functionality of the implemented contracts. However, in several interfaces there are functions from the counterpart contracts that are not defined. 
    
    1.  Recommendation: Consider applying the necessary changes in the mentioned interfaces and contracts so that definitions and implementations fully match.
        
    2.  [OpenZeppelin's Audit of Opyn Gamma Protocol](https://blog.openzeppelin.com/opyn-gamma-protocol-audit/)
        
198.  **Actions not executed atomically might lead to inconsistent state**: The _setAssetPricer_, _setLockingPeriod,_ and _setDisputePeriod_ functions of the Oracle contract execute actions that are always expected to be performed atomically. Failing to do so can lead to inconsistent states in the system.
    
    1.  Recommendation: Consider implementing an additional function that calls the _setAssetPricer_, _setLockingPeriod_, and _setDisputePeriod_ functions, so that these actions can be executed atomically in a single transaction.
        
    2.  [OpenZeppelin's Audit of Opyn Gamma Protocol](https://blog.openzeppelin.com/opyn-gamma-protocol-audit/)
        
199.  **Chainlink pricer is using a deprecated API**: The Chainlink Pricer is currently using multiple functions from a deprecated Chainlink API such as _latestAnswer_() in L61, _getTimestamp_() in L74. These functions might suddenly stop working if Chainlink stopped supporting deprecated APIs.
    
    1.  Recommendation: Consider refactoring these to use the latest Chainlink API.
        
    2.  [OpenZeppelin's Audit of Opyn Gamma Protocol](https://blog.openzeppelin.com/opyn-gamma-protocol-audit/)
        
200.  **Funds can be lost**: The _sweepTimelockBalances_ function accepts a list of users with unlocked balances to distribute. However, if there are duplicate users in the list, their balances will be counted multiple times when calculating the total amount to withdraw from the yield service.
    
    1.  Recommendation: Consider checking for duplicate users when calculating the amount to withdraw.
        
    2.  [OpenZeppelin's Audit of PoolTogether V3](https://blog.openzeppelin.com/pooltogether-v3-audit/)
        
201.  **Use** _**delete**_ **to clear variables**: The Controller contract sets a variable to the zero address in order to clear it. Similarly, the _SetToken_ clears the locker by assigning the zero address.
    
    1.  Recommendation: The _delete_ key better conveys the intention and is also more idiomatic. Consider replacing assignments of zero with _delete_ statements.
        
    2.  [OpenZeppelin's Audit of Set Protocol](https://blog.openzeppelin.com/set-protocol-audit/)