# [L-01] functions having same name
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

# [L-04] No check for address of uniswap or owner
## Context
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L1371
## Recommended 
Check the address of uniswap or owner are same or not , if same then revert 
and also check for 0 address

....
....
....


# [NC-1] Enormous natspec comments in a line
## Context
There are various lines in the protocol where letters exceed the threshold limit of letters
## Recommended 
Solidity suggests natspec comments to limit upto 120 letters in a line