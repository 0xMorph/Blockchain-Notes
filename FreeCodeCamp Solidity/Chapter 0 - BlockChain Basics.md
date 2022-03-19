## Chapter 0: BlockChain Basics
Public to private keys for wallets use the `Elliptic Curve Digital Signature Algorithm`

We use Metamask as our wallet, for testing we can add funds through a Faucet like Rinkeby Faucet (Check the [link token contracts page](https://docs.chain.link/docs/link-token-contracts/#rinkeby)) OR, use the [Kovan ETH Faucet](https://faucets.chain.link/)

#### BlockChains
Blockchains can't be changed in the pass, it's peer to peer and generates hashes from previous hashes and if one in the pass is changed, the rest in the future changes as well, invalidating that change. - [Blockchain Demo](https://andersbrownworth.com/blockchain/)

Nothing can change or corrupt the blockchain.

#### Prove of Work (Consensus) 
^ not called this if we're at ETH 2. It verifies who mined the hash. Cost lots of electricity though, but Proof of Stake seeks to solve this, included in ETH 2.

Split to `Chain Selection` and `Sybil Resistance` 

`Chain Selection` uses Nakamoto Consensus that picks the longest chain to be validated. The 51% attack counters this by having the longest chain.

`Sybil Resistance` uses Proof of Stake that puts up collateral as a sybil resistance. So each possible anonymous account needs to put up collateral.

#### Gas Price
Each transaction made costs a price, if more people make transactions, the more the gas price. So this is not scalable.

But ETH 2 solves it via Proof of Stake and sharding, making a blockchain within another block chain.

#### Layers
Layer 1 = Bitcoin, Ethereum, any original blockchain
Layer 2 = Any application built on top of layer 1, like Optimism