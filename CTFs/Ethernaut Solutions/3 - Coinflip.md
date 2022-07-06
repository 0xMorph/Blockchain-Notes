## Overview
Goal is to get 10 consecutive wins.

Constructor sets `consecutiveWins` to zero.

Our main and only `flip` function takes in a `_guess` and returns a boolean.

`blockValue` will be the hash of the previous block number, shown through `block.number - 1`.

If `lashHash` is equal to `blockValue`, the transaction reverts, so every transaction would use a different `blockValue` than before