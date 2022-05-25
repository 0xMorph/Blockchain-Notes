## Solidity 101
1.  Solidity is a high-level language for implementing smart contracts on Ethereum (and the blockchains) targeting the EVM. Solidity was proposed in 2014 by Gavin Wood and was later developed by Ethereum’s Solidity team, led by Christian Reitwiessner, Alex Beregszaszi & others. (See [here](https://en.wikipedia.org/wiki/Solidity))
    
2.  It is influenced mainly by C++, a little from Python and early-on from JavaScript. The syntax and OOP concepts are from C++.  Solidity’s modifiers, multiple inheritance, C3 linearization and the super keyword are influences from Python. Function-level scoping and var keyword were JavaScript influences early-on but those have been reduced since v0.4.0. (See [here](https://docs.soliditylang.org/en/v0.8.9/language-influences.html))
    
3.  Solidity is statically typed, supports inheritance, libraries and complex user-defined types. It is a fully-featured high-level language.
    
4.  The layout of a Solidity source file can contain an arbitrary number of pragma directives, import directives and struct/enum/contract definitions. The best-practices for layout within a contract is the following order: state variables, events, modifiers, constructor and functions.
    
5.  SPDX License Identifier: Solidity source files are recommended to start with a comment indicating its license e.g.:“_// SPDX-License-Identifier: MIT_”, where the compiler includes the supplied string in the bytecode metadata to make it machine readable. SPDX stands for Software Package Data Exchange (See [SPDX](https://spdx.org/)).
    
6.  Pragmas: The pragma keyword is used to enable certain compiler features or checks. A pragma directive is always local to a source file, so you have to add the pragma to all your files if you want to enable it in your whole project. If you import another file, the pragma from that file does not automatically apply to the importing file. There are two types: 1) Version: a)  Compiler version b) ABI Coder version 2) Experimental: a) SMTChecker
    
7.  Version Pragma: This indicates the specific Solidity compiler version to be used for that source file and is used as follows: “_pragma solidity x.y.z;_” where x.y.z indicates the version of the compiler.
    
    1.  Using the version pragma does not change the version of the compiler. It also does not enable or disable features of the compiler. It just instructs the compiler to check whether its version matches the one required by the pragma. If it does not match, the compiler issues an error.
        
    2.  The latest compiler versions are in the 0.8.z range
        
    3.  A different y in x.y.z indicates breaking changes e.g. 0.6.0 introduces breaking changes over 0.5.z. A different z in x.y.z indicates bug fixes.
        
    4.  A ‘^’ symbol prefixed to x.y.z in the pragma indicates that the source file may be compiled only from versions starting with x.y.z until x.(y+1).z. For e.g., “_pragma solidity ^0.8.3;_” indicates that source file may be compiled with compiler version starting from 0.8.3 until any 0.8.z but not 0.9.z. This is known as a “floating pragma.”
        
    5.  Complex pragmas are also possible using ‘>’,’>=‘,’<‘ and ‘<=‘ symbols to combine multiple versions e.g. “_pragma solidity >=0.8.0 <0.8.3;_”
        
8.  ABI Coder Pragma: This indicates the choice between the two implementations of the ABI encoder and decoder: “_pragma abicoder v1;_” or “_pragma abicoder v2;”_ 
    
    1.  The new ABI coder (v2) is able to encode and decode arbitrarily nested arrays and structs. It might produce less optimal code and has not received as much testing as the old encoder. This is activated by default.
        
    2.  The set of types supported by the new encoder is a strict superset of the ones supported by the old one. Contracts that use it can interact with ones that do not without limitations. The reverse is possible only as long as the non-abicoder v2 contract does not try to make calls that would require decoding types only supported by the new encoder. The compiler can detect this and will issue an error. Simply enabling abicoder v2 for your contract is enough to make the error go away.
        
    3.  This pragma applies to all the code defined in the file where it is activated, regardless of where that code ends up eventually. This means that a contract whose source file is selected to compile with ABI coder v1 can still contain code that uses the new encoder by inheriting it from another contract. This is allowed if the new types are only used internally and not in external function signatures.
        
9.  Experimental Pragma: This can be used to enable features of the compiler or language that are not yet enabled by default
    
    1.  SMTChecker: The use of “_pragma experimental SMTChecker;”_ performs additional safety checks which are obtained by querying an SMT solver (See [SMTChecker](https://docs.soliditylang.org/en/v0.8.9/smtchecker.html#formal-verification))
        
    2.  The SMTChecker module automatically tries to prove that the code satisfies the specification given by require and assert statements. That is, it considers require statements as assumptions and tries to prove that the conditions inside assert statements are always true. If an assertion failure is found, a counterexample may be given to the user showing how the assertion can be violated. If no warning is given by the SMTChecker for a property, it means that the property is safe.
        
    3.  Other checks: Arithmetic underflow and overflow, Division by zero, Trivial conditions and unreachable code, Popping an empty array, Out of bounds index access, Insufficient funds for a transfer.
        
10.  Imports: Solidity supports import statements to help modularise your code that are similar to those available in JavaScript (from ES6 on) e.g. “_import “filename”;_"
    
11.  Comments: Single-line comments (//) and multi-line comments (/*...*/) are possible. Comments are recommended as in-line documentation of what contracts, functions, variables, expressions, control and data flow are expected to do as per the implementation, and any assumptions/invariants made/needed. They help in readability and maintainability.
    
12.  NatSpec Comments: NatSpec stands for “Ethereum Natural Language Specification Format.” These are written with a triple slash (///) or a double asterisk block(/** ... */) directly above function declarations or statements to generate documentation in JSON format for developers and end-users. It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). These comments contain different types of tags:
    
    1.  @title: A title that should describe the contract/interface
        
    2.  @author: The name of the author (for contract, interface)
        
    3.  @notice: Explain to an end user what this does (for contract, interface, function, public state variable, event)
        
    4.  @dev: Explain to a developer any extra details (for contract, interface, function, state variable, event)
        
    5.  @param: Documents a parameter (just like in doxygen) and must be followed by parameter name (for function, event)
        
    6.  @return: Documents the return variables of a contract’s function (function, public state variable)
        
    7.  @inheritdoc: Copies all missing tags from the base function and must be followed by the contract name (for function, public state variable)
        
    8.  @custom…: Custom tag, semantics is application-defined (for everywhere)
        
13.  Contracts: They are similar to classes in object-oriented languages in that they contain persistent data in state variables and functions that can modify these variables. Contracts can inherit from other contracts
    
14.  Contracts can contain declarations of State Variables, Functions, Function Modifiers, Events, Errors, Struct Types and Enum Types
    
15.  State Variables: They are variables that can be accessed by all functions of the contract and whose values are permanently stored in contract storage
    
16.  State Visibility Specifiers: State variables have to be specified as being public, internal or private:
    
    1.  public: Public state variables are part of the contract interface and can be either accessed internally or via messages. An automatic getter function is generated.
        
    2.  internal: Internal state variables can only be accessed internally from within the current contract or contracts deriving from it
        
    3.  private: Private state variables can only be accessed from the contract they are defined in and not even in derived contracts. Everything that is inside a contract is visible to all observers external to the blockchain. Making variables private only prevents other contracts from reading or modifying the information, but it will still be visible to the whole world outside of the blockchain.
        
17.  State Variables: Constant & Immutable
    
    1.  State variables can be declared as _constant_ or _immutable_. In both cases, the variables cannot be modified after the contract has been constructed. For _constant_ variables, the value has to be fixed at compile-time, while for _immutable_, it can still be assigned at construction time i.e. in the constructor or point of declaration.
        
    2.  For constant variables, the value has to be a constant at compile time and it has to be assigned where the variable is declared. Any expression that accesses storage, blockchain data (e.g. block.timestamp, address(this).balance or block.number) or execution data (msg.value or gasleft()) or makes calls to external contracts is disallowed.
        
    3.  Immutable variables can be assigned an arbitrary value in the constructor of the contract or at the point of their declaration. They cannot be read during construction time and can only be assigned once.
        
    4.  The compiler does not reserve a storage slot for these variables, and every occurrence is replaced by the respective value.
        
18.  Compared to regular state variables, the gas costs of constant and immutable variables are much lower:
    
    1.  For a constant variable, the expression assigned to it is copied to all the places where it is accessed and also re-evaluated each time. This allows for local optimizations.
        
    2.  Immutable variables are evaluated once at construction time and their value is copied to all the places in the code where they are accessed. For these values, 32 bytes are reserved, even if they would fit in fewer bytes. Due to this, constant values can sometimes be cheaper than immutable values.
        
    3.  The only supported types are strings (only for constants) and value types.
        
19.  Functions: Functions are the executable units of code. Functions are usually defined inside a contract, but they can also be defined outside of contracts. They have different levels of visibility towards other contracts.
    
20.  Function parameters: Function parameters are declared the same way as variables, and the name of unused parameters can be omitted. Function parameters can be used as any other local variable and they can also be assigned to.
    
21.  Function Return Variables: Function return variables are declared with the same syntax after the _returns_ keyword. 
    
    1.  The names of return variables can be omitted. Return variables can be used as any other local variable and they are initialized with their default value and have that value until they are (re-)assigned.
        
    2.  You can either explicitly assign to return variables and then leave the function as above, or you can provide return values (either a single or multiple ones) directly with the return statement
        
    3.  If you use an early return to leave a function that has return variables, you must provide return values together with the return statement
        
    4.  When a function has multiple return types, the statement return (v0, v1, ..., vn) can be used to return multiple values. The number of components must be the same as the number of return variables and their types have to match, potentially after an implicit conversion
        
22.  Function Modifiers: They can be used to change the behaviour of functions in a declarative way. For example, you can use a modifier to automatically check a condition prior to executing the function. The function’s control flow continues after the “_” in the preceding modifier. Multiple modifiers are applied to a function by specifying them in a whitespace-separated list and are evaluated in the order presented. The modifier can choose not to execute the function body at all and in that case the return variables are set to their default values just as if the function had an empty body. The _ symbol can appear in the modifier multiple times. Each occurrence is replaced with the function body.
    
23.  Function Visibility Specifiers: Functions have to be specified as being _public_, _external_, _internal_ or _private:_
    
    1.  _public:_ Public functions are part of the contract interface and can be either called internally or via messages. 
        
    2.  _external:_ External functions are part of the contract interface, which means they can be called from other contracts and via transactions. An external function f cannot be called internally (i.e. f() does not work, but this.f() works).
        
    3.  _internal:_ Internal functions can only be accessed internally from within the current contract or contracts deriving from it
        
    4.  _private:_ Private functions can only be accessed from the contract they are defined in and not even in derived contracts
        
24.  Function Mutability Specifiers: Functions can be specified as being _pure_ or _view_:
    
    1.  _view_ functions can read contract state but cannot modify it. This is enforced at runtime via _STATICCALL_ opcode. The following are considered state modifying: 1) Writing to state variables 2) Emitting events 3) Creating other contracts 4) Using selfdestruct 5) Sending Ether via calls 6) Calling any function not marked view or pure 7) Using low-level calls 8) Using inline assembly that contains certain opcodes.
        
    2.  _pure_ functions can neither read contract state nor modify it. The following are considered reading from state: 1) Reading from state variables 2) Accessing address(this).balance or <address>.balance 3) Accessing any of the members of block, tx, msg (with the exception of msg.sig and msg.data) 4) Calling any function not marked pure 5) Using inline assembly that contains certain opcodes.
        
    3.  It is not possible to prevent functions from reading the state at the level of the EVM. It is only possible to prevent them from writing to the state via _STATICCALL._ Therefore,  only _view_ can be enforced at the EVM level, but not _pure_.
        
25.  Function Overloading: A contract can have multiple functions of the same name but with different parameter types. This process is called “overloading.”
    
    1.  Overloaded functions are selected by matching the function declarations in the current scope to the arguments supplied in the function call.
        
    2.  Return parameters are not taken into account for overload resolution.
        
26.  Free Functions: Functions that are defined outside of contracts are called “free functions” and always have implicit internal visibility. Their code is included in all contracts that call them, similar to internal library functions.
    
27.  Events: They are an abstraction on top of the EVM’s logging functionality. Emitting events cause the arguments to be stored in the transaction’s log - a special data structure in the blockchain. These logs are associated with the address of the contract, are incorporated into the blockchain, and stay there as long as a block is accessible. The Log and its event data is not accessible from within contracts (not even from the contract that created them). Applications can subscribe and listen to these events through the RPC interface of an Ethereum client.
    
28.  Indexed Event Parameters: Adding the attribute _indexed_ for up to three parameters adds them to a special data structure known as “topics” instead of the data part of the log. If you use arrays (including string and bytes) as indexed arguments, its Keccak-256 hash is stored as a topic instead, this is because a topic can only hold a single word (32 bytes). All parameters without the _indexed_ attribute are ABI-encoded into the data part of the log. Topics allow you to search for events, for example when filtering a sequence of blocks for certain events. You can also filter events by the address of the contract that emitted the event.
    
29.  Emit: Events are emitted using `emit`, followed by the name of the event and the arguments e.g. “_emit Deposit(msg.sender, _id, msg.value);_”
    
30.  Struct Types: They are custom defined types that can group several variables of same/different types together to create a custom data structure. The struct members are accessed using ‘.’ e.g.: struct s {address user; uint256 amount} where s.user and s.amount access the struct members.
    
31.  Enums: They can be used to create custom types with a finite set of constant values to improve readability. They need a minimum of one member and can have a maximum of 256. They can be explicitly converted to/from integers. The options are represented by unsigned integer values starting from 0. The default value is the first member.
    
32.  Constructor: Contracts can be created “from outside” via Ethereum transactions or from within Solidity contracts. When a contract is created, its constructor (a function declared with the _constructor_ keyword) is executed once. A constructor is optional and only one constructor is allowed. After the constructor has executed, the final code of the contract is stored on the blockchain. This code includes all public and external functions and all functions that are reachable from there through function calls. The deployed code does not include the constructor code or internal functions only called from the constructor.
    
33.  Receive Function: A contract can have at most one _receive_ function, declared using _receive() external payable { ... }_ without the function keyword. This function cannot have arguments, cannot return anything and must have external visibility and payable state mutability.
    
    1.  The receive function is executed on a call to the contract with empty calldata. This is the function that is executed on plain Ether transfers via _.send()_ or _.transfer()_.
        
    2.  In the worst case, the receive function can only rely on 2300 gas being available (for example when send or transfer is used), leaving little room to perform other operations except basic logging
        
    3.  A contract without a receive Ether function can receive Ether as a recipient of a _coinbase transaction_ (aka miner block reward) or as a destination of a _selfdestruct_. A contract cannot react to such Ether transfers and thus also cannot reject them. This means that _address(this).balance_ can be higher than the sum of some manual accounting implemented in a contract (i.e. having a counter updated in the receive Ether function).
        
34.  Fallback Function: A contract can have at most one fallback function, declared using either _fallback () external [payable]_ or _fallback (bytes calldata _input) external [payable] returns (bytes memory _output),_ both without the function keyword. This function must have external visibility.
    
    1.  The fallback function is executed on a call to the contract if none of the other functions match the given function signature, or if no data was supplied at all and there is no receive Ether function. The fallback function always receives data, but in order to also receive Ether it must be marked payable.
        
    2.  In the worst case, if a payable fallback function is also used in place of a receive function, it can only rely on 2300 gas being available
        
35.  Solidity is a statically-typed language, which means that the type of each variable (state and local) needs to be specified in code at compile-time. This is unlike dynamically-typed languages where types are required only with runtime values. Statically-typed languages perform compile-time type-checking according to the language rules. Other examples are C, C++, Java, Rust, Go, Scala.
    
