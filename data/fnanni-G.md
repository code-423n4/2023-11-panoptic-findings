1. Simplify ifs in TokenId.sol validate() (~70 gas per leg)

The code here https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/types/TokenId.sol#L507-L518 is equivalent to 
```
if ((isLong ^ isLongP) == (tokenType ^ tokenTypeP))
```

2. Avoid redundant riskPartner checks in TokenId.sol validate()

The code here https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/types/TokenId.sol#L490-L520 is executed twice per pair of riskPartnerIndexes, but only once is needed. Replacing L491 with `if (riskPartnerIndex > 1)` removes this inefficiency.

3. Optimize TokenId strike checks in validate()

Instead of parsing token id's strike parameter twice https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/types/TokenId.sol#L482-L485, use a local variable to store the parsed strike variable.

4. Remove unnecessary checks

Checking the positionSize https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L670-L671 doesn't seem needed, because Uniswap V3 already reverts on minting 0 and nothing out of the ordinary seems to happen when burning 0.

5. Remove address(0) check

Checking that the univ3pool is not null is redundant, because calling a null address as it were a Uniswap pool will revert anyways. https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L683 

6. Don't use liquidity chunks inside PanopticMath and Math to reduce the back and forth conversion of tickLower,tickUpper,liquidity <--> liquidityChunk

Example 1: https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L586-L621

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/PanopticMath.sol#L133 PanopticMath.getLiquidityChunk() returns a liquidity chunk but it would be better if it returned the 3 parameters individually. 7 shifts and several casting can be avoided.

Example 2: https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L101-L128

3 shifts and casts can be saved by avoiding liquidityChunks in Math.getAmount1ForLiquidity(). Same for Math.getAmount0ForLiquidity(). Here https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L882-L902 it might be reasonable to pass liquidityChunk to _createLegInAMM(), but createChunk should be invoked outside PanopticMath.getLiquidityChunk() only for that purpose.