Term | Definition
----|-----------
Assert | When false, compiles to Oxfe, code needs to be fixed asap, should use `assert()` to avoid conditions that should never occur
Big-endian | Most significant digit is first, a positional number representation
BIPs | Bitcoin improvement proposals, ie BIP-21 seeks to improve Bitcoin's URI scheme
Blocks | collection of info on the comprised transactions, other block headers are known as `ommers`. Blocks are added to the Ethereum network by miners.
Blockchain | sequence of blocks validated by the proof-of-work system, linking each blocks back to the genesis block. Unlike Bitcoin, no block size limit, instead uses varying gas limits.
Bytecode | abstract instructions designed for efficient execution, expressed in numeric
Byzantium Fork | first 2 hard forks for the Metropolis dev stage, includes EIP-649: Metropolis Difficulty Bomb Delay and Block Reward Reduction, where `Ice Age` was delayed by 1 year and block reward was reduced from 5 to 3 ether.
Consensus | When most nodes on the network have the same block in their locally validated best blockchain
Consensus Rules | Block validation rules that full nodes follow to stay in consensus with other nodes
Constantinople Fork | 2nd part f the Metropolis stage, expected to include a switch to a hybrid proof-of-work/proof-of-stake consensus algorithm.
Contract Account | Account containing code that executes when a transaction is received
Contract creation transaction | special transaction with 'zero address' as the recipient, that is used to register a contract and record it on the Ethereum blockchain
DAO | Decentralized Autonomous Organization, company without hierarchical management
DApp | Decentralized Application
Deed | NFT standard, the deed proves ownership and is not interchangeable, for now not recognised as legal documents in any jurisdiction
Difficulty | network-wide setting controlling how much computation is required to produce a proof of work
Digital Signature | short string of data a user produces for a document using a private key, so anyone with the corresponding public key so that the document can verify that the document was "signed" by the owner of the private key and the document was not changed after it was signed
ECDSA | Elliptic Curve Digital Signature Algorithm, cryptograhy used by Ethereum to ensure funds can only be spent by their owners
EIP | Ethereum Improvement Proposal, 
ENS | Ethereum name service
Entropy | under cryptography, lack of predictability or level of randomness
EOA | externally owned account, created by or for human users of the ETH network
ETC | Ethereum request for Comments, label for some EIPs that attempt to define a specific standard of ETH usage.
Ethash | proof-of-work algorithm of ETH 1.0
Ether | native cryptocurrency used by Ethereum ecosystem, covering gas costs when executing smart contracts
Event | allows the use of EVM logging facilities, DApps listen for events and use them to trigger JS callbacks in the user interface.
EVM | Ethereum Virtual Machine, stack-based vm that executes bytecode. In ETH, the execution model specifies how the system state is altered given a series of bytecode and small environment data
EVM Assembly Language | human readable form of EVM bytecode
Fallback Function | default function called in absence of data or a declared function name
Faucet | server that dispenses test ether that can be used on a testnet
Finney | denomination of ether. 1 finney = 10^15 wei, 10^3 finney = 1 ether.
Fork | change in protocol causing the creation of an alternative chain, or a temporal divergence in 2 potential block paths during mining
Frontier | test dev stage of Ethereum, July 2015 to March 2016
Ganache | personal ETH blockchain for testing and inspecting state while controlling how the chain operates
Gas | Virtual fuel used in ETH to execute smart contracts, the EVM uses an accounting mechanism to measure the gas consumption and limit the consumtion of computing resources
Gavin Wood | CTO of Ethereum and cofounder
Genesis Block | first block in a blockchain, used to initialize a network and its cryptocurrency
Geth | Go Ethereum, ETH protocol written in Go
Hard Fork | Permanent divergence in the blockchain, commonly occurs when nonupgraded noded can't validate blocks created by upgraded node that follow newer consenses rules.
Hash | fixed-length fingerprint of a variable-size input, produced via a hash function
HD wallet | wallet using hierarchical deterministic key creation and transfer protocol
HD wallet seed | value used to generate the master private key and master chain node of an HD wallet, can be represented by mnemonic works, easier to copy, backup and restore private keys
Homestead | 2nd dev stage of ETH, launched at March 2016 at block #1,150,000
ICAP | Inter-exchange Client Address Protocol, an ETH address thats compatible with IBAN encoding, offers a versatile, checksumed and interoperable encoding for ETH address. Also uses a new IBAN pseudo-country code "XE" for "extended ETH", used in nonjurisdictional currencies.
Ice age | Hard fork of ETH at block #200,000 introducing an exponential dificulty increase, motivating a transition to proof-of-stake
Immutable deployed code problem | once a contract is deployed, its immutable, cannot be changed. 
Internal transaction (aka message) | transaction sent from a contract account to another account or an EOA
IPFS | InterPlanetary File System
KDF | Key derivation function, aka password stretching algorithm used to keystore formats to protect against bruteforcing, dictionary and rainbow table attacks
Keccak-256 | Cryptographic hash function
Keystone File | JSON encoded file containing a single randomly generated private key, encrypted with a passphrase
LevelDB | Open source on-disk key-value store, used as a lightweight, single purpose library, with bindings to many platforms
Library | Special contract with no payable, no fallback and no data storage function. Other contracts can call this contract for read-only computation
Lightweight Client | An ETH client that does not store a local copy of the blockchain, or validate blocks and transactions. Offers functions of a wallet and can create and broadcast transactions.
Merkle Patricia Tree | Data structure used in ETH to efficiently store key-value pairs
Message | internal transaction that is never serialized and only sent within the EVM
Message call | Act of passing a message from one accoutn to another, if the destination account is associated with EVM code, the VM will be started with the state of that object and the message acted upon.
Metropolis | 3rd dev stage of ETH, launched October 2017
Miner | Network node that finds valid proof of work for new blocks, by repeated hashing
Mist | First ETH enabled browser, contains a browser-based wallet, first wallet to introduce camelCase checksum
Network | ETH network, a peer-to-peer network that propagates transactions and blaocks to every ETH node
NFT | Non-fungible token, aka a deed, introduced through ERC721, each token is distinct and unique, representing ownership of digital or physical assets
Node | software client that participates in the network
Nonce | Value that can only be used once, theres 2 types used in ETH, account nonce is a transaction counter in each account used to prevent replay attacks, a proof-of-work nonce is the random value in a block used to satisfy the proof-of-work
Ommer | Child block of an ancestor that is not an ancestor itself, when a miner finds a valid block, another miner may have published a competing block added to the tip of the blockchain. Unlike BTC, orphaned blocks in ETH can be included as ommers and receive a partial block award.
Parity | one of the most prominent interoperable implementations of ETH client software
Private key | secret key
Proof-of-stake | method by a crypto blockchain aiming to achieve distributed consensus. PoS asks users to prove ownership of a certain ammount of crypto(their "stake" in the network) to participate in the validation of transactions
Proof-of-work | piece of data requiring significant computation to find. In ETH, miners must find a numeric solution to the Ethash algorithm that meets a network-wide difficulty target
Public Key | digits derived 1-way from a private key, shared publicly and used by anyone to verify a digital signature made with the corresponding private key.
Receipt | Data returned by an ETH client to represent the result from a transaction, including a hash of the transaction, its block number, gas used, and in case of deplyment of a smart contract, address of the contract
Re-entrancy attack | attack where an attacker contract calls a victim contract function in a way that during execution, the victim calls the attackers contract again, recursively. Resulting in theft of funds skipping parts of the victim's contract that update balances or count withdrawal amounts.
Reward | amount of ether included in each new block as a reward by the network to the miner with the PoW
RLP | Recursive Length Prefix, an encoding standard by ETH devs to encode and serialize objects (data structures) of arbitrary complexity and length
Secret Key | secret digits allowing ETH users to prove ownership of an account or contracts, by producing a digital signature
Serenity | 4th and final dev stage of ETH. Not yet having a planned release date
Serpent | A procedural (crucial) smart contract programming language with syntax like Python
SHA | Secure hash algorithm, published by NIST
Singleton | programming term for an object of only a single instance in existance
Smart Contract | program that executes on the ETH computing interface
Solidity | Procedural (imperative) programming language with syntax like JS, C++ and Java
Solidity inline assembly | EVM assembly language in a Solidity program
Spurious Dragon | Hard fork of the ETH blockchain, at block #2,675,000 to address more DoS attack vectors and clear states, also a replay attack protection mechanism
Swarm | Decentralized (P2P) storage network, used with Web3 and Whisper to build DApps
Szabo | A denomination of Ether, 1 szabo = 10^12 wei, 1 ether = 10^6 szabo
Tangerine Whistle | Hard fork of ETH blockchain at block #2,463,000 to change gas calculation for certain I/O intensive operations and clear the accumulated state from DoS attacks
Testnet | Simulation network for the main ETH network
Transaction | Data committed to the ETH blockchain signed by an originating account targeting a specific address, the transaction contains metadata like the gas limits for that transaction
Truffle | Commonly used ETH dev framework
Turing complete | system of data manipulation rules is said to be "Turing Complete" if it can be used to simulate any Turing machine
Vyper | like Serpent, intended to get closer to a pure fictional language
Wallet | Software that holds secret keys, but keys can also be retrieved from offline storage, never actually stores the coins and tokens
Web3 | 3rd version of web, represents a focus for decentralized protocols for applications
Wei | Smallet denomination of ether 10^18 wei = 1 ether
Whisper | Decentralied (P2P) messaging service, used with Web3 and Swarm to build Dapps
Zero Address | special ETH address composed of only zeros, specified as the destination address of a contract creation transaction.