36.  Solidity has two categories of types: Value Types and Reference Types. Value Types are called so because variables of these types will always be passed by value, i.e. they are always copied when they are used as function arguments or in assignments. In contrast, Reference Types can be modified through multiple different names i.e. references to the same underlying variable.
    
37.  Value Types: Types that are passed by value, i.e. they are always copied when they are used as function arguments or in assignments — Booleans, Integers, Fixed Point Numbers, Address, Contract, Fixed-size Byte Arrays (bytes1, bytes2, …, bytes32), Literals (Address, Rational, Integer, String, Unicode, Hexadecimal), Enums, Functions.
    
38.  Reference Types: Types that can be modified through multiple different names. Arrays (including Dynamically-sized bytes array _bytes_ and _string),_ Structs, Mappings.
    
39.  Default Values: A variable which is declared will have an initial default value whose byte-representation is all zeros. The “default values” of variables are the typical “zero-state” of whatever the type is. For example, the default value for a bool is false. The default value for the uint or int types is 0. For statically-sized arrays and bytes1 to bytes32, each individual element will be initialized to the default value corresponding to its type. For dynamically-sized arrays, bytes and string, the default value is an empty array or string. For the enum type, the default value is its first member.
    
40.  Scoping: Scoping in Solidity follows the widespread scoping rules of C99
    
    1.  Variables are visible from the point right after their declaration until the end of the smallest { }-block that contains the declaration. As an exception to this rule, variables declared in the initialization part of a for-loop are only visible until the end of the for-loop.
        
    2.  Variables that are parameter-like (function parameters, modifier parameters, catch parameters, …) are visible inside the code block that follows - the body of the function/modifier for a function and modifier parameter and the catch block for a catch parameter.
        
    3.  Variables and other items declared outside of a code block, for example functions, contracts, user-defined types, etc., are visible even before they were declared. This means you can use state variables before they are declared and call functions recursively.
        
41.  Boolean: bool Keyword and the possible values are constants true and false. 
    
    1.  Operators are ! (logical negation) && (logical conjunction, “and”) || (logical disjunction, “or”)== (equality) and != (inequality). 
        
    2.  The operators || and && apply the common short-circuiting rules. This means that in the expression f(x) || g(y), if f(x) evaluates to true, g(y) will not be evaluated even if it may have side-effects.
        
42.  Integers: int / uint: Signed and unsigned integers of various sizes. Keywords uint8 to uint256 in steps of 8 (unsigned of 8 up to 256 bits) and int8 to int256. uint and int are aliases for uint256 and int256, respectively. Operators are: 
    
    1.  Comparisons: <=, <, ==, !=, >=, > (evaluate to bool)
        
    2.  Bit operators: &, |, ^ (bitwise exclusive or), ~ (bitwise negation)
        
    3.  Shift operators: << (left shift), >> (right shift)
        
    4.  Arithmetic operators: +, -, unary - (only for signed integers), *, /, % (modulo), ** (exponentiation)
        
43.  Integers in Solidity are restricted to a certain range. For example, with uint32, this is 0 up to 2**32 - 1. There are two modes in which arithmetic is performed on these types: The “wrapping” or “unchecked” mode and the “checked” mode. By default, arithmetic is always “checked”, which means that if the result of an operation falls outside the value range of the type, the call is reverted through a failing assertion. You can switch to “unchecked” mode using unchecked { ... }. This was introduced in compiler version 0.8.0.
    
44.  Fixed Point Numbers: Fixed point numbers using keywords _fixed_ / _ufixed_ are not fully supported by Solidity yet. They can be declared, but cannot be assigned to or from. There are fixed-point libraries that are widely used for this such as DSMath, PRBMath, ABDKMath64x64 etc.
    
45.  Address Type: The address type comes in two types: (1) _address_: Holds a 20 byte value (size of an Ethereum address) (2) _address payable_: Same as _address_, but with the additional members _transfer_ and _send_. _address payable_ is an address you can send Ether to, while a plain _address_ cannot be sent Ether. 
    
    1.  Operators are <=, <, ==, !=, >= and >
        
    2.  Conversions: Implicit conversions from _address payable_ to _address_ are allowed, whereas conversions from _address_ to _address payable_ must be explicit via _payable(<address>)_. Explicit conversions to and from _address_ are allowed for _uint160_, integer literals, _bytes20_ and contract types. 
        
    3.  Only expressions of type _address_ and contract-type can be converted to the type _address payable_ via the explicit conversion _payable(...)_. For contract-type, this conversion is only allowed if the contract can receive Ether, i.e., the contract either has a receive or a payable fallback function.
        
46.  Members of Address Type:
    
    1.  _<address>.balance (uint256)_: balance of the Address in Wei
        
    2.  _<address>.code (bytes memory)_: code at the Address (can be empty)
        
    3.  _<address>.codehash (bytes32)_: the codehash of the Address
        
    4.  _<address payable>.transfer(uint256 amount)_: send given amount of Wei to Address, reverts on failure, forwards 2300 gas stipend, not adjustable
        
    5.  _<address payable>.send(uint256 amount)_ returns (bool): send given amount of Wei to Address, returns false on failure, forwards 2300 gas stipend, not adjustable
        
    6.  _<address>.call(bytes memory)_ returns (bool, bytes memory): issue low-level CALL with the given payload, returns success condition and return data, forwards all available gas, adjustable
        
    7.  _<address>.delegatecall(bytes memory)_ returns (bool, bytes memory): issue low-level DELEGATECALL with the given payload, returns success condition and return data, forwards all available gas, adjustable
        
    8.  _<address>.staticcall(bytes memory)_ returns (bool, bytes memory): issue low-level STATICCALL with the given payload, returns success condition and return data, forwards all available gas, adjustable
        
47.  Transfer: The _transfer_ function fails if the balance of the current contract is not large enough or if the Ether transfer is rejected by the receiving account. The _transfer_ function reverts on failure. The code in _receive_ function or if not present then in _fallback_ function is executed with the transfer call. If that execution runs out of gas or fails in any way, the Ether transfer will be reverted and the current contract will stop with an exception.
    
48.  Send: The _send_ function is the low-level counterpart of _transfer_. If the execution fails then _send_ only returns false and does not revert unlike _transfer._ So the return value of _send_ must be checked by the caller.
    
49.  Call/Delegatecall/Staticcall: In order to interface with contracts that do not adhere to the ABI, or to get more direct control over the encoding, the functions _call_, _delegatecall_ and _staticcall_ are provided. They all take a single _bytes_ memory parameter and return the success condition (as a bool) and the returned data _(bytes memory)_. The functions _abi.encode_, _abi.encodePacked_, _abi.encodeWithSelector_ and _abi.encodeWithSignature_ can be used to encode structured data.
    
    1.  _gas_ and _value_ modifiers can be used with these functions (_delegatecall_ doesn’t support _value_) to specify the amount of gas and Ether value passed to the callee.
        
    2.  With _delegatecall_, only the code of the given address is used but all other aspects (storage, balance, msg.sender etc.) are taken from the current contract. The purpose of _delegatecall_ is to use library/logic code which is stored in callee contract but operate on the state of the caller contract
        
    3.  With staticcall, the execution will revert if the called function modifies the state in any way
        
50.  Contract Type: Every contract defines its own type. Contracts can be explicitly converted to and from the _address_ type. Contract types do not support any operators. The members of contract types are the _external_ functions of the contract including any state variables marked as public.
    
51.  Fixed-size Byte Arrays: The value types _bytes1, bytes2, bytes3, …, bytes32_ hold a sequence of bytes from one to up to 32. The type _byte[]_ is an array of bytes, but due to padding rules, it wastes 31 bytes of space for each element (except in storage). It is better to use the _bytes_ type instead.
    
52.  Literals: They can be of 5 types: 
    
    1.  Address Literals: Hexadecimal literals that pass the address checksum test are of address type. Hexadecimal literals that are between 39 and 41 digits long and do not pass the checksum test produce an error. The mixed-case address checksum format is defined in [EIP-55](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-55.md).
        
    2.  Rational and Integer Literals: Integer literals are formed from a sequence of numbers in the range 0-9. Decimal fraction literals are formed by a . with at least one number on one side. Scientific notation is also supported, where the base can have fractions and the exponent cannot. Underscores can be used to separate the digits of a numeric literal to aid readability and are semantically ignored.
        
    3.  String Literals: String literals are written with either double or single-quotes ("foo" or ‘bar’). They can only contain printable ASCII characters and a set of escape characters
        
    4.  Unicode Literals: Unicode literals prefixed with the keyword _unicode_ can contain any valid UTF-8 sequence. They also support the very same escape sequences as regular string literals.
        
    5.  Hexadecimal Literals: Hexadecimal literals are hexadecimal digits prefixed with the keyword hex and are enclosed in double or single-quotes e.g. _hex”001122FF”_, _hex'0011_22_FF'_.
        
53.  Enums: Enums are one way to create a user-defined type in Solidity. They require at least one member and its default value when declared is the first member. They cannot have more than 256 members.
    
54.  Function Types: Function types are the types of functions. Variables of function type can be assigned from functions and function parameters of function type can be used to pass functions to and return functions from function calls.  They come in two flavours - internal and external functions. Internal functions can only be called inside the current contract. External functions consist of an address and a function signature and they can be passed via and returned from external function calls.
    
55.  Reference Types & Data Location: Every reference type has an additional annotation — the data location where it is stored. There are three data locations: _memory_, _storage_ and _calldata_. 
    
    1.  _memory_: whose lifetime is limited to an external function call
        
    2.  _storage_: whose lifetime is limited to the lifetime of a contract and the location where the state variables are stored
        
    3.  _calldata_: which is a non-modifiable, non-persistent area where function arguments are stored and behaves mostly like memory. It is required for parameters of external functions but can also be used for other variables.
        
56.  Data Location & Assignment: Data locations are not only relevant for persistence of data, but also for the semantics of assignments.
    
    1.  Assignments between storage and memory (or from calldata) always create an independent copy.
        
    2.  Assignments from memory to memory only create references. This means that changes to one memory variable are also visible in all other memory variables that refer to the same data.
        
    3.  Assignments from storage to a local storage variable also only assign a reference.
        
    4.  All other assignments to storage always copy. Examples for this case are assignments to state variables or to members of local variables of storage struct type, even if the local variable itself is just a reference.
        
57.  Arrays: Arrays can have a compile-time fixed size, or they can have a dynamic size
    
    1.  The type of an array of fixed size k and element type T is written as T[k], and an array of dynamic size as T[].
        
    2.  Indices are zero-based
        
    3.  Array elements can be of any type, including mapping or struct. 
        
    4.  Accessing an array past its end causes a failing assertion
        
58.  Array members:
    
    1.  _length_: returns number of elements in array
        
    2.  _push()_: appends a zero-initialised element at the end of the array and returns a reference to the element
        
    3.  _push(x)_:  appends a given element at the end of the array and returns nothing
        
    4.  _pop_: removes an element from the end of the array and implicitly calls delete on the removed element
        
59.  Variables of type _bytes_ and _string_ are special arrays
    
    1.  _bytes_ is similar to _byte[]_, but it is packed tightly in calldata and memory
        
    2.  _string_ is equal to _bytes_ but does not allow length or index access
        
    3.  Solidity does not have string manipulation functions, but there are third-party string libraries
        
    4.  Use _bytes_ for arbitrary-length raw byte data and _string_ for arbitrary-length string (UTF-8) data
        
    5.  Use _bytes_ over _byte[]_ because it is cheaper, since _byte[]_ adds 31 padding bytes between the elements
        
    6.  If you can limit the length to a certain number of bytes, always use one of the value types _bytes1_ to _bytes32_ because they are much cheaper
        
60.  Memory Arrays: Memory arrays with dynamic length can be created using the _new_ operator
    
    1.  As opposed to storage arrays, it is not possible to resize memory arrays i.e. the _.push_ member functions are not available
        
    2.  You either have to calculate the required size in advance or create a new memory array and copy every element
        
61.  Array Literals: An array literal is a comma-separated list of one or more expressions, enclosed in square brackets ([…])
    
    1.  It is always a statically-sized memory array whose length is the number of expressions
        
    2.  The base type of the array is the type of the first expression on the list such that all other expressions can be implicitly converted to it. It is a type error if this is not possible.
        
    3.  Fixed size memory arrays cannot be assigned to dynamically-sized memory arrays
        
62.  Gas costs of push and pop: Increasing the length of a storage array by calling push() has constant gas costs because storage is zero-initialised, while decreasing the length by calling pop() has a cost that depends on the “size” of the element being removed. If that element is an array, it can be very costly, because it includes explicitly clearing the removed elements similar to calling delete on them.
    
63.  Array Slices: Array slices are a view on a contiguous portion of an array. They are written as x[start:end], where start and end are expressions resulting in a uint256 type (or implicitly convertible to it). The first element of the slice is x[start] and the last element is x[end - 1]
    
    1.  If start is greater than end or if end is greater than the length of the array, an exception is thrown
        
    2.  Both start and end are optional: start defaults to 0 and end defaults to the length of the array
        
    3.  Array slices do not have any members
        
    4.  They are implicitly convertible to arrays of their underlying type and support index access. Index access is not absolute in the underlying array, but relative to the start of the slice
        
    5.  Array slices do not have a type name which means no variable can have an array slices as type and they only exist in intermediate expressions
        
    6.  Array slices are only implemented for calldata arrays.
        
    7.  Array slices are useful to ABI-decode secondary data passed in function parameters
        
64.  Struct Types: Structs help define new aggregate types by combining other value/reference types into one unit. Struct types can be used inside mappings and arrays and they can themselves contain mappings and arrays. It is not possible for a struct to contain a member of its own type
    
65.  Mapping Types: Mappings define key-value pairs and are declared using the syntax _mapping(_KeyType => _ValueType) _VariableName._ 
    
    1.  The __KeyType_ can be any built-in value type, bytes, string, or any contract or enum type. Other user-defined or complex types, such as mappings, structs or array types are not allowed. __ValueType_ can be any type, including mappings, arrays and structs.
        
    2.  Key data is not stored in a mapping, only its keccak256 hash is used to look up the value
        
    3.  They do not have a length or a concept of a key or value being set
        
    4.  They can only have a data location of storage and thus are allowed for state variables, as storage reference types in functions, or as parameters for library functions
        
    5.  They cannot be used as parameters or return parameters of contract functions that are publicly visible. These restrictions are also true for arrays and structs that contain mappings.
        
    6.  You cannot iterate over mappings, i.e. you cannot enumerate their keys. It is possible, though, to implement a data structure on top of them and iterate over that.
        
66.  Operators Involving LValues (i.e. a variable or something that can be assigned to)
    
    1.  a += e is equivalent to a = a + e. The operators -=, *=, /=, %=, |=, &= and ^= are defined accordingly
        
    2.  a++ and a-- are equivalent to a += 1 / a -= 1 but the expression itself still has the previous value of a
        
    3.  In contrast, --a and ++a have the same effect on a but return the value after the change
        
67.  delete
    
    1.  delete a assigns the initial value for the type to a
        
    2.  For integers it is equivalent to a = 0
        
    3.  For arrays, it assigns a dynamic array of length zero or a static array of the same length with all elements set to their initial value
        
    4.  _delete a[x]_ deletes the item at index x of the array and leaves all other elements and the length of the array untouched
        
    5.  For structs, it assigns a struct with all members reset
        
    6.  delete has no effect on mappings. So if you delete a struct, it will reset all members that are not mappings and also recurse into the members unless they are mappings.
        
    7.  For mappings, individual keys and what they map to can be deleted: If a is a mapping, then delete a[x] will delete the value stored at x
        
