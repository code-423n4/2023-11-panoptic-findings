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

##

## [G-3] Use bitwise operations directly instead of calling these functions separately

The ``getPoolId()`` functions can be optimized by using bitwise operations directly in the function that calls them, instead of calling these functions separately. This will save the gas used for function calls.

This saves ``200-400 gas`` per function call (depends on the EVM version and gas schedule)

```diff
FILE: 2023-11-panoptic/contracts/SemiFungiblePositionManager.sol

- 368:   uint64 poolId = PanopticMath.getPoolId(univ3pool);

+        uint64 poolId= uint64(uint160(univ3pool) >> 96)

```
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L368

##

## [G-4] memory variable should created outside of loop 

Declaring variables inside loops, especially when the loop iterates many times, can be inefficient. This is because each iteration of the loop creates new instances of these variables.

the variables _moved, _itmAmounts, and _totalCollected are declared inside the for loop but outside the inner block {...}. This is still within the loop's scope, so these variables are redeclared on each iteration. It might be more efficient to declare them outside of the loop.

The variables _univ3pool, _tokenId, _isBurn, _positionSize, and _leg are declared within an inner block. This is a common practice to avoid the "stack too deep" error in Solidity. These variables seem to be derivatives of state variables or function arguments, which are read once and then used multiple times in the loop.
If these variables are not expected to change during the loop iterations, declaring them outside the loop and before entering it could be more efficient. This way, you only read from state variables or compute their values once.

```solidity
FILE: 2023-11-panoptic/contracts/SemiFungiblePositionManager.sol

   for (uint256 leg = 0; leg < numLegs; ) {
            int256 _moved;
            int256 _itmAmounts;
            int256 _totalCollected;

            {
                // cache the univ3pool, tokenId, isBurn, and _positionSize variables to get rid of stack too deep error
                IUniswapV3Pool _univ3pool = univ3pool;
                uint256 _tokenId = tokenId;
                bool _isBurn = isBurn;
                uint128 _positionSize = positionSize;
                uint256 _leg;

                unchecked {
                    // Reverse the order of the legs if this call is burning a position (LIFO)
                    // We loop in reverse order if burning a position so that any dependent long liquidity is returned to the pool first,
                    // allowing the corresponding short liquidity to be removed
                    _leg = _isBurn ? numLegs - leg - 1 : leg;
                }

                // for this _leg index: extract the liquidity chunk: a 256bit word containing the liquidity amount and upper/lower tick
                // @dev see `contracts/types/LiquidityChunk.sol`
                uint256 liquidityChunk = PanopticMath.getLiquidityChunk(
                    _tokenId,
                    _leg,
                    _positionSize,
                    _univ3pool.tickSpacing()
                );

                (_moved, _itmAmounts, _totalCollected) = _createLegInAMM(
                    _univ3pool,
                    _tokenId,
                    _leg,
                    liquidityChunk,
                    _isBurn
                );

                unchecked {
                    // increment accumulators of the upper bound on tokens contained across all legs of the position at any given tick
                    amount0 += Math.getAmount0ForLiquidity(liquidityChunk);

                    amount1 += Math.getAmount1ForLiquidity(liquidityChunk);
                }
            }

            totalMoved = totalMoved.add(_moved);
            itmAmounts = itmAmounts.add(_itmAmounts);
            totalCollected = totalCollected.add(_totalCollected);

            unchecked {
                ++leg;
            }
        }

```
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L860-L912


Renetrancy guard optimizations


