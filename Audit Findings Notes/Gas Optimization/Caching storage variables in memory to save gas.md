It's cheaper in gas cost to cache the variable in memory.

`SLOAD` costs 1000 gas, while `MLOAD` and `MSTORE` cost 3.

In `for` loops, when using the length of a storage array as the condition being checked after each loop, caching the array length in memory can yield significant gas savings if the array length is high