68.  Implicit Conversions: An implicit type conversion is automatically applied by the compiler in some cases during assignments, when passing arguments to functions and when applying operators
    
    1.  implicit conversion between value-types is possible if it makes sense semantically and no information is lost
        
    2.  For example, uint8 is convertible to uint16 and int128 to int256, but int8 is not convertible to uint256, because uint256 cannot hold values such as -1
        
69.  Explicit Conversions: If the compiler does not allow implicit conversion but you are confident a conversion will work, an explicit type conversion is sometimes possible. This may result in unexpected behaviour and allows you to bypass some security features of the compiler e.g. int to uint
    
    1.  If an integer is explicitly converted to a smaller type, higher-order bits are cut off
        
    2.  If an integer is explicitly converted to a larger type, it is padded on the left (i.e., at the higher order end)
        
    3.  Fixed-size bytes types while explicitly converting to a smaller type and will cut off the bytes to the right
        
    4.  Fixed-size bytes types while explicitly converting to a larger type and will pad bytes to the right.
        
70.  Conversions between Literals and Elementary Types
    
    1.  Decimal and hexadecimal number literals can be implicitly converted to any integer type that is large enough to represent it without truncation
        
    2.  Decimal number literals cannot be implicitly converted to fixed-size byte arrays
        
    3.  Hexadecimal number literals can be, but only if the number of hex digits exactly fits the size of the bytes type. As an exception both decimal and hexadecimal literals which have a value of zero can be converted to any fixed-size bytes type
        
    4.  String literals and hex string literals can be implicitly converted to fixed-size byte arrays, if their number of characters matches the size of the bytes type
        
71.  A literal number can take a suffix of wei, gwei (1e9) or ether (1e18) to specify a sub-denomination of Ether
    
72.  Suffixes like seconds, minutes, hours, days and weeks after literal numbers can be used to specify units of time where seconds are the base unit where 1 == 1 seconds,1 minutes == 60 seconds, 1 hours == 60 minutes, 1 days == 24 hours and 1 weeks == 7 days
    
    1.  Take care if you perform calendar calculations using these units, because not every year equals 365 days and not even every day has 24 hours because of leap seconds
        
    2.  These suffixes cannot be applied directly to variables but can be applied by multiplication
        
73.  Block and Transaction Properties:
    
    1.  _blockhash(uint blockNumber)_ returns _(bytes32)_: hash of the given block - only works for 256 most recent, excluding current, blocks
        
    2.  _block.chainid (uint)_: current chain id
        
    3.  _block.coinbase (address payable)_: current block miner’s address
        
    4.  _block.difficulty (uint)_: current block difficulty
        
    5.  _block.gaslimit (uint)_: current block gaslimit
        
    6.  _block.number (uint)_: current block number
        
    7.  _block.timestamp (uint)_: current block timestamp as seconds since unix epoch
        
    8.  _msg.data (bytes calldata)_: complete calldata
        
    9.  _msg.sender (address)_: sender of the message (current call)
        
    10.  _msg.sig (bytes4)_: first four bytes of the calldata (i.e. function identifier)
        
    11.  _msg.value (uint)_: number of wei sent with the message
        
    12.  _tx.gasprice (uint)_: gas price of the transaction
        
    13.  _gasleft() returns (uint256)_: remaining gas
        
    14.  _tx.origin (address)_: sender of the transaction (full call chain)
        
74.  The values of all members of msg, including msg.sender and msg.value can change for every external function call. This includes calls to library functions.
    
75.  Do not rely on block.timestamp or blockhash as a source of randomness. Both the timestamp and the block hash can be influenced by miners to some degree. The current block timestamp must be strictly larger than the timestamp of the last block, but the only guarantee is that it will be somewhere between the timestamps of two consecutive blocks in the canonical chain. 
    
76.  The block hashes are not available for all blocks for scalability reasons. You can only access the hashes of the most recent 256 blocks, all other values will be zero.
    
77.  ABI Encoding and Decoding Functions:
    
    1.  _abi.decode(bytes memory encodedData, (...)) returns (...)_: ABI-decodes the given data, while the types are given in parentheses as second argument.
        
    2.  _abi.encode(...)_ returns (bytes memory): ABI-encodes the given arguments
        
    3.  _abi.encodePacked(...)_ returns (bytes memory): Performs packed encoding of the given arguments. Note that packed encoding can be ambiguous!
        
    4.  _abi.encodeWithSelector(bytes4 selector, ...) returns (bytes memory)_: ABI-encodes the given arguments starting from the second and prepends the given four-byte selector
        
    5.  _abi.encodeWithSignature(string memory signature, ...) returns (bytes memory)_: Equivalent to abi.encodeWithSelector(bytes4(keccak256(bytes(signature))), …)
        
78.  Error Handling:
    
    1.  _assert(bool condition)_: causes a Panic error and thus state change reversion if the condition is not met - to be used for internal errors.
        
    2.  _require(bool condition)_: reverts if the condition is not met - to be used for errors in inputs or external components.
        
    3.  _require(bool condition, string memory message)_: reverts if the condition is not met - to be used for errors in inputs or external components. Also provides an error message.
        
    4.  _revert()_: abort execution and revert state changes
        
    5.  _revert(string memory reason)_: abort execution and revert state changes, providing an explanatory string
        
79.  Mathematical and Cryptographic Functions:
    
    1.  _addmod(uint x, uint y, uint k) returns (uint)_: compute (x + y) % k where the addition is performed with arbitrary precision and does not wrap around at 2**256. Assert that k != 0 starting from version 0.5.0.
        
    2.  _mulmod(uint x, uint y, uint k) returns (uint)_: compute (x * y) % k where the multiplication is performed with arbitrary precision and does not wrap around at 2**256. Assert that k != 0 starting from version 0.5.0.
        
    3.  _keccak256(bytes memory) returns (bytes32)_: compute the Keccak-256 hash of the input
        
    4.  _sha256(bytes memory) returns (bytes32)_: compute the SHA-256 hash of the input
        
    5.  _ripemd160(bytes memory) returns (bytes20)_: compute RIPEMD-160 hash of the input
        
    6.  _ecrecover(bytes32 hash, uint8 v, bytes32 r, bytes32 s) returns (address):_ recover the address associated with the public key from elliptic curve signature or return zero on error. The function parameters correspond to ECDSA values of the signature: _r = first 32 bytes of signature, s = second 32 bytes of signature, v = final 1 byte of signature. ecrecover_ returns an address, and not an address payable.
        
80.  If you use _ecrecover_, be aware that a valid signature can be turned into a different valid signature without requiring knowledge of the corresponding private key. This is usually not a problem unless you require signatures to be unique or use them to identify items. OpenZeppelin has a ECDSA helper library that you can use as a wrapper for _ecrecover_ without this issue.
    
81.  Contract Related:
    
    1.  _this_ (current contract’s type): the current contract, explicitly convertible to Address
        
    2.  _selfdestruct(address payable recipient)_: Destroy the current contract, sending its funds to the given Address and end execution. 
        
82.  _selfdestruct_ has some peculiarities: the receiving contract’s receive function is not executed and the contract is only really destroyed at the end of the transaction and revert’s might “undo” the destruction.
    
83.  Type Information: The expression type(X) can be used to retrieve information about the type X, where X can be either a contract or an integer type. For a contract type C, the following type information is available:
    
    1.  _type(C).name_: The name of the contract.
        
    2.  _type(C).creationCode_: Memory byte array that contains the creation bytecode of the contract. This can be used in inline assembly to build custom creation routines, especially by using the create2 opcode. This property cannot be accessed in the contract itself or any derived contract. It causes the bytecode to be included in the bytecode of the call site and thus circular references like that are not possible.
        
    3.  _type(C).runtimeCode_: Memory byte array that contains the runtime bytecode of the contract. This is the code that is usually deployed by the constructor of C. If C has a constructor that uses inline assembly, this might be different from the actually deployed bytecode. Also note that libraries modify their runtime bytecode at time of deployment to guard against regular calls. The same restrictions as with ._creationCode_ also apply for this property.
        
    4.  For an interface type I, the following type information is available: _type(I).interfaceId_: A bytes4 value containing the EIP-165 interface identifier of the given interface I. This identifier is defined as the XOR of all function selectors defined within the interface itself - excluding all inherited functions.
        
84.  For an integer type T, , the following type information is available:
    
    1.  _type(T).min_: The smallest value representable by type T.
        
    2.  _type(T).max_: The largest value representable by type T.
        
85.  Control Structures: Solidity has _if, else, while, do, for, break, continue, return_, with the usual semantics known from C or JavaScript
    
    1.  Parentheses can not be omitted for conditionals, but curly braces can be omitted around single-statement bodies
        
    2.  Note that there is no type conversion from non-boolean to boolean types as there is in C and JavaScript, so _if (1) { ... }_ is not valid Solidity.
        
86.  Exceptions: Solidity uses state-reverting exceptions to handle errors. Such an exception undoes all changes made to the state in the current call (and all its sub-calls) and flags an error to the caller
    
    1.  When exceptions happen in a sub-call, they “bubble up” (i.e., exceptions are rethrown) automatically. Exceptions to this rule are send and the low-level functions _call_, _delegatecall_ and _staticcall_: they return false as their first return value in case of an exception instead of “bubbling up”.
        
    2.  Exceptions in external calls can be caught with the _try/catch_ statement
        
    3.  Exceptions can contain data that is passed back to the caller. This data consists of a 4-byte selector and subsequent ABI-encoded data. The selector is computed in the same way as a function selector, i.e., the first four bytes of the keccak256-hash of a function signature - in this case an error signature.
        
    4.  Solidity supports two error signatures: Error(string) and Panic(uint256). The first (“error”) is used for “regular” error conditions while the second (“panic”) is used for errors that should not be present in bug-free code.
        
87.  The low-level functions _call_, _delegatecall_ and _staticcall_ return true as their first return value if the account called is non-existent, as part of the design of the EVM. Account existence must be checked prior to calling if needed.
    
88.  The _assert_ function creates an error of type Panic(uint256). Assert should only be used to test for internal errors, and to check invariants. Properly functioning code should never create a Panic, not even on invalid external input. 
    
89.  A Panic exception is generated in the following situations. The error code supplied with the error data indicates the kind of panic:
    
    1.  0x01: If you call assert with an argument that evaluates to false.
        
    2.  0x11: If an arithmetic operation results in underflow or overflow outside of an unchecked { ... } block.
        
    3.  0x12; If you divide or modulo by zero (e.g. 5 / 0 or 23 % 0).
        
    4.  0x21: If you convert a value that is too big or negative into an enum type.
        
    5.  0x22: If you access a storage byte array that is incorrectly encoded.
        
    6.  0x31: If you call .pop() on an empty array.
        
    7.  0x32: If you access an array, bytesN or an array slice at an out-of-bounds or negative index (i.e. x[i] where i >= x.length or i < 0).
        
    8.  0x41: If you allocate too much memory or create an array that is too large.
        
    9.  0x51: If you call a zero-initialized variable of internal function type.
        
90.  The _require_ function either creates an error of type Error(string) or an error without any error data and it should be used to ensure valid conditions that cannot be detected until execution time. This includes conditions on inputs or return values from calls to external contracts. You can optionally provide a message string for _require_, but not for _assert_.
    
91.  A _Error(string)_ exception (or an exception without data) is generated in the following situations:
    
    1.  Calling _require_ with an argument that evaluates to false.
        
    2.  If you perform an external function call targeting a contract that contains no code
        
    3.  If your contract receives Ether via a public function without payable modifier (including the constructor and the fallback function)
        
    4.  If your contract receives Ether via a public getter function
        
92.  _revert_: A direct revert can be triggered using the revert statement and the revert function. The revert statement takes a custom error as a direct argument without parentheses: revert CustomError(arg1, arg2). The _revert()_ function is another way to trigger exceptions from within other code blocks to flag an error and revert the current call. The function takes an optional string message containing details about the error that is passed back to the caller and it will create an Error(string) exception. Using a custom error instance will usually be much cheaper than a string description, because you can use the name of the error to describe it, which is encoded in only four bytes. A longer description can be supplied via NatSpec which does not incur any costs.
    
93.  _try/catch_: The _try_ keyword has to be followed by an expression representing an external function call or a contract creation (_new ContractName()_). Errors inside the expression are not caught (for example if it is a complex expression that also involves internal function calls), only a revert happening inside the external call itself. The returns part (which is optional) that follows declares return variables matching the types returned by the external call. In case there was no error, these variables are assigned and the contract’s execution continues inside the first success block. If the end of the success block is reached, execution continues after the _catch_ blocks.
    
94.  Solidity supports different kinds of catch blocks depending on the type of error:
    
    1.  _catch Error(string memory reason) { ... }_: This catch clause is executed if the error was caused by revert("reasonString") or require(false, "reasonString") (or an internal error that causes such an exception).
        
    2.  _catch Panic(uint errorCode) { ... }_: If the error was caused by a panic, i.e. by a failing assert, division by zero, invalid array access, arithmetic overflow and others, this catch clause will be run.
        
    3.  _catch (bytes memory lowLevelData) { ... }_: This clause is executed if the error signature does not match any other clause, if there was an error while decoding the error message, or if no error data was provided with the exception. The declared variable provides access to the low-level error data in that case.
        
    4.  _catch { ... }_: If you are not interested in the error data, you can just use _catch { ... }_ (even as the only catch clause) instead of the previous clause.
        
95.  If execution reaches a catch-block, then the state-changing effects of the external call have been reverted. If execution reaches the success block, the effects were not reverted. If the effects have been reverted, then execution either continues in a catch block or the execution of the try/catch statement itself reverts (for example due to decoding failures as noted above or due to not providing a low-level catch clause). 
    
96.  The reason behind a failed call can be manifold. Do not assume that the error message is coming directly from the called contract: The error might have happened deeper down in the call chain and the called contract just forwarded it. Also, it could be due to an out-of-gas situation and not a deliberate error condition: The caller always retains 63/64th of the gas in a call and thus even if the called contract goes out of gas, the caller still has some gas left
    
97.  Programming style: coding conventions for writing solidity code. Style is about consistency. Consistency with style is important. Consistency within a project is more important. Consistency within one module or function is most important. Two main categories: 1) Layout 2) Naming Conventions. Programming style affects readability and maintainability, both of which affect security.
    
98.  Code Layout:
    
    1.  Indentation: Use 4 spaces per indentation level
        
    2.  Tabs or Spaces: Spaces are the preferred indentation method. Mixing tabs and spaces should be avoided.
        
    3.  Blank Lines: Surround top level declarations in solidity source with two blank lines.
        
    4.  Maximum Line Length: Keeping lines to a maximum of 79 (or 99) characters helps readers easily parse the code.
        
    5.  Wrapped lines should conform to the following guidelines: The first argument should not be attached to the opening parenthesis. One, and only one, indent should be used. Each argument should fall on its own line. The terminating element, );, should be placed on the final line by itself.
        
    6.  Source File Encoding: UTF-8 or ASCII encoding is preferred.
        
    7.  Imports: Import statements should always be placed at the top of the file.
        
    8.  Order of Functions: Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. Functions should be grouped according to their visibility and ordered: constructor, receive function (if exists), fallback function (if exists), external, public, internal, private. Within a grouping, place the view and pure functions last.
        
