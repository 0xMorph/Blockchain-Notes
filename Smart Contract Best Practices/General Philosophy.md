## Prepping for Failure
- Pause contract when things go wrong ("circuit breaker")
- Manage the amount of money at risk (rate limiting, maximum usage)
- Have an effective upgrade path for bugfixes and improvements

## Staying up to data
- Check contracts when new bug is discovered
- Upgrade all versions of tools and libraries asap
- Adopt new security techniques that are useful

## Simplicity wins
- Keep contracts and functions small through modularizing code
- Use already written tools or code
- Clarity > Performance
- Use blockchain for parts of system that require decentralization

## Rolling out
- Test every attack vectors
- Rollout in phases with increased usage and testing each phase

## Blockchain Properties
- Vary of external contract calls
- Private data in smart contracts is also viewable by everyone
- Gas costs and block gas limit keep  in mind
- Timestamps are imprecise on the blockchain, miners can influence it by several seconds
- Randomness is non-trivial, most approaches are flawed and gameable on a blockchain

## Simplicity vs Complexity
The ideal smart contract:
- Reuses code
- Supports upgradeable components

#### Rigid vs Upgradeable
With upgradable features like `Killable`, `Upgradeable` or `Modifiable` there's a tradeoff between security and malleability.

Like a governance-free finite-time-frame token-sale contract, it's simple, this no complex and potential attack surfaces.

#### Monolithic vs Modular
Monolithic keeps all knowledge locally identifiable and readable, only a few contracts exists as monoliths. 

In blockchain we trend towards long living complex contract systems compared to traditional short lived ones.

#### Duplication vs Reuse
Using proven previously-deployed contracts "we" own is safest to achive code reuse. Even [OpenZeppelin's Solidity Library](https://github.com/OpenZeppelin/openzeppelin-contracts) seek to provide patterns such that secure code can be re-used without duplication.