# 1, PoolId can be generated infinitely without escape lead to out of gas for user

## Lines of code
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L370-#L372
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/PanopticMath.sol#L48-#L59

## Details
In `initializeAMMPool`, to avoid Id collision, function will check if previous ID was existed before or not:

        while (address(s_poolContext[poolId].pool) != address(0)) {
            poolId = PanopticMath.getFinalPoolId(poolId, token0, token1, fee);
        }
This is function to generate new poolId:

    function getFinalPoolId(
        uint64 basePoolId,
        address token0,
        address token1,
        uint24 fee
    ) internal pure returns (uint64) {
        unchecked {
            return
                basePoolId +
                (uint64(uint256(keccak256(abi.encodePacked(token0, token1, fee)))) >> 32);
        }
    }
But problem is if new poolId generated is same as old one, while loop is still true and it will keep generating same poolId till user run out of gas, and the pool will never be initialized

## Impact
User will run out of gas because of `while` condition loop infinity. Submited as low because chance that it can be happened is very low.

## Recommended Mitigation Steps
`poolId` should be selected by user in acceptable range.



# 2, User failed minting/borrowing if the new tick reaches boundary

## Lines of code
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L713

## Details
Developer noted in `mintTokenizedPosition` and `burnTokenizedPosition` about two slippage parameter:

    /// @param slippageTickLimitLow The lower price slippage limit when minting an ITM position (set to larger than slippageTickLimitHigh for swapping when minting)
    /// @param slippageTickLimitHigh The higher slippage limit when minting an ITM position (set to lower than slippageTickLimitLow for swapping when minting)

And in the code:

        if ((newTick >= tickLimitHigh) || (newTick <= tickLimitLow)) revert Errors.PriceBoundFail();

It can see that new tick is not accepted if it reaches boundary, it is not true compare to what developer noted in function. If the new tick equal to lower or higher tick, transaction will be reverted

## Impact
User transaction can be unintentionally reverted.

## Recommended Mitigation Steps
Checking condition should be changed to:

        if ((newTick > tickLimitHigh) || (newTick < tickLimitLow)) revert Errors.PriceBoundFail();