99.  More Code Layout:
    
    1.  Whitespace in Expressions: Avoid extraneous whitespace in the following situations —  Immediately inside parenthesis, brackets or braces, with the exception of single line function declarations.
        
    2.  Control Structures:  The braces denoting the body of a contract, library, functions and structs should: open on the same line as the declaration, close on their own line at the same indentation level as the beginning of the declaration. The opening brace should be preceded by a single space.
        
    3.  Function Declaration: For short function declarations, it is recommended for the opening brace of the function body to be kept on the same line as the function declaration. The closing brace should be at the same indentation level as the function declaration. The opening brace should be preceded by a single space.
        
    4.  Mappings: In variable declarations, do not separate the keyword mapping from its type by a space. Do not separate any nested mapping keyword from its type by whitespace.
        
    5.  Variable Declarations: Declarations of array variables should not have a space between the type and the brackets.
        
    6.  Strings should be quoted with double-quotes instead of single-quotes.
        
    7.  Operators: Surround operators with a single space on either side. Operators with a higher priority than others can exclude surrounding whitespace in order to denote precedence.This is meant to allow for improved readability for complex statements. You should always use the same amount of whitespace on either side of an operator
        
    8.  Layout contract elements in the following order: Pragma statements, Import statements, Interfaces, Libraries, Contracts. Inside each contract, library or interface, use the following order: Type declarations, State variables, Events, Functions
        
100.  Naming Convention:
    
    1.  Types: lowercase, lower_case_with_underscores, UPPERCASE, UPPER_CASE_WITH_UNDERSCORES, CapitalizedWords, mixedCase, Capitalized_Words_With_Underscores
        
    2.  Names to Avoid: l - Lowercase letter el, O - Uppercase letter oh, I - Uppercase letter eye. Never use any of these for single letter variable names. They are often indistinguishable from the numerals one and zero.
        
    3.  Contracts and libraries should be named using the CapWords style. Contract and library names should also match their filenames. If a contract file includes multiple contracts and/or libraries, then the filename should match the core contract. This is not recommended however if it can be avoided. Examples: SimpleToken, SmartBank, CertificateHashRepository, Player, Congress, Owned.
        
    4.  Structs should be named using the CapWords style. Examples: MyCoin, Position, PositionXY.
        
    5.  Events should be named using the CapWords style. Examples: Deposit, Transfer, Approval, BeforeTransfer, AfterTransfer.
        
    6.  Functions should use mixedCase. Examples: getBalance, transfer, verifyOwner, addMember, changeOwner.
        
101.  More Naming Convention:
    
    1.  Function arguments should use mixedCase. Examples: initialSupply, account, recipientAddress, senderAddress, newOwner.
        
    2.  Local and state variable names should use mixedCase. Examples: totalSupply, remainingSupply, balancesOf, creatorAddress, isPreSale, tokenExchangeRate.
        
    3.  Constants should be named with all capital letters with underscores separating words. Examples: MAX_BLOCKS, TOKEN_NAME, TOKEN_TICKER, CONTRACT_VERSION.
        
    4.  Modifier names should use mixedCase. Examples: onlyBy, onlyAfter, onlyDuringThePreSale.
        
    5.  Enums, in the style of simple type declarations, should be named using the CapWords style. Examples: TokenGroup, Frame, HashStyle, CharacterLocation.
        
    6.  Avoiding Naming Collisions: single_trailing_underscore_. This convention is suggested when the desired name collides with that of a built-in or otherwise reserved name.

102.  Solidity supports multiple inheritance including polymorphism:
    
    1.  Polymorphism means that a function call (internal and external) always executes the function of the same name (and parameter types) in the most derived contract in the inheritance hierarchy
        
    2.  When a contract inherits from other contracts, only a single contract is created on the blockchain, and the code from all the base contracts is compiled into the created contract.
        
    3.  Function Overriding: Base functions can be overridden by inheriting contracts to change their behavior if they are marked as _virtual_. The overriding function must then use the _override_ keyword in the function header. 
        
    4.  Languages that allow multiple inheritance have to deal with several problems. One is the Diamond Problem. Solidity is similar to Python in that it uses “C3 Linearization” to force a specific order in the directed acyclic graph (DAG) of base classes. So when a function is called that is defined multiple times in different contracts, the given bases are searched from right to left (left to right in Python) in a depth-first manner, stopping at the first match.
        
103.  Contract Types:
    
    1.  Abstract Contracts: Contracts need to be marked as abstract when at least one of their functions is not implemented. They use the _abstract_ keyword.
        
    2.  Interfaces: They cannot have any functions implemented. There are further restrictions: 1) They cannot inherit from other contracts, but they can inherit from other interfaces 2) All declared functions must be external 3) They cannot declare a constructor 4) They cannot declare state variables. They use the _interface_ keyword.
        
    3.  Libraries: They are deployed only once at a specific address and their code is reused using the DELEGATECALL opcode. This means that if library functions are called, their code is executed in the context of the calling contract. They use the _library_ keyword.
        
104.  Using For: The directive using A for B; can be used to attach library functions (from the library A) to any type (B) in the context of a contract. These functions will receive the object they are called on as their first parameter.
    
    1.  The using A for B; directive is active only within the current contract, including within all of its functions, and has no effect outside of the contract in which it is used. 
        
    2.  The directive may only be used inside a contract, not inside any of its functions.
        
105.  Base Class Functions: It is possible to call functions further up in the inheritance hierarchy internally by explicitly specifying the contract using ContractName.functionName() or using super.functionName() if you want to call the function one level higher up in the flattened inheritance hierarchy 
    
106.  State Variable Shadowing: This is considered as an error. A derived contract can only declare a state variable x, if there is no visible state variable with the same name in any of its bases.
    
107.  Function Overriding Changes: The overriding function may only change the visibility of the overridden function from external to public. The mutability may be changed to a more strict one following the order: nonpayable can be overridden by view and pure. view can be overridden by pure. payable is an exception and cannot be changed to any other mutability.
    
108.  Virtual Functions: Functions without implementation have to be marked virtual outside of interfaces. In interfaces, all functions are automatically considered virtual. Functions with private visibility cannot be virtual.
    
109.  Public State Variable Override: Public state variables can override external functions if the parameter and return types of the function matches the getter function of the variable. While public state variables can override external functions, they themselves cannot be overridden.
    
110.  Modifier Overriding: Function modifiers can override each other. This works in the same way as function overriding (except that there is no overloading for modifiers). The virtual keyword must be used on the overridden modifier and the override keyword must be used in the overriding modifier
    
111.  Base Constructors: The constructors of all the base contracts will be called following the linearization rules. If the base constructors have arguments, derived contracts need to specify all of them either in the inheritance list or in the derived constructor.
    
112.  Name Collision Error: It is an error when any of the following pairs in a contract have the same name due to inheritance: 1) a function and a modifier 2) a function and an event 3) an event and a modifier
    
113.  Library Restrictions: In comparison to contracts, libraries are restricted in the following ways:
    
    1.  they cannot have state variables
        
    2.  they cannot inherit nor be inherited
        
    3.  they cannot receive Ether
        
    4.  they cannot be destroyed
        
    5.  it can only access state variables of the calling contract if they are explicitly supplied (it would have no way to name them, otherwise)
        
    6.  Library functions can only be called directly (i.e. without the use of DELEGATECALL) if they do not modify the state (i.e. if they are view or pure functions), because libraries are assumed to be stateless 
        
114.  EVM Storage: Storage is a key-value store that maps 256-bit words to 256-bit words and is accessed with EVM’s SSTORE/SLOAD instructions. All locations in storage are initialized as zero.
    
115.  Storage Layout: State variables of contracts are stored in storage in a compact way such that multiple values sometimes use the same storage slot. Except for dynamically-sized arrays and mappings, data is stored contiguously item after item starting with the first state variable, which is stored in slot 0
    
116.  Storage Layout Packing: For each state variable, a size in bytes is determined according to its type. Multiple, contiguous items that need less than 32 bytes are packed into a single storage slot if possible, according to the following rules:
    
    1.  The first item in a storage slot is stored lower-order aligned
        
    2.  Value types use only as many bytes as are necessary to store them
        
    3.  If a value type does not fit the remaining part of a storage slot, it is stored in the next storage slot
        
117.  Storage Layout & Structs/Arrays: 
    
    1.  Structs and array data always start a new slot and their items are packed tightly according to these rules
        
    2.  Items following struct or array data always start a new storage slot
        
    3.  The elements of structs and arrays are stored after each other, just as if they were given as individual values. 
        
118.  Storage Layout & Inheritance: For contracts that use inheritance, the ordering of state variables is determined by the C3-linearized order of contracts starting with the most base-ward contract. If allowed by the above rules, state variables from different contracts do share the same storage slot.
    
119.  Storage Layout & Types: It might be beneficial to use reduced-size types if you are dealing with storage values because the compiler will pack multiple elements into one storage slot, and thus, combine multiple reads or writes into a single operation.
    
    1.  If you are not reading or writing all the values in a slot at the same time, this can have the opposite effect, though: When one value is written to a multi-value storage slot, the storage slot has to be read first and then combined with the new value such that other data in the same slot is not destroyed.
        
120.  Storage Layout & Ordering: Ordering of storage variables and struct members affects how they can be packed tightly. For example, declaring your storage variables in the order of uint128, uint128, uint256 instead of uint128, uint256, uint128, as the former will only take up two slots of storage whereas the latter will take up three.
    
121.  Storage Layout for Mappings & Dynamically-sized Arrays: Due to their unpredictable size, mappings and dynamically-sized array types cannot be stored “in between” the state variables preceding and following them. Instead, they are considered to occupy only 32 bytes with regards to the rules above and the elements they contain are stored starting at a different storage slot that is computed using a Keccak-256 hash.
    
122.  Storage Layout for  Dynamic Arrays: If the storage location of the array ends up being a slot p after applying the storage layout rules, this slot stores the number of elements in the array (byte arrays and strings are an exception). Array data is located starting at keccak256(p) and it is laid out in the same way as statically-sized array data would: One element after the other, potentially sharing storage slots if the elements are not longer than 16 bytes. Dynamic arrays of dynamic arrays apply this rule recursively.
    
123.  Storage Layout for Mappings: For mappings, the slot stays empty, but it is still needed to ensure that even if there are two mappings next to each other, their content ends up at different storage locations. The value corresponding to a mapping key k is located at keccak256(h(k) . p) where . is concatenation and h is a function that is applied to the key depending on its type: 1) for value types, h pads the value to 32 bytes in the same way as when storing the value in memory. 2) for strings and byte arrays, h computes the keccak256 hash of the unpadded data. If the mapping value is a non-value type, the computed slot marks the start of the data. If the value is of struct type, for example, you have to add an offset corresponding to the struct member to reach the member.
    
124.  Storage Layout for bytes and string: bytes and string are encoded identically. In general, the encoding is similar to byte1[], in the sense that there is a slot for the array itself and a data area that is computed using a keccak256 hash of that slot’s position. However, for short values (shorter than 32 bytes) the array elements are stored together with the length in the same slot.
    
    1.  if the data is at most 31 bytes long, the elements are stored in the higher-order bytes (left aligned) and the lowest-order byte stores the value length * 2. For byte arrays that store data which is 32 or more bytes long, the main slot p stores length * 2 + 1 and the data is stored as usual in keccak256(p). This means that you can distinguish a short array from a long array by checking if the lowest bit is set: short (not set) and long (set).
        
125.  EVM Memory: EVM memory is linear and can be addressed at byte level and accessed with MSTORE/MSTORE8/MLOAD instructions. All locations in memory are initialized as zero.
    
126.  Memory Layout: Solidity places new memory objects at the free memory pointer and memory is never freed. The free memory pointer points to 0x80 initially.
    
127.  Reserved Memory: Solidity reserves four 32-byte slots, with specific byte ranges (inclusive of endpoints) being used as follows:
    
    1.  0x00 - 0x3f (64 bytes): scratch space for hashing methods
        
    2.  0x40 - 0x5f (32 bytes): currently allocated memory size (aka. free memory pointer)
        
    3.  0x60 - 0x7f (32 bytes): zero slot (The zero slot is used as initial value for dynamic memory arrays and should never be written to)
        
128.  Memory Layout & Arrays: Elements in memory arrays in Solidity always occupy multiples of 32 bytes (this is even true for byte[], but not for bytes and string). 
    
    1.  Multi-dimensional memory arrays are pointers to memory arrays
        
    2.  The length of a dynamic array is stored at the first slot of the array and followed by the array elements
        
129.  Free Memory Pointer: There is a “free memory pointer” at position 0x40 in memory. If you want to allocate memory, use the memory starting from where this pointer points at and update it. Considering the reserved memory, allocatable memory starts at 0x80, which is the initial value of the free memory pointer.
    
130.  Zeroed Memory: There is no guarantee that the memory has not been used before and thus you cannot assume that its contents are zero bytes. There is no built-in mechanism to release or free allocated memory. 
    
131.  Reserved Keywords: These keywords are reserved in Solidity. They might become part of the syntax in the future: _after, alias, apply, auto, case, copyof, default, define, final, immutable, implements, in, inline, let, macro, match, mutable, null, of, partial, promise, reference, relocatable, sealed, sizeof, static, supports, switch, typedef, typeof, unchecked_
    
132.  Inline Assembly: Inline assembly is a way to access the Ethereum Virtual Machine at a low level. This bypasses several important safety features and checks of Solidity. You should only use it for tasks that need it, and only if you are confident with using it.
    
    1.  The language used for inline assembly in Solidity is called Yul
        
    2.  An inline assembly block is marked by _assembly { ... }_, where the code inside the curly braces is code in the Yul language 
        
133.  Inline Assembly Access to External Variables, Functions and Libraries: 
    
    1.  You can access Solidity variables and other identifiers by using their name.
        
    2.  Local variables of value type are directly usable in inline assembly
        
    3.  Local variables that refer to memory/calldata evaluate to the address of the variable in memory/calldata and not the value itself
        
    4.  For local storage variables or state variables, a single Yul identifier is not sufficient, since they do not necessarily occupy a single full storage slot. Therefore, their “address” is composed of a slot and a byte-offset inside that slot. To retrieve the slot pointed to by the variable x, you use x.slot, and to retrieve the byte-offset you use x.offset. Using x itself will result in an error.
        
    5.  Local Solidity variables are available for assignments
        
    6.  Assignments are possible to assembly-local variables and to function-local variables. Take care that when you assign to variables that point to memory or storage, you will only change the pointer and not the data.
        
    7.  You can assign to the .slot part of a local storage variable pointer. For these (structs, arrays or mappings), the .offset part is always zero. It is not possible to assign to the .slot or .offset part of a state variable, though
        
134.  Yul Syntax: Yul parses comments, literals and identifiers in the same way as Solidity. Inside a code block, the following elements can be used:
    
    1.  literals, i.e. 0x123, 42 or "abc" (strings up to 32 characters)
        
    2.  calls to builtin functions, e.g. add(1, mload(0))
        
    3.  variable declarations, e.g. let x := 7, let x := add(y, 3) or let x (initial value of 0 is assigned)
        
    4.  identifiers (variables), e.g. add(3, x)
        
    5.  assignments, e.g. x := add(y, 3)
        
    6.  blocks where local variables are scoped inside, e.g. { let x := 3 { let y := add(x, 1) } }
        
    7.  if statements, e.g. if lt(a, b) { sstore(0, 1) }
        
    8.  switch statements, e.g. switch mload(0) case 0 { revert() } default { mstore(0, 1) }
        
    9.  for loops, e.g. for { let i := 0} lt(i, 10) { i := add(i, 1) } { mstore(i, 7) }
        
    10.  function definitions, e.g. function f(a, b) -> c { c := add(a, b) }
        
