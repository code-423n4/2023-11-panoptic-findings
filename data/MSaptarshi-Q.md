# [L-01]Compilations error functions having same name
## Context
https://github.com/code-423n4/2023-11-panoptic/blob/2647928c33be4a58883110befd7fd065448478ef/contracts/SemiFungiblePositionManager.sol#L544
https://github.com/code-423n4/2023-11-panoptic/blob/2647928c33be4a58883110befd7fd065448478ef/contracts/SemiFungiblePositionManager.sol#L563
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/LeftRight.sol#L444
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/LeftRight.sol#L25
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/LeftRight.sol#L89
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/LeftRight.sol#L108
 ## Recommended 
Change the name of one of the function

# [L-02] What if the operator is already approved before
## Context
If it is already approved then this line will fail
https://github.com/code-423n4/2023-11-panoptic/blob/2647928c33be4a58883110befd7fd065448478ef/contracts/tokens/ERC1155Minimal.sol#L78 
## Recommended 
Check if already approved

# [L-03] Remove reentrancy lock on pool check
## Context
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L331
## Recommended 
Check if already lock removed

# [L-04] No check for address of **uniswap** or owner
## Context
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L1371
## Recommended 
Check the address of **uniswap** or owner are same or not , if same then revert 
and also check for 0 address

# [L-05] Same function calling may not return the desired value
## Context
There are instances throughout the whole protocol codebase, that some functions are calling external functions, which may not return the same value as expected .
## Recommended 
Check the type of value we expect or change the function name

....
....
....


# [NC-1] Enormous natspec comments in a line
## Context
There are various lines in the protocol where letters exceed the threshold limit of letters
## Recommended 
Solidity suggests natspec comments to limit upto 120 letters in a line

# [NC-2] Different solidity compiler versions
## Context
There are various instances in the protocol where different contracts have different compiler versions which may result in compiling error.
https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/libraries/PanopticMath.sol#L2
https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L2
https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/tokens/ERC1155Minimal.sol#L2
## Recommended 
Solidity suggests using same compiler versions for every inherited or parent contract