# [L-01] functions having same name
https://github.com/code-423n4/2023-11-panoptic/blob/2647928c33be4a58883110befd7fd065448478ef/contracts/SemiFungiblePositionManager.sol#L544
https://github.com/code-423n4/2023-11-panoptic/blob/2647928c33be4a58883110befd7fd065448478ef/contracts/SemiFungiblePositionManager.sol#L563
 ## Recommended 
Change the name of one of the function

# [L-02] What if the operator is already approved before
If it is already approved then this line will fail
https://github.com/code-423n4/2023-11-panoptic/blob/2647928c33be4a58883110befd7fd065448478ef/contracts/tokens/ERC1155Minimal.sol#L78 
## Recommended 
Check if already approved

# [L-03] Remove reentrancy lock on pool check
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L331
## Recommended 
Check if already lock removed