135.  Solidity v0.6.0 Breaking Semantic Changes - changes where existing code changes its behaviour without the compiler notifying you about it:
    
    1.  The resulting type of an exponentiation is the type of the base. It used to be the smallest type that can hold both the type of the base and the type of the exponent, as with symmetric operations. Additionally, signed types are allowed for the base of the exponentiation.
        
136.  Solidity v0.6.0 Explicitness Requirements:
    
    1.  Functions can now only be overridden when they are either marked with the virtual keyword or defined in an interface. Functions without implementation outside an interface have to be marked virtual. When overriding a function or modifier, the new keyword override must be used. When overriding a function or modifier defined in multiple parallel bases, all bases must be listed in parentheses after the keyword like so: override(Base1, Base2).
        
    2.  Member-access to length of arrays is now always read-only, even for storage arrays. It is no longer possible to resize storage arrays by assigning a new value to their length. Use push(), push(value) or pop() instead, or assign a full array, which will of course overwrite the existing content. The reason behind this is to prevent storage collisions of gigantic storage arrays.
        
    3.  The new keyword abstract can be used to mark contracts as abstract. It has to be used if a contract does not implement all its functions. Abstract contracts cannot be created using the new operator, and it is not possible to generate bytecode for them during compilation.
        
    4.  Libraries have to implement all their functions, not only the internal ones.
        
    5.  The names of variables declared in inline assembly may no longer end in _slot or _offset.
        
    6.  Variable declarations in inline assembly may no longer shadow any declaration outside the inline assembly block. If the name contains a dot, its prefix up to the dot may not conflict with any declaration outside the inline assembly block.
        
    7.  State variable shadowing is now disallowed. A derived contract can only declare a state variable x, if there is no visible state variable with the same name in any of its bases.
        
137.  Solidity v0.6.0 Semantic and Syntactic Changes:
    
    1.  Conversions from external function types to address are now disallowed. Instead external function types have a member called address, similar to the existing selector member.
        
    2.  The function push(value) for dynamic storage arrays does not return the new length anymore (it returns nothing).
        
    3.  The unnamed function commonly referred to as “fallback function” was split up into a new fallback function that is defined using the fallback keyword and a receive ether function defined using the receive keyword.
        
    4.  If present, the receive ether function is called whenever the call data is empty (whether or not ether is received). This function is implicitly payable.
        
    5.  The new fallback function is called when no other function matches (if the receive ether function does not exist then this includes calls with empty call data). You can make this function payable or not. If it is not payable then transactions not matching any other function which send value will revert. You should only need to implement the new fallback function if you are following an upgrade or proxy pattern.
        
138.  Solidity v0.6.0 New Features:
    
    1.  The try/catch statement allows you to react on failed external calls.
        
    2.  struct and enum types can be declared at file level.
        
    3.  Array slices can be used for calldata arrays, for example abi.decode(msg.data[4:], (uint, uint)) is a low-level way to decode the function call payload.
        
    4.  Natspec supports multiple return parameters in developer documentation, enforcing the same naming check as @param.
        
    5.  Yul and Inline Assembly have a new statement called leave that exits the current function.
        
    6.  Conversions from address to address payable are now possible via payable(x), where x must be of type address.
        
139.  Solidity v0.7.0 Breaking Semantic Changes - changes where existing code changes its behaviour without the compiler notifying you about it:
    
    1.  Exponentiation and shifts of literals by non-literals (e.g. 1 << x or 2 ** x) will always use either the type uint256 (for non-negative literals) or int256 (for negative literals) to perform the operation. Previously, the operation was performed in the type of the shift amount / the exponent which can be misleading.
        
140.  Solidity v0.7.0 Changes to the Syntax - changes that might cause existing contracts to not compile anymore:
    
    1.  In external function and contract creation calls, Ether and gas is now specified using a new syntax: x.f{gas: 10000, value: 2 ether}(arg1, arg2). The old syntax – x.f.gas(10000).value(2 ether)(arg1, arg2) – will cause an error.
        
    2.  The global variable now is deprecated, block.timestamp should be used instead. The single identifier now is too generic for a global variable and could give the impression that it changes during transaction processing, whereas block.timestamp correctly reflects the fact that it is just a property of the block.
        
    3.  NatSpec comments on variables are only allowed for public state variables and not for local or internal variables
        
    4.  The token gwei is a keyword now (used to specify, e.g. 2 gwei as a number) and cannot be used as an identifier
        
    5.  String literals now can only contain printable ASCII characters and this also includes a variety of escape sequences, such as hexadecimal (\xff) and unicode escapes (\u20ac).
        
    6.  Unicode string literals are supported now to accommodate valid UTF-8 sequences. They are identified with the unicode prefix: unicode"Hello 😃".
        
    7.  State Mutability: The state mutability of functions can now be restricted during inheritance. Functions with default state mutability can be overridden by pure and view functions while view functions can be overridden by pure functions. At the same time, public state variables are considered view and even pure if they are constants.
        
    8.  Disallow . in user-defined function and variable names in inline assembly. It is still valid if you use Solidity in Yul-only mode.
        
    9.  Slot and offset of storage pointer variable x are accessed via x.slot and x.offset instead of x_slot and x_offset.
        
141.  Solidity v0.7.0 Removal of Unused or Unsafe Features
    
    1.  If a struct or array contains a mapping, it can only be used in storage. Previously, mapping members were silently skipped in memory, which is confusing and error-prone.
        
    2.  Assignments to structs or arrays in storage do not work if they contain mappings. Previously, mappings were silently skipped during the copy operation, which is misleading and error-prone.
        
    3.  Visibility (public / external) is not needed for constructors anymore: To prevent a contract from being created, it can be marked abstract. This makes the visibility concept for constructors obsolete.
        
    4.  Type Checker: Disallow virtual for library functions: Since libraries cannot be inherited from, library functions should not be virtual.
        
    5.  Multiple events with the same name and parameter types in the same inheritance hierarchy are disallowed.
        
    6.  using A for B only affects the contract it is mentioned in. Previously, the effect was inherited. Now, you have to repeat the using statement in all derived contracts that make use of the feature.
        
    7.  Shifts by signed types are disallowed. Previously, shifts by negative amounts were allowed, but reverted at runtime.
        
    8.  The finney and szabo denominations are removed. They are rarely used and do not make the actual amount readily visible. Instead, explicit values like 1e20 or the very common gwei can be used.
        
    9.  The keyword var cannot be used anymore. Previously, this keyword would parse but result in a type error and a suggestion about which type to use. Now, it results in a parser error.
        
142.  Solidity v0.8.0 Breaking Semantic Changes - changes where existing code changes its behaviour without the compiler notifying you about it:
    
    1.  Arithmetic operations revert on underflow and overflow. You can use unchecked { ... } to use the previous wrapping behaviour. Checks for overflow are very common, so they are the default to increase readability of code, even if it comes at a slight increase of gas costs.
        
    2.  ABI coder v2 is activated by default. You can choose to use the old behaviour using _pragma abicoder v1;_. The pragma _pragma experimental ABIEncoderV2;_ is still valid, but it is deprecated and has no effect. If you want to be explicit, please use _pragma abicoder v2;_ instead.
        
    3.  Exponentiation is right associative, i.e., the expression _a**b**c_ is parsed as _a**(b**c)_. Before 0.8.0, it was parsed as _(a**b)**c_. This is the common way to parse the exponentiation operator.
        
    4.  Failing assertions and other internal checks like division by zero or arithmetic overflow do not use the invalid opcode but instead the revert opcode. More specifically, they will use error data equal to a function call to Panic(uint256) with an error code specific to the circumstances. This will save gas on errors while it still allows static analysis tools to distinguish these situations from a revert on invalid input, like a failing require.
        
    5.  If a byte array in storage is accessed whose length is encoded incorrectly, a panic is caused. A contract cannot get into this situation unless inline assembly is used to modify the raw representation of storage byte arrays.
        
    6.  If constants are used in array length expressions, previous versions of Solidity would use arbitrary precision in all branches of the evaluation tree. Now, if constant variables are used as intermediate expressions, their values will be properly rounded in the same way as when they are used in run-time expressions.
        
    7.  The type byte has been removed. It was an alias of bytes1.
        
143.  Solidity v0.8.0 New Restrictions - changes that might cause existing contracts to not compile anymore:
    
    1.  Explicit conversions from negative literals and literals larger than type(uint160).max to address are disallowed.
        
    2.  Explicit conversions between literals and an integer type T are only allowed if the literal lies between type(T).min and type(T).max. In particular, replace usages of uint(-1) with type(uint).max.
        
    3.  Explicit conversions between literals and enums are only allowed if the literal can represent a value in the enum.
        
    4.  Explicit conversions between literals and address type (e.g. address(literal)) have the type address instead of address payable. One can get a payable address type by using an explicit conversion, i.e., payable(literal).
        
    5.  Address literals have the type _address_ instead of _address payable_. They can be converted to address payable by using an explicit conversion
        
    6.  Function call options can only be given once, i.e. c.f{gas: 10000}{value: 1}() is invalid and has to be changed to c.f{gas: 10000, value: 1}()
        
    7.  The global functions log0, log1, log2, log3 and log4 have been removed. These are low-level functions that were largely unused. Their behaviour can be accessed from inline assembly.
        
    8.  enum definitions cannot contain more than 256 members. This will make it safe to assume that the underlying type in the ABI is always uint8.
        
    9.  Declarations with the name this, super and _ are disallowed, with the exception of public functions and events. 
        
    10.  The global variables _tx.origin_ and _msg.sender_ have the type address instead of address payable. One can convert them into address payable by using an explicit conversion.
        
    11.  Explicit conversion into address type always returns a non-payable address type
        
    12.  The _chainid_ builtin in inline assembly is now considered view instead of pure
        
144.  Zero Address Check: address(0) which is 20-bytes of 0’s is treated specially in Solidity contracts because the private key corresponding to this address is unknown. Ether and tokens sent to this address cannot be retrieved and setting access control roles to this address also won’t work (no private key to sign transactions). Therefore zero addresses should be used with care and checks should be implemented for user-supplied address parameters.
    
145.  _tx.origin_ Check: Recall that Ethereum has two types of accounts: Externally Owned Account (EOA) and Contract Account. Transactions can originate only from EOAs. In situations where contracts would like to determine if the _msg.sender_ was a contract or not, checking if _msg.sender_ is equal to _tx.origin_ is an effective check. 
    
146.  Overflow/Underflow Check: Until Solidity version 0.8.0 which introduced checked arithmetic by default, arithmetic was unchecked and therefore susceptible to overflows and underflows which could lead to critical vulnerabilities. The recommended best-practice for such contracts is to use OpenZeppelin’s SafeMath library for arithmetic.
    
147.  OpenZeppelin Libraries: OpenZeppelin’s smart contract libraries are perhaps the most commonly used libraries in smart contract projects. These include contracts for popular token standards, access control, security, safe math, proxies and other utilities.
    
148.  OpenZeppelin ERC20: Implements the popular ERC20 token standard. The functions are:
    
    1.  _constructor(string name_, string symbol_)_: Sets the values for name and symbol. The default value of decimals is 18. To select a different value for decimals you should overload it. All three of these values are immutable: they can only be set once during construction.
        
    2.  _name()_ → _string_: Returns the name of the token.
        
    3.  _symbol()_ → _string_: Returns the symbol of the token, usually a shorter version of the name.
        
    4.  _decimals()_ → _uint8_: Returns the number of decimals used to get its user representation. For example, if decimals equals 2, a balance of 505 tokens should be displayed to a user as 5.05 (505 / 10 ** 2). Tokens usually opt for a value of 18, imitating the relationship between Ether and Wei. This is the value ERC20 uses, unless this function is overridden.
        
    5.  _totalSupply()_: Returns the amount of tokens in existence.
        
    6.  _balanceOf(address account)_ → _uint256_: Returns the amount of tokens owned by account
        
    7.  _transfer(address recipient, uint256 amount)_ → _bool_: Moves amount tokens from the caller’s account to recipient. Returns a boolean value indicating whether the operation succeeded. Emits a Transfer event.
        
    8.  _allowance(address owner, address spender)_ → _uint256_: Returns the remaining number of tokens that spender will be allowed to spend on behalf of owner through transferFrom. This is zero by default. This value changes when approve or transferFrom are called.
        
    9.  _approve(address spender, uint256 amount)_ → _bool_: Sets amount as the allowance of spender over the caller’s tokens. Returns a boolean value indicating whether the operation succeeded. Emits an Approval event. Warning: changing an allowance with this method brings the risk that someone may use both the old and the new allowance by unfortunate transaction ordering. One possible solution to mitigate this race condition is to first reduce the spender’s allowance to 0 and set the desired value afterwards: https://github.com/ethereum/EIPs/issues/20#issuecomment-263524729
        
    10.  _transferFrom(address sender, address recipient, uint256 amount)_ → _bool_: Moves amount tokens from sender to recipient using the allowance mechanism. amount is then deducted from the caller’s allowance. Returns a boolean value indicating whether the operation succeeded. Emits a Transfer event.
        
    11.  _increaseAllowance(address spender, uint256 addedValue)_ → _bool_ (Non-standard): Atomically increases the allowance granted to spender by the caller. This is an alternative to approve that can be used as a mitigation for the warning above. Emits an Approval event indicating the updated allowance. Requirement is that spender cannot be the zero address.
        
    12.  _decreaseAllowance(address spender, uint256 subtractedValue)_ → _bool_ (Non-standard): Atomically decreases the allowance granted to spender by the caller. This is an alternative to approve that can be used as a mitigation for the warning described above. Emits an Approval event indicating the updated allowance. Requirements are: 1) spender cannot be the zero address. 2) spender must have allowance for the caller of at least subtractedValue.
        

The different extensions/presets are:

1.  OpenZeppelin ERC20Burnable: Extension of ERC20 that allows token holders to destroy both their own tokens and those that they have an allowance for, in a way that can be recognized off-chain (via event analysis).
    
2.  OpenZeppelin ERC20Capped: Extension of ERC20 that adds a cap to the supply of tokens and enforces it in the mint function.
    
3.  OpenZeppelin ERC20Pausable: ERC20 token with pausable token transfers, minting and burning. Useful for scenarios such as preventing trades until the end of an evaluation period, or having an emergency switch for freezing all token transfers in the event of a large bug. The __beforeTokenTransfer()_ internal function enforces the not paused condition. 
    
4.  OpenZeppelin ERC20Snapshot: This contract extends an ERC20 token with a snapshot mechanism. When a snapshot is created, the balances and total supply at the time are recorded for later access. This can be used to safely create mechanisms based on token balances such as trustless dividends or weighted voting. Snapshots are created by the internal __snapshot_ function, which will emit the Snapshot event and return a snapshot id. To get the total supply at the time of a snapshot, call the function _totalSupplyAt_ with the snapshot id. To get the balance of an account at the time of a snapshot, call the _balanceOfAt_ function with the snapshot id and the account address.
    
5.  OpenZeppelin ERC20PresetFixedSupply: ERC20 token, including: 1) Preminted initial supply 2) Ability for holders to burn (destroy) their tokens 3) No access control mechanism (for minting/pausing) and hence no governance. This contract uses _ERC20Burnable_ contract to include burn capabilities
    
6.  OpenZeppelin ERC20PresetMinterPauser: ERC20 token, including: 1) ability for holders to burn (destroy) their tokens 2) a minter role that allows for token minting (creation) 3) a pauser role that allows to stop all token transfers. This contract uses _AccessControl_ contract to lock permissioned functions using the different roles. The account that deploys the contract will be granted the minter and pauser roles, as well as the default admin role, which will let it grant both minter and pauser roles to other accounts.
    

