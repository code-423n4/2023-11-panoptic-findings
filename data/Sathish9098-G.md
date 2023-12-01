# GAS OPTIMIZATION

##

## [G-1] Caching ``univ3pool.tickSpacing()`` to eliminate repeated external calls in loop iterations 

Assign the value from univ3pool.tickSpacing() to this variable before the loop begins.
Use this cached value inside the loop instead of calling the function repeatedly.

```diff
FILE: 2023-11-panoptic/contracts/SemiFungiblePositionManager.sol

uint256 numLegs = id.countLegs();
+ uint24 tickSpacing_ = univ3pool.tickSpacing() ;
        for (uint256 leg = 0; leg < numLegs; ) {
            // for this leg index: extract the liquidity chunk: a 256bit word containing the liquidity amount and upper/lower tick
            // @dev see `contracts/types/LiquidityChunk.sol`
            uint256 liquidityChunk = PanopticMath.getLiquidityChunk(
                id,
                leg,
                uint128(amount),
-                univ3pool.tickSpacing()
+                tickSpacing_
            );

         
```
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L590C40-L590C40

##

## [G-2] Caching Repeated ``liquidityChunk.tickLower()`` and ``liquidityChunk.tickUpper()``function Calls 

Caching values like liquidityChunk.tickLower() and liquidityChunk.tickUpper() inside the loop can significantly improve gas efficiency, especially if these methods are called multiple times within the loop. This optimization reduces the number of function calls, each of which consumes gas.

```diff
FILE: 2023-11-panoptic/contracts/SemiFungiblePositionManager.sol

 uint256 liquidityChunk = PanopticMath.getLiquidityChunk(
                id,
                leg,
                uint128(amount),
                univ3pool.tickSpacing()
            );

+        int24 tickLower_ = liquidityChunk.tickLower();
+        int24 tickUpper_ = liquidityChunk.tickUpper();
            //construct the positionKey for the from and to addresses
            bytes32 positionKey_from = keccak256(
                abi.encodePacked(
                    address(univ3pool),
                    from,
                    id.tokenType(leg),
-                    liquidityChunk.tickLower(),
+                    tickLower_
-                    liquidityChunk.tickUpper()
+                   tickUpper_ 
                )
            );
            bytes32 positionKey_to = keccak256(
                abi.encodePacked(
                    address(univ3pool),
                    to,
                    id.tokenType(leg),
-                   liquidityChunk.tickLower(),
+                   tickLower_
-                   liquidityChunk.tickUpper()
+                   tickUpper_ 
                )
            );

```
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L599-L609


