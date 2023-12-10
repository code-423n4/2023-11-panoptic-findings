## NatSpec Inconsistency in *initializeAMMPool* Function
https://github.com/code-423n4/2023-11-panoptic/blob/2647928c33be4a58883110befd7fd065448478ef/contracts/SemiFungiblePositionManager.sol#L346-356
According to the NatSpec, the function should revert if the Uniswap v3 pool is already initialized. However, the function currently causes a revert when the pool is not initialized.

## Potential Array Index Out-of-Bounds in *afterTokenTransfer* Function
https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L544-556
There is no explicit check to ensure that the *amounts* array is at least as long as the *ids* array. If *amounts.length* is less than *ids.length*, the contract will revert when the loop tries to access an undefined index in the *amounts* array.