149.  OpenZeppelin SafeERC20: Wrappers around ERC20 operations that throw on failure when the token contract implementation returns false. Tokens that return no value and instead revert or throw on failure are also supported with non-reverting calls assumed to be successful. Adds _safeTransfer_, _safeTransferFrom_, _safeApprove_, _safeDecreaseAllowance_, and _safeIncreaseAllowance_.
    
150.  OpenZeppelin TokenTimelock: A token holder contract that will allow a beneficiary to extract the tokens after a given release time. Useful for simple vesting schedules like "advisors get all of their tokens after 1 year".
    
151.  OpenZeppelin ERC721: Implements the popular ERC721 Non-Fungible Token Standard. The functions are:
    
    1.  _balanceOf(address owner)_ → _uint256 balance_: Returns the number of tokens in owner's account.
        
    2.  _ownerOf(uint256 tokenId)_ → _address owner_: Returns the owner of the tokenId token. Requirements: tokenId must exist.
        
    3.  _transferFrom(address from, address to, uint256 tokenId)_: Transfers tokenId token from from to to. Requirements: from cannot be the zero address. to cannot be the zero address. tokenId token must be owned by from. If the caller is not from, it must be approved to move this token by either approve or setApprovalForAll. Emits a Transfer event. 
        
    4.  _safeTransferFrom(address from, address to, uint256 tokenId)_: Safely transfers tokenId token from from to to, checking first that contract recipients are aware of the ERC721 protocol to prevent tokens from being forever locked. Requirements: 1) from cannot be the zero address 2) to cannot be the zero address. 3) tokenId token must exist and be owned by from 4) If the caller is not from, it must be have been allowed to move this token by either approve or setApprovalForAll 5) If to refers to a smart contract, it must implement IERC721Receiver.onERC721Received, which is called upon a safe transfer. Emits a Transfer event. (The use of this function is encouraged over the related but unsafe transferFrom function.)
        
    5.  _approve(address to, uint256 tokenId)_: Gives permission to to to transfer tokenId token to another account. The approval is cleared when the token is transferred. Only a single account can be approved at a time, so approving the zero address clears previous approvals. Requirements: 1) The caller must own the token or be an approved operator 2) tokenId must exist. Emits an Approval event.
        
    6.  _getApproved(uint256 tokenId)_ → _address operator_: Returns the account approved for tokenId token. Requirements: tokenId must exist.
        
    7.  _setApprovalForAll(address operator, bool _approved)_: Approve or remove operator as an operator for the caller. Operators can call transferFrom or safeTransferFrom for any token owned by the caller. Requirements: The operator cannot be the caller. Emits an ApprovalForAll event.
        
    8.  _isApprovedForAll(address owner, address operator)_ → _bool_: Returns if the operator is allowed to manage all of the assets of owner.
        

The different extensions/presets/utilities are:

1.  OpenZeppelin ERC721Burnable: ERC721 Token that can be irreversibly burned (destroyed). 
    
2.  OpenZeppelin ERC721Enumerable: This implements an optional extension of ERC721 defined in the EIP that adds enumerability of all the token ids in the contract as well as all token ids owned by each account.
    
3.  OpenZeppelin ERC721Pausable: ERC721 token with pausable token transfers, minting and burning. Useful for scenarios such as preventing trades until the end of an evaluation period, or having an emergency switch for freezing all token transfers in the event of a large bug.
    
4.  OpenZeppelin ERC721URIStorage: ERC721 token with storage based token URI management.
    
5.  OpenZeppelin ERC721PresetMinterPauserAutoId: ERC721 token, including: 1) ability for holders to burn (destroy) their tokens 2) a minter role that allows for token minting (creation) 3) a pauser role that allows to stop all token transfers 4) token ID and URI autogeneration. This contract uses AccessControl to lock permissioned functions using the different roles. The account that deploys the contract will be granted the minter and pauser roles, as well as the default admin role, which will let it grant both minter and pauser roles to other accounts.
    
6.  OpenZeppelin ERC721Holder: Implementation of the IERC721Receiver interface. Accepts all token transfers. 
    

152.  OpenZeppelin ERC777: Like ERC20, ERC777 is a standard for fungible tokens with improvements such as getting rid of the confusion around decimals, minting and burning with proper events, among others, but its killer feature is receive hooks. ERC777 is backwards compatible with ERC20 (See [here](https://eips.ethereum.org/EIPS/eip-777))
    
    1.  A hook is simply a function in a contract that is called when tokens are sent to it, meaning accounts and contracts can react to receiving tokens. This enables a lot of interesting use cases, including atomic purchases using tokens (no need to do approve and transferFrom in two separate transactions), rejecting reception of tokens (by reverting on the hook call), redirecting the received tokens to other addresses, among many others. 
        
    2.  Both contracts and regular addresses can control and reject which token they send by registering a tokensToSend hook. (Rejection is done by reverting in the hook function.)
        
    3.  Both contracts and regular addresses can control and reject which token they receive by registering a tokensReceived hook. (Rejection is done by reverting in the hook function.)
        
    4.  The tokensReceived hook allows to send tokens to a contract and notify it in a single transaction, unlike ERC-20 which requires a double call (approve/transferFrom) to achieve this.
        
    5.  Furthermore, since contracts are required to implement these hooks in order to receive tokens, no tokens can get stuck in a contract that is unaware of the ERC777 protocol, as has happened countless times when using ERC20s. 
        
    6.  It mandates that decimals always returns a fixed value of 18, so there’s no need to set it ourselves
        
    7.  Has a concept of  _defaultOperators_ which are special accounts (usually other smart contracts) that will be able to transfer tokens on behalf of their holders
        
    8.  Implements _send_ (besides _transfer_) where if the recipient contract has not registered itself as aware of the ERC777 protocol then transfers to it are disabled to prevent tokens from being locked forever. Accounts can be notified of tokens being sent to them by having a contract implement this _IERC777Recipient_ interface and registering it on the ERC1820 global registry.
        
153.  OpenZeppelin ERC1155: is a novel token standard that aims to take the best from previous standards to create a fungibility-agnostic and gas-efficient token contract.
    
    1.  The distinctive feature of ERC1155 is that it uses a single smart contract to represent multiple tokens at once
        
    2.  Accounts have a distinct balance for each token id, and non-fungible tokens are implemented by simply minting a single one of them.
        
    3.  This approach leads to massive gas savings for projects that require multiple tokens. Instead of deploying a new contract for each token type, a single ERC1155 token contract can hold the entire system state, reducing deployment costs and complexity.
        
    4.  Because all state is held in a single contract, it is possible to operate over multiple tokens in a single transaction very efficiently. The standard provides two functions, _balanceOfBatch_ and _safeBatchTransferFrom_, that make querying multiple balances and transferring multiple tokens simpler and less gas-intensive.
        
154.  OpenZeppelin Ownable: provides a basic access control mechanism, where there is an account (an owner) that can be granted exclusive access to specific functions. By default, the owner account will be the one that deploys the contract. This can later be changed with _transferOwnership_. This module is used through inheritance. It will make available the modifier _onlyOwner_, which can be applied to your functions to restrict their use to the owner.
    
155.  OpenZeppelin AccessControl: provides a general role based access control mechanism. Multiple hierarchical roles can be created and assigned each to multiple accounts. Roles can be used to represent a set of permissions. _hasRole_ is used to restrict access to a function call. Roles can be granted and revoked dynamically via the _grantRole_ and _revokeRole_ functions which can only be called by the role’s associated admin accounts.
    

While the simplicity of _Ownable_ can be useful for simple systems or quick prototyping, different levels of authorization are often needed. You may want for an account to have permission to ban users from a system, but not create new tokens. Role-Based Access Control (RBAC) offers flexibility in this regard. We will effectively be defining multiple roles, each allowed to perform different sets of actions. An account may have, for example, 'moderator', 'minter' or 'admin' roles, which you will then check for instead of simply using _onlyOwner_. Separately, you will be able to define rules for how accounts can be granted a role, have it revoked, and more.

OpenZeppelin AccessControlEnumerable: Extension of AccessControl that allows enumerating the members of each role.

156.  OpenZeppelin Pausable: provides an emergency stop mechanism using functions pause and unpause that can be triggered by an authorized account. This module is used through inheritance. It will make available the modifiers _whenNotPaused_ and _whenPaused_, which can be applied to the functions of your contract. Only the functions using the modifiers will be affected when the contract is paused or unpaused.
    
157.  OpenZeppelin ReentrancyGuard: prevents reentrant calls to a function. Inheriting from ReentrancyGuard will make the _nonReentrant_ modifier available, which can be applied to functions to make sure there are no nested (reentrant) calls to them.
    
158.  OpenZeppelin PullPayment: provides a pull-payment strategy, where the paying contract doesn’t invoke any functions on the receiver account which must withdraw its payments itself. Pull-payments are often considered the best practice when it comes to sending Ether, security-wise. It prevents recipients from blocking execution and eliminates reentrancy concerns.
    
159.  OpenZeppelin Address: Collection of functions related to the address type:
    
    1.  _isContract(address account)_ → _bool_: Returns true if account is a contract. It is unsafe to assume that an address for which this function returns false is an externally-owned account (EOA) and not a contract. Among others, isContract will return false for the following types of addresses: 1) an externally-owned account 2) a contract in construction 3) an address where a contract will be created 4) an address where a contract lived, but was destroyed
        
    2.  _sendValue(address payable recipient, uint256 amount)_: Replacement for Solidity’s transfer: sends amount wei to recipient, forwarding all available gas and reverting on errors. EIP1884 increases the gas cost of certain opcodes, possibly making contracts go over the 2300 gas limit imposed by transfer, making them unable to receive funds via transfer. sendValue removes this limitation.
        
    3.  _functionCall(address target, bytes data)_ → _bytes_: Performs a Solidity function call using a low level call. A plain `call` is an unsafe replacement for a function call: use this function instead. If target reverts with a revert reason, it is bubbled up by this function (like regular Solidity function calls). Returns the raw returned data. Requirements: target must be a contract. calling target with data must not revert.
        
    4.  _functionCallWithValue(address target, bytes data, uint256 value)_ → _bytes_: Same as functionCall, but also transferring value wei to target. Requirements: 1) the calling contract must have an ETH balance of at least value. 2) the called Solidity function must be payable.
        
    5.  _functionStaticCall(address target, bytes data)_ → _bytes_: Same as functionCall, but performing a static call.
        
    6.  _functionDelegateCall(address target, bytes data)_ → _bytes_: Same as functionCall, but performing a delegate call.
        

The above functionCall* functions have variants which pass an _errorMessage_ parameter that specifies the fallback revert reason when target reverts.

160.  OpenZeppelin Arrays: Collection of functions related to array types:
    
    1.  _findUpperBound(uint256[] array, uint256 element)_ → _uint256_: Searches a sorted array and returns the first index that contains a value greater or equal to element. If no such index exists (i.e. all values in the array are strictly less than element), the array length is returned. Time complexity O(log n). array is expected to be sorted in ascending order, and to contain no repeated elements.
        
161.  OpenZeppelin Context: Provides information about the current execution context, including the sender of the transaction and its data. While these are generally available via _msg.sender_ and _msg.data_, they should not be accessed in such a direct manner, since when dealing with meta-transactions the account sending and paying for execution may not be the actual sender (as far as an application is concerned). This contract is only required for intermediate, library-like contracts. 
    
162.  OpenZeppelin Counters: Provides counters that can only be incremented or decremented by one. This can be used e.g. to track the number of elements in a mapping, issuing ERC721 ids, or counting request ids. Functions are:
    
    1.  current(struct Counters.Counter counter) → uint256
        
    2.  increment(struct Counters.Counter counter)
        
    3.  decrement(struct Counters.Counter counter)
        
163.  OpenZeppelin Create2: makes usage of the CREATE2 EVM opcode easier and safer. CREATE2 can be used to compute in advance the address where a smart contract will be deployed, which allows for interesting new mechanisms known as 'counterfactual interactions’. 
    
    1.  _deploy(uint256 amount, bytes32 salt, bytes bytecode)_ → _address_: Deploys a contract using CREATE2. The address where the contract will be deployed can be known in advance via computeAddress.The bytecode for a contract can be obtained from Solidity with type(contractName).creationCode. Requirements: 1) bytecode must not be empty. 2) salt must have not been used for bytecode already. 3) the factory must have a balance of at least amount. 4) if amount is non-zero, bytecode must have a payable constructor.
        
    2.  _computeAddress(bytes32 salt, bytes32 bytecodeHash)_ → _address_: Returns the address where a contract will be stored if deployed via deploy. Any change in the bytecodeHash or salt will result in a new destination address.
        
    3.  _computeAddress(bytes32 salt, bytes32 bytecodeHash, address deployer)_ → _address_: Returns the address where a contract will be stored if deployed via deploy from a contract located at deployer. If the deployer is this contract’s address, it returns the same value as computeAddress.
        
164.  OpenZeppelin Multicall: Provides a function to batch together multiple calls in a single external call
    
    1.  _multicall(bytes[] calldata data) external_ → _bytes[]_: Receives and executes a batch of function calls on this contract
        
165.  OpenZeppelin Strings: String operations:
    
    1.  _toString(uint256 value)_ → _string_: Converts a uint256 to its ASCII string decimal representation.
        
    2.  _toHexString(uint256 value)_ → _string_: Converts a uint256 to its ASCII string hexadecimal representation.
        
    3.  _toHexString(uint256 value, uint256 length)_ → string: Converts a uint256 to its ASCII string hexadecimal representation with fixed length.
        
166.  OpenZeppelin ECDSA: provides functions for recovering and managing Ethereum account ECDSA signatures. These are often generated via _web3.eth.sign_, and are a 65 byte array (of type bytes in Solidity) arranged the following way: [[v (1)], [r (32)], [s (32)]]. The data signer can be recovered with _ECDSA.recover_, and its address compared to verify the signature. Most wallets will hash the data to sign and add the prefix '\x19Ethereum Signed Message:\n', so when attempting to recover the signer of an Ethereum signed message hash, you’ll want to use _toEthSignedMessageHash._
    
    1.  The `ecrecover` EVM opcode allows for malleable (non-unique) signatures. _This library prevents that by requiring the `s` value to be in the lower half order, and the `v` value to be either 27 or 28._
        
167.  OpenZeppelin MerkleProof: This deals with verification of Merkle Trees proofs.
    
    1.  _verify:_ which can prove that some value is part of a Merkle tree. Returns true if a `leaf` can be proved to be a part of a Merkle tree defined by `root`. For this, a `proof` must be provided, containing sibling hashes on the branch from the leaf to the root of the tree. Each pair of leaves and each pair of pre-images are assumed to be sorted.
        
168.  OpenZeppelin SignatureChecker: Provide a single mechanism to verify both private-key (EOA) ECDSA signature and ERC1271 contract signatures. Using this instead of ECDSA.recover in your contract will make them compatible with smart contract wallets such as Argent and Gnosis.
    
    1.  Externally Owned Accounts (EOA) can sign messages with their associated private keys, but currently contracts cannot. This is a problem for many applications that implement signature based off-chain methods, since contracts can't easily interact with them as they do not possess a private key. ERC 1271 proposes a standard way for any contracts to verify whether a signature on behalf of a given contract is valid.
        
    2.  Note: unlike ECDSA signatures, contract signature's are revocable, and the outcome of this function can thus change through time. It could return true at block N and false at block N+1 (or the opposite).
        
