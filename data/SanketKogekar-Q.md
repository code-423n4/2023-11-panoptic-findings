- Sponsors should note that `multicall` function receives all calldata for multiple calls in a single transaction. This means there is only one `msg.value` associated with the entire transaction, not one for each individual call within the function.

https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/multicall/Multicall.sol#L12-L36

- There is a chance for loss of precision in certain calculations like -

```solidity
uint256 numerator = totalLiquidity ** 2 -
                        totalLiquidity *
                        removedLiquidity +
                        ((removedLiquidity ** 2) / 2 ** (VEGOID));
```

https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1316

Use solady's 'FixedPointMathLib' to avoid dust in contract.