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
#### Mistake in Code Description
- The correct value of type(int24).max should be 8388607 not 8388608 as corrected below
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L1389
```solidity
 ```solidity
 function getAccountPremium(
        address univ3pool,
        address owner,
        uint256 tokenType,
        int24 tickLower,
        int24 tickUpper,
        int24 atTick,
        uint256 isLong
    ) external view returns (uint128 premiumToken0, uint128 premiumToken1) {
        ...

---        // Compute the premium up to the current block (ie. after last touch until now). Do not proceed if atTick == type(int24).max = 8388608
+++        // Compute the premium up to the current block (ie. after last touch until now). Do not proceed if atTick == type(int24).max = 8388607
        if (atTick < type(int24).max) {
            ...
        }

        ...
    }
```
###  Report 4:
#### Absence of Callback Function and implementation when burning Liquidity
- It is necessary to have a callback implementation just as present during minting of Liquidity and Swapping to handle excesses, panoptic Protocol should add this implementation and design during burning of Liquidity.
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L1169-L1190
```solidity
function _burnLiquidity(
        uint256 liquidityChunk,
        IUniswapV3Pool univ3pool
    ) internal returns (int256 movedAmounts) {
        // burn that option's liquidity in the Uniswap Pool.
        // This will send the underlying tokens back to the Panoptic Pool (msg.sender)
        (uint256 amount0, uint256 amount1) = univ3pool.burn(
            liquidityChunk.tickLower(),
            liquidityChunk.tickUpper(),
            liquidityChunk.liquidity()
        );

        // amount0 The amount of token0 that was sent back to the Panoptic Pool
        // amount1 The amount of token1 that was sent back to the Panoptic Pool
        // no need to safecast to int from uint here as the max position size is int128
        // decrement the amountsOut with burnt amounts. amountsOut = notional value of tokens moved
        unchecked {
            movedAmounts = int256(0).toRightSlot(-int128(int256(amount0))).toLeftSlot(
                -int128(int256(amount1))
            );
        }
    }
```
###  Report 5:
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