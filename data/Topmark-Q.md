### Report 1:
#### Wrong Comment Description.
- The comment description for the getLiquidityForAmount1(...) function is wrong, it should be "token1" not "token0" as corrected below.
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L150
```solidity
---   /// @notice Calculates the amount of liquidity for a given amount of token0 and liquidityChunk
+++   /// @notice Calculates the amount of liquidity for a given amount of token1 and liquidityChunk
    /// @param liquidityChunk variable that efficiently packs the liquidity, tickLower, and tickUpper.
    /// @param amount1 The amount of token1
    /// @return liquidity The calculated amount of liquidity
    function getLiquidityForAmount1(
        uint256 liquidityChunk,
        uint256 amount1
    ) internal pure returns (uint128 liquidity) {
        uint160 lowPriceX96 = getSqrtRatioAtTick(liquidityChunk.tickLower());
        uint160 highPriceX96 = getSqrtRatioAtTick(liquidityChunk.tickUpper());
        unchecked {
            return toUint128(mulDiv(amount1, Constants.FP96, highPriceX96 - lowPriceX96));
        }
    }
```
###  Report 2:
#### Incomplete comment description in the getSqrtRatioAtTick(...) function.
- As noted in the code provided below, the last word in the comment description i.e "Uniswap's" shows there is a missing continuation, necessary continuation should be added to the description.
- Secondly the word "is" used in the comment is not correct, it should be "it" not "is"
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L88
```solidity
  function getSqrtRatioAtTick(int24 tick) internal pure returns (uint160 sqrtPriceX96) {
        ...

---            // Downcast + rounding up to keep is consistent with Uniswap's
---            // Downcast + rounding up to keep it consistent with Uniswap's ...
            sqrtPriceX96 = uint160((sqrtR >> 32) + (sqrtR % (1 << 32) == 0 ? 0 : 1));
        }
    }
```
###  Report 3:
#### Missing Validation
- Absence of validation to ensure positionSize to be burnt is not empty, necessary validation should be added as done in the code below.
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L487
```solidity
 function burnTokenizedPosition(
        uint256 tokenId,
        uint128 positionSize,
        int24 slippageTickLimitLow,
        int24 slippageTickLimitHigh
    )
        external
        ReentrancyLock(tokenId.univ3pool())
        returns (int256 totalCollected, int256 totalSwapped, int24 newTick)
    {
+++ require ( positionSize != 0 , "positionSize Error");
        // burn this ERC1155 token id
        _burn(msg.sender, tokenId, positionSize);
 ...
}
```