169.  OpenZeppelin EIP712: EIP 712 is a standard for hashing and signing of typed structured data. This contract implements the EIP 712 domain separator (__domainSeparatorV4_) that is used as part of the encoding scheme, and the final step of the encoding to obtain the message digest that is then signed via ECDSA (__hashTypedDataV4_). Protocols need to implement the type-specific encoding they need in their contracts using a combination of abi.encode and keccak256.
    
    1.  _constructor(string name, string version)_: Initializes the domain separator and parameter caches. The meaning of name and version is specified in EIP 712: 1) name is the user readable name of the signing domain, i.e. the name of the DApp or the protocol 2) version: the current major version of the signing domain.
        
    2.  __domainSeparatorV4()_ → _bytes32_: Returns the domain separator for the current chain.
        
        1.  __hashTypedDataV4(bytes32 structHash)_ → _bytes32_: Given an already hashed struct, this function returns the hash of the fully encoded EIP712 message for this domain. This hash can be used together with ECDSA.recover to obtain the signer of a message.
            
170.  OpenZeppelin Escrow: holds funds designated for a payee until they withdraw them. The contract that uses this escrow as its payment method should be its owner, and provide public methods redirecting to the escrow's deposit and withdraw if the escrow rules are satisfied.
    
    1.  _depositsOf(address payee)_ → _uint256_: 
        
    2.  _deposit(address payee)_: Stores the sent amount as credit to be withdrawn.
        
    3.  _withdraw(address payable payee)_: Withdraw accumulated balance for a payee, forwarding all gas to the recipient.
        
171.  OpenZeppelin ConditionalEscrow: Derived from Escrow and only allows withdrawal if a condition is met by providing the _withdrawalAllowed()_ function which returns whether an address is allowed to withdraw their funds and is to be implemented by derived contracts.
    
172.  OpenZeppelin RefundEscrow: Derived from ConditionalEscrow and holds funds for a beneficiary, deposited from multiple parties. The owner account (that is, the contract that instantiates this contract) may deposit, close the deposit period, and allow for either withdrawal by the beneficiary, or refunds to the depositors.
    
173.  OpenZeppelin ERC165: In Solidity, it’s frequently helpful to know whether or not a contract supports an interface you’d like to use. ERC165 is a standard that helps do runtime interface detection using a lookup table. You can register interfaces using __registerInterface(bytes4)_ and _supportsInterface(bytes4 interfaceId)_ returns a bool indicating if that interface is supported or not.
    
174.  OpenZeppelin Math: Standard math utilities missing in the Solidity language:
    
    1.  _max(uint256 a, uint256 b)_: Returns the larger of two numbers
        
    2.  _min(uint256 a, uint256 b)_: Returns the smaller of two numbers
        
    3.  _average(uint256 a, uint256 b)_: Returns the average of two numbers. The result is rounded towards zero.
        
175.  OpenZeppelin SafeMath: provides mathematical functions that protect your contract from overflows and underflows. Include the contract with using SafeMath for uint256; and then call the functions:
    
    1.  myNumber.add(otherNumber): Returns the addition of two unsigned integers, reverting on overflow. Counterpart to Solidity's `+` operator.
        
    2.  myNumber.sub(otherNumber): Returns the subtraction of two unsigned integers, reverting on overflow (when the result is negative). Counterpart to Solidity's `-` operator.
        
    3.  myNumber.div(otherNumber): Returns the division of two unsigned integers, reverting on overflow. The result is rounded towards zero. Counterpart to Solidity's `/` operator.
        
    4.  myNumber.mul(otherNumber): Returns the multiplication of two unsigned integers, reverting on overflow. Counterpart to Solidity's `*` operator.
        
    5.  myNumber.mod(otherNumber): Returns the modulus of two unsigned integers, reverting when dividing by zero. Counterpart to Solidity's `%` operator.
        

The corresponding try* functions return results with an overflow flag instead of reverting.

176.  OpenZeppelin SignedSafeMath: provides the same mathematical functions as SafeMath but for signed integers
    
    1.  myNumber.add(otherNumber): Returns the addition of two signed integers, reverting on overflow. Counterpart to Solidity's `+` operator.
        
    2.  myNumber.sub(otherNumber): Returns the subtraction of two signed integers, reverting on overflow (when the result is negative). Counterpart to Solidity's `-` operator.
        
    3.  myNumber.div(otherNumber): Returns the division of two signed integers, reverting on overflow. The result is rounded towards zero. Counterpart to Solidity's `/` operator.
        
    4.  myNumber.mul(otherNumber): Returns the multiplication of two signed integers, reverting on overflow. Counterpart to Solidity's `*` operator.
        
177.  OpenZeppelin SafeCast: Wrappers over Solidity's uintXX/intXX casting operators with added overflow checks. Downcasting from uint256/int256 in Solidity does not revert on overflow. This can easily result in undesired exploitation or bugs, since developers usually assume that overflows raise errors. `SafeCast` restores this intuition by reverting the transaction when such an operation overflows.
    
    1.  _toUint128(uint256 value) returns (uint128)_: Returns the downcasted uint128 from uint256, reverting on overflow (when the input is greater than largest uint128). Similar functions are available for toUint64(uint256 value), toUint32(uint256 value), toUint16(uint256 value), toUint8(uint256 value)
        
    2.  _toInt128(int256 value) internal pure returns (uint256)_: Returns the downcasted int128 from int256, reverting on overflow (when the input is less than smallest int128 or greater than largest int128). Similar functions are available for toInt64(int256 value), toInt32(int256 value), toInt16(int256 value), toInt8(int256 value).
        
    3.  _function toInt256(uint256 value) returns (int256)_: Converts an unsigned uint256 into a signed int256
        
    4.  _function toUint256(int256 value)_ returns (uint256): Converts a signed int256 into an unsigned uint256
        
    5.  Similar functions downcasting to 224/96/64/32/16/8 bits for both unsigned and signed.
        
178.  OpenZeppelin EnumerableMap: Library for managing an enumerable variant of Solidity’s mapping type. Maps have the following properties: 1) Entries are added, removed, and checked for existence in constant time (O(1)) 2) Entries are enumerated in O(n). No guarantees are made on the ordering. As of v3.0.0, only maps of type uint256 → address (UintToAddressMap) are supported.
    
    1.  _set(struct EnumerableMap.UintToAddressMap map, uint256 key, address value)_ → _bool_: Adds a key-value pair to a map, or updates the value for an existing key. Returns true if the key was added to the map, that is if it was not already present.
        
    2.  _remove(struct EnumerableMap.UintToAddressMap map, uint256 key)_ → _bool_: Removes a value from a set. Returns true if the key was removed from the map, that is if it was present.
        
    3.  _contains(struct EnumerableMap.UintToAddressMap map, uint256 key)_ → _bool_: Returns true if the key is in the map.
        
    4.  _length(struct EnumerableMap.UintToAddressMap map)_ → _uint256_: Returns the number of elements in the map.
        
    5.  _at(struct EnumerableMap.UintToAddressMap map, uint256 index)_ → _uint256, address_: Returns the element stored at position index in the set. Note that there are no guarantees on the ordering of values inside the array, and it may change when more values are added or removed. Requirements: index must be strictly less than length.
        
    6.  _tryGet(struct EnumerableMap.UintToAddressMap map, uint256 key)_ → _bool, address_: Tries to return the value associated with key. Does not revert if key is not in the map.
        
    7.  _get(struct EnumerableMap.UintToAddressMap map, uint256 key)_ → _address:_ Returns the value associated with key. Requirements: key must be in the map.
        
179.  OpenZeppelin EnumerableSet: Library for managing sets of primitive types. Sets have the following properties: 1) Elements are added, removed, and checked for existence in constant time (O(1)) 2) Elements are enumerated in O(n). No guarantees are made on the ordering. As of v3.3.0, sets of type bytes32 (Bytes32Set), address (AddressSet) and uint256 (UintSet) are supported.
    
    1.  _add(struct EnumerableSet.Bytes32Set set, bytes32 value)_ → _bool_: Add a value to a set. Returns true if the value was added to the set, that is if it was not already present.
        
    2.  _remove(struct EnumerableSet.Bytes32Set set, bytes32 value)_ → _bool_: Removes a value from a set. Returns true if the value was removed from the set, that is if it was present.
        
    3.  _contains(struct EnumerableSet.Bytes32Set set, bytes32 value)_ → _bool_: Returns true if the value is in the set.
        
    4.  _length(struct EnumerableSet.Bytes32Set set)_ → _uint256_: Returns the number of values in the set.
        
    5.  _at(struct EnumerableSet.Bytes32Set set, uint256 index)_ → _bytes32_: Returns the value stored at position index in the set. Note that there are no guarantees on the ordering of values inside the array, and it may change when more values are added or removed. Requirements: index must be strictly less than length.
        
    6.  _add(struct EnumerableSet.AddressSet set, address value)_ → _bool_: Add a value to a set. Returns true if the value was added to the set, that is if it was not already present.
        
    7.  _remove(struct EnumerableSet.AddressSet set, address value)_ → _bool_: Removes a value from a set. Returns true if the value was removed from the set, that is if it was present.
        
    8.  _contains(struct EnumerableSet.AddressSet set, address value)_ → _bool_: Returns true if the value is in the set. length(struct EnumerableSet.AddressSet set) → uint256: Returns the number of values in the set.
        
    9.  _at(struct EnumerableSet.AddressSet set, uint256 index)_ → _address_: Returns the value stored at position index in the set. Note that there are no guarantees on the ordering of values inside the array, and it may change when more values are added or removed. Requirements: index must be strictly less than length.
        
    10.  _add(struct EnumerableSet.UintSet set, uint256 value)_ → _bool_: Add a value to a set. Returns true if the value was added to the set, that is if it was not already present.
        
    11.  _remove(struct EnumerableSet.UintSet set, uint256 value)_ → _bool_: Removes a value from a set. Returns true if the value was removed from the set, that is if it was present.
        
    12.  _contains(struct EnumerableSet.UintSet set, uint256 value)_ → _bool_: Returns true if the value is in the set. O(1).
        
    13.  _length(struct EnumerableSet.UintSet set)_ → _uint256_: Returns the number of values on the set.
        
    14.  _at(struct EnumerableSet.UintSet set, uint256 index)_ → _uint256_: Returns the value stored at position index in the set. Note that there are no guarantees on the ordering of values inside the array, and it may change when more values are added or removed. Requirements: index must be strictly less than length.
        
180.  OpenZeppelin BitMaps: Library for managing uint256 to bool mapping in a compact and efficient way, providing the keys are sequential.
    
    1.  struct BitMap: mapping(uint256 => uint256) _data;
        
    2.  get(BitMap storage bitmap, uint256 index) → _bool:_ Returns whether the bit at `index` is set.
        
    3.  setTo(BitMap storage bitmap, uint256 index, bool value): Sets the bit at `index` to the boolean `value`
        
    4.  function set(BitMap storage bitmap, uint256 index): Sets the bit at `index`
        
    5.  function unset(BitMap storage bitmap, uint256 index): Unsets the bit at `index`
        
181.  OpenZeppelin PaymentSplitter: allows to split Ether payments among a group of accounts. The sender does not need to be aware that the Ether will be split in this way, since it is handled transparently by the contract. The split can be in equal parts or in any other arbitrary proportion. The way this is specified is by assigning each account to a number of shares. Of all the Ether that this contract receives, each account will then be able to claim an amount proportional to the percentage of total shares they were assigned.
    
    1.  PaymentSplitter follows a pull payment model. This means that payments are not automatically forwarded to the accounts but kept in this contract, and the actual transfer is triggered as a separate step by calling the release function.
        
182.  OpenZeppelin TimelockController: acts as a timelocked controller. When set as the owner of an Ownable smart contract, it enforces a timelock on all _onlyOwner_ maintenance operations. This gives time for users of the controlled contract to exit before a potentially dangerous maintenance operation is applied. By default, this contract is self administered, meaning administration tasks have to go through the timelock process. The proposer (resp executor) role is in charge of proposing (resp executing) operations. A common use case is to position this TimelockController as the owner of a smart contract, with a multisig or a DAO as the sole proposer.
    
    1.  _constructor(uint256 minDelay, address[] proposers, address[] executors)_: Initializes the contract with a given minDelay.
        
    2.  _receive()_: Contract might receive/hold ETH as part of the maintenance process.
        
    3.  _isOperation(bytes32 id)_ → _bool pending_: Returns whether an id corresponds to a registered operation. This includes both Pending, Ready and Done operations.
        
    4.  _isOperationPending(bytes32 id)_ → _bool pending_: Returns whether an operation is pending or not.
        
    5.  _isOperationReady(bytes32 id)_ → _bool ready_: Returns whether an operation is ready or not.
        
    6.  _isOperationDone(bytes32 id)_ → _bool done_: Returns whether an operation is done or not.
        
    7.  _getTimestamp(bytes32 id)_ → _uint256 timestamp_: Returns the timestamp at which an operation becomes ready (0 for unset operations, 1 for done operations).
        
    8.  _getMinDelay()_ → _uint256 duration_: Returns the minimum delay for an operation to become valid. This value can be changed by executing an operation that calls updateDelay.
        
    9.  _hashOperation(address target, uint256 value, bytes data, bytes32 predecessor, bytes32 salt)_ → _bytes32 hash_: Returns the identifier of an operation containing a single transaction.
        
    10.  _hashOperationBatch(address[] targets, uint256[] values, bytes[] datas, bytes32 predecessor, bytes32 salt)_ → _bytes32 hash_: Returns the identifier of an operation containing a batch of transactions.
        
    11.  _schedule(address target, uint256 value, bytes data, bytes32 predecessor, bytes32 salt, uint256 delay)_: Schedule an operation containing a single transaction. Emits a CallScheduled event. Requirements: the caller must have the 'proposer' role.
        
    12.  _scheduleBatch(address[] targets, uint256[] values, bytes[] datas, bytes32 predecessor, bytes32 salt, uint256 delay)_: Schedule an operation containing a batch of transactions. Emits one CallScheduled event per transaction in the batch. Requirements: the caller must have the 'proposer' role.
        
    13.  _cancel(bytes32 id)_: Cancel an operation. Requirements: the caller must have the 'proposer' role.
        
    14.  _execute(address target, uint256 value, bytes data, bytes32 predecessor, bytes32 salt)_: Execute an (ready) operation containing a single transaction. Emits a CallExecuted event. Requirements: the caller must have the 'executor' role.
        
    15.  _executeBatch(address[] targets, uint256[] values, bytes[] datas, bytes32 predecessor, bytes32 salt)_: Execute an (ready) operation containing a batch of transactions. Emits one CallExecuted event per transaction in the batch. Requirements: the caller must have the 'executor' role.
        
    16.  _updateDelay(uint256 newDelay)_: Changes the minimum timelock duration for future operations. Emits a MinDelayChange event. Requirements: the caller must be the timelock itself. This can only be achieved by scheduling and later executing an operation where the timelock is the target and the data is the ABI-encoded call to this function.
        
183.  OpenZeppelin ERC2771Context: A Context variant for ERC2771. ERC2771 provides support for meta transactions, which are transactions that have been:
    
    1.  Authorized by the Transaction Signer. For example, signed by an externally owned account
        
    2.  Relayed by an untrusted third party that pays for the gas (the Gas Relay)
        

The problem is that for a contract that is not natively aware of meta transactions, the _msg.sender_ of the transaction will make it appear to be coming from the Gas Relay and not the Transaction Signer. A secure protocol for a contract to accept meta transactions needs to prevent the Gas Relay from forging, modifying or duplicating requests by the Transaction Signer. The entities are:

1.  Transaction Signer - entity that signs & sends to request to Gas Relay
    
2.  Gas Relay - receives a signed request off-chain from Transaction Signer and pays gas to turn it into a valid transaction that goes through Trusted Forwarder
    
3.  Trusted Forwarder - a contract that is trusted by the Recipient to correctly verify the signature and nonce before forwarding the request from Transaction Signer
    
4.  Recipient - a contract that can securely accept meta-transactions through a Trusted Forwarder by being compliant with this standard.
    

184.  OpenZeppelin MinimalForwarder: provides a simple minimal forwarder (as described above) to be used together with an ERC2771 compatible contract. It verifies the nonce and signature of the forwarded request before calling the destination contract.
    
    1.  struct ForwardRequest {address from; address to; uint256 value; uint256 gas; uint256 nonce; bytes data;}
        
    2.  verify(ForwardRequest calldata req, bytes calldata signature) public view → (bool)
        
    3.  execute(ForwardRequest calldata req, bytes calldata signature) → (success, returndata)
        
185.  OpenZeppelin Proxy: This abstract contract provides a fallback function that delegates all calls to another contract using the EVM instruction delegatecall. We refer to the second contract as the implementation behind the proxy, and it has to be specified by overriding the virtual _implementation function. Additionally, delegation to the implementation can be triggered manually through the _fallback function, or to a different contract through the _delegate function. The success and return data of the delegated call will be returned back to the caller of the proxy.
    
    1.  __delegate(address implementation)_: Delegates the current call to implementation. This function does not return to its internal call site, it will return directly to the external caller.
        
    2.  __implementation()_ → _address_: This is a virtual function that should be overridden so it returns the address to which the fallback function and __fallback_ should delegate.
        
    3.  __fallback()_: Delegates the current call to the address returned by __implementation()_. This function does not return to its internal call site, it will return directly to the external caller.
        
    4.  _fallback()_: Fallback function that delegates calls to the address returned by __implementation()_. Will run if no other function in the contract matches the call data.
        
    5.  _receive()_: Fallback function that delegates calls to the address returned by __implementation()_. Will run if call data is empty.
        
    6.  __beforeFallback()_: Hook that is called before falling back to the implementation. Can happen as part of a manual _fallback call, or as part of the Solidity fallback or receive functions. If overridden, should call _super._beforeFallback()_.
        
186.  OpenZeppelin ERC1967Proxy: implements an upgradeable proxy. It is upgradeable because calls are delegated to an implementation address that can be changed. This address is stored in storage in the location specified by EIP1967, so that it doesn’t conflict with the storage layout of the implementation behind the proxy. Upgradeability is only provided internally through __upgradeTo_.
    
    1.  _constructor(address _logic, bytes _data)_: Initializes the upgradeable proxy with an initial implementation specified by _logic. If _data is nonempty, it’s used as data in a delegate call to _logic. This will typically be an encoded function call, and allows initializing the storage of the proxy like a Solidity constructor.
        
    2.  __implementation()_ → _address_ _impl_: Returns the current implementation address.
        
    3.  __upgradeTo(address newImplementation)_: Upgrades the proxy to a new implementation. Emits an Upgraded event.
        
187.  OpenZeppelin TransparentUpgradeableProxy: implements a proxy that is upgradeable by an admin. To avoid proxy selector clashing, which can potentially be used in an attack, this contract uses the transparent proxy pattern. This pattern implies two things that go hand in hand: 1) If any account other than the admin calls the proxy, the call will be forwarded to the implementation, even if that call matches one of the admin functions exposed by the proxy itself 2) If the admin calls the proxy, it can access the admin functions, but its calls will never be forwarded to the implementation. If the admin tries to call a function on the implementation it will fail with an error that says "admin cannot fallback to proxy target”. 
    

These properties mean that the admin account can only be used for admin actions like upgrading the proxy or changing the admin, so it’s best if it’s a dedicated account that is not used for anything else. This will avoid headaches due to sudden errors when trying to call a function from the proxy implementation.

1.  _constructor(address _logic, address admin_, bytes _data)_: Initializes an upgradeable proxy managed by _admin, backed by the implementation at _logic, and optionally initialized with _data.
    
2.  _admin()_ → _address admin__: Returns the current admin.
    
3.  _implementation()_ → _address implementation__: Returns the current implementation.
    
4.  _changeAdmin(address newAdmin)_: Changes the admin of the proxy. Emits an AdminChanged event.
    
5.  _upgradeTo(address newImplementation)_: Upgrade the implementation of the proxy.
    
6.  _upgradeToAndCall(address newImplementation, bytes data)_: Upgrade the implementation of the proxy, and then call a function from the new implementation as specified by data, which should be an encoded function call. This is useful to initialize new storage variables in the proxied contract.
    
7.  __admin()_ → _address adm_: Returns the current admin.
    
8.  __beforeFallback()_: Makes sure the admin cannot access the fallback function.
    

188.  OpenZeppelin ProxyAdmin: This is an auxiliary contract meant to be assigned as the admin of a TransparentUpgradeableProxy. 
    
    1.  _getProxyImplementation(contract TransparentUpgradeableProxy proxy)_ → _address_: Returns the current implementation of proxy. Requirements: This contract must be the admin of proxy.
        
    2.  _getProxyAdmin(contract TransparentUpgradeableProxy proxy)_ → _address_: Returns the current admin of proxy. Requirements: This contract must be the admin of proxy.
        
    3.  _changeProxyAdmin(contract TransparentUpgradeableProxy proxy, address newAdmin)_: Changes the admin of proxy to newAdmin. Requirements: This contract must be the current admin of proxy.
        
    4.  _upgrade(contract TransparentUpgradeableProxy proxy, address implementation)_: Upgrades proxy to implementation. Requirements: This contract must be the admin of proxy.
        
    5.  _upgradeAndCall(contract TransparentUpgradeableProxy proxy, address implementation, bytes data)_: Upgrades proxy to implementation and calls a function on the new implementation. Requirements: This contract must be the admin of proxy.
        
189.  OpenZeppelin BeaconProxy: implements a proxy that gets the implementation address for each call from a UpgradeableBeacon. The beacon address is stored in storage slot uint256(keccak256('eip1967.proxy.beacon')) - 1, so that it doesn’t conflict with the storage layout of the implementation behind the proxy.
    
    1.  _constructor(address beacon, bytes data)_: Initializes the proxy with beacon. If data is nonempty, it’s used as data in a delegate call to the implementation returned by the beacon. This will typically be an encoded function call, and allows initializing the storage of the proxy like a Solidity constructor. Requirements: beacon must be a contract with the interface IBeacon.
        
    2.  __beacon()_ → _address beacon_: Returns the current beacon address.
        
    3.  __implementation()_ → _address_: Returns the current implementation address of the associated beacon.
        
    4.  __setBeacon(address beacon, bytes data)_: Changes the proxy to use a new beacon. If data is nonempty, it’s used as data in a delegate call to the implementation returned by the beacon. Requirements: 1) beacon must be a contract 2) The implementation returned by beacon must be a contract.
        
190.  OpenZeppelin UpgradeableBeacon: is used in conjunction with one or more instances of BeaconProxy to determine their implementation contract, which is where they will delegate all function calls. An owner is able to change the implementation the beacon points to, thus upgrading the proxies that use this beacon.
    
    1.  _constructor(address implementation_)_: Sets the address of the initial implementation, and the deployer account as the owner who can upgrade the beacon.
        
    2.  _implementation()_ → _address_: Returns the current implementation address.
        
    3.  _upgradeTo(address newImplementation)_: Upgrades the beacon to a new implementation. Emits an Upgraded event. Requirements: 1) msg.sender must be the owner of the contract 2) newImplementation must be a contract.
        
191.  OpenZeppelin Clones: EIP 1167 is a standard for deploying minimal proxy contracts, also known as “clones". To simply and cheaply clone contract functionality in an immutable way, this standard specifies a minimal bytecode implementation that delegates all calls to a known, fixed address. The library includes functions to deploy a proxy using either create (traditional deployment) or create2 (salted deterministic deployment). It also includes functions to predict the addresses of clones deployed using the deterministic method.
    
    1.  _clone(address implementation)_ → _address instance_: Deploys and returns the address of a clone that mimics the behaviour of implementation. This function uses the create opcode, which should never revert.
        
    2.  _cloneDeterministic(address implementation, bytes32 salt)_ → _address instance_: Deploys and returns the address of a clone that mimics the behaviour of implementation. This function uses the create2 opcode and a salt to deterministically deploy the clone. Using the same implementation and salt multiple times will revert, since the clones cannot be deployed twice at the same address.
        
    3.  _predictDeterministicAddress(address implementation, bytes32 salt, address deployer)_ → address predicted: Computes the address of a clone deployed using Clones.cloneDeterministic.
        
    4.  _predictDeterministicAddress(address implementation, bytes32 salt)_ → _address predicted_: Computes the address of a clone deployed using Clones.cloneDeterministic.
        
192.  OpenZeppelin Initializable: aids in writing upgradeable contracts, or any kind of contract that will be deployed behind a proxy. Since a proxied contract cannot have a constructor, it is common to move constructor logic to an external initializer function, usually called _initialize_. It then becomes necessary to protect this initializer function so it can only be called once. The initializer modifier provided by this contract will have this effect. 
    

To avoid leaving the proxy in an uninitialized state, the initializer function should be called as early as possible by providing the encoded function call as the _data argument. When used with inheritance, manual care must be taken to not invoke a parent initializer twice, or to ensure that all initializers are idempotent. This is not verified automatically as constructors are by Solidity.

193.  Dappsys DSProxy: implements a proxy deployed as a standalone contract which can then be used by the owner to execute code. A user would pass in the bytecode for the contract as well as the calldata for the function they want to execute.The proxy will create a contract using the bytecode. It will then delegatecall the function and arguments specified in the calldata.
    
194.  Dappsys DSMath: provides arithmetic functions for the common numerical primitive types of Solidity. You can safely add, subtract, multiply, and divide uint numbers without fear of integer overflow. You can also find the minimum and maximum of two numbers. Additionally, this package provides arithmetic functions for two new higher level numerical concepts called _wad_ (18 decimals) and _ray_ (27 decimals). These are used to represent fixed-point decimal numbers. A wad is a decimal number with 18 digits of precision and a ray is a decimal number with 27 digits of precision. These functions are necessary to account for the difference between how integer arithmetic behaves normally, and how decimal arithmetic should actually work.
    

The standard functions are the uint set, so their function names are not  prefixed: _add_, _sub_, _mul_, _min_, and _max_. There is no div function, as divide-by-zero checking is built into the Solidity compiler. The int functions have an _i_ prefix: _imin_ and _imax_. Wad functions have a _w_ prefix: _wmul_, _wdiv_. Ray functions have a _r_ prefix: _rmul_, _rdiv_, _rpow_.

195.  Dappsys DSAuth: Provides a flexible and updatable auth pattern which is completely separate from application logic. By default, the auth modifier will restrict function-call access to the including contract owner and the including contract itself. The auth modifier provided by DSAuth triggers the internal isAuthorized function to require that the msg.sender is authorized ie. the sender is either: 1) the contract owner 2) the contract itself or 3) has been granted permission via a specified authority.
    
196.  Dappsys DSGuard: Manages an Access Control List which maps source and destination addresses to function signatures. Intended to be used as an authority for DSAuth where it acts as a lookup table for the _canCall_ function to provide boolean answers as to whether a particular address is authorized to call a given function at another address. The ACL is a mapping of _[src][dst][sig] => boolean_ where an address src can be either permitted or forbidden access to a function _sig_ at address _dst_ according to the boolean value. When used as an authority by DSAuth the _src_ is considered to be the _msg.sender_, the _dst_ is the including contract and _sig_ is the function which invoked the auth modifier.
    
197.  Dappsys DSRoles: A role-driven authority for ds-auth which facilitates access to lists of user roles and capabilities. Works as a set of lookup tables for the canCall function to provide boolean answers as to whether a user is authorized to call a given function at given address. DSRoles provides 3 different ways of permitting/forbidding function call access to users:1) Root Users: any users added to the _root_users whitelist will be authorized to call any function regardless of what roles or capabilities might be defined. 2) Public Capabilities: public capabilities are global capabilities which apply to all users and take precedence over any user specific role-capabilities which might be defined. 3) Role Capabilities: capabilities which are associated with a particular role. Role capabilities are only checked if the user does not have root access and the capability is not public.
    
198.  WETH: WETH stands for Wrapped Ether. For protocols that work with ERC-20 tokens but also need to handle Ether, WETH contracts allow converting Ether to its ERC-20 equivalent WETH (called wrapping) and vice-versa (called unwrapping).  WETH can be created by sending ether to a WETH smart contract where the Ether is stored and in turn receiving the WETH ERC-20 token at a 1:1 ratio. This WETH can be sent back to the same smart contract to be “unwrapped” i.e. redeemed back for the original Ether at a 1:1 ratio. The most widely used WETH contract is [WETH9](https://etherscan.io/address/0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2#code) which holds more than 7 million Ether for now.
    
199.  Uniswap V2: Uniswap is an automated liquidity protocol powered by a constant product formula and implemented in a system of non-upgradeable smart contracts on the Ethereum blockchain. The automated market making algorithm used by Uniswap is _x*y=k_, where x and y represent a token pair that allow one token to be exchanged for the other as long as the “constant product” formula is preserved i.e. trades must not change the product (k) of a pair’s reserve balances (x and y). Core concepts:
    
    1.  Pools: Each Uniswap liquidity pool is a trading venue for a pair of ERC20 tokens. When a pool contract is created, its balances of each token are 0; in order for the pool to begin facilitating trades, someone must seed it with an initial deposit of each token. This first liquidity provider is the one who sets the initial price of the pool. They are incentivized to deposit an equal value of both tokens into the pool. Whenever liquidity is deposited into a pool, unique tokens known as liquidity tokens are minted and sent to the provider’s address. These tokens represent a given liquidity provider’s contribution to a pool. 
        
    2.  Swaps: allows one to trade one ERC-20 token for another, where one token is withdrawn (purchased) and a proportional amount of the other deposited (sold), in order to maintain the constant _x*y=k_
        
    3.  Flash Swaps: allows one to withdraw up to the full reserves of any ERC20 token on Uniswap and execute arbitrary logic at no upfront cost, provided that by the end of the transaction they either: 1) pay for the withdrawn ERC20 tokens with the corresponding pair tokens 2) return the withdrawn ERC20 tokens along with a small fee
        
    4.  Oracles: enables developers to build highly decentralized and manipulation-resistant on-chain price oracles. A price oracle is any tool used to view price information about a given asset. Every pair measures (but does not store) the market price at the beginning of each block, before any trades take place i.e. price at the end of the previous block which is added to a single cumulative-price variable weighted by the amount of time this price existed. This variable can be used by external contracts to track accurate time-weighted average prices (TWAPs) across any time interval.
        
200.  Uniswap V3: [Introduces](https://uniswap.org/blog/uniswap-v3/)
    
    1.  Concentrated liquidity: giving individual LPs granular control over what price ranges their capital is allocated to. Individual positions are aggregated together into a single pool, forming one combined curve for users to trade against
        
    2.  Multiple fee tiers: allowing LPs to be appropriately compensated for taking on varying degrees of risk
        
    3.  V3 oracles are capable of providing time-weighted average prices (TWAPs) on demand for any period within the last ~9 days. This removes the need for integrators to checkpoint historical values.
        
201.  Chainlink Oracles & Price Feeds: Chainlink Price Feeds provide aggregated data (via its _AggregatorV3Interface_ contract interface) from various high quality data providers, fed on-chain by decentralized oracles on the Chainlink Network. To get price data into smart contracts for an asset that isn’t covered by an existing price feed, such as the price of a particular stock, one can customize Chainlink oracles to call any external API.