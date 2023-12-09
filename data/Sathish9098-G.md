# GAS OPTIMIZATION

##

## [G-1] ``ReentrancyLock`` modifier can be more gas optimized 

Changing a storage value from 0 to a non-zero value (like 1) does indeed incur a higher gas cost (20,000 gas) compared to changing it from one non-zero value to another (5,000 gas). However, this higher cost is a one-time expense per storage slot when it first transitions from 0 to a non-zero value. Once a storage slot is set to a non-zero value, subsequent changes between non-zero values are less expensive.

``Initial Transition from 0 to 1``: The first time the locked field is set from 0 to 1, it will incur the higher gas cost. This happens only once per pool.

``Subsequent Toggles Between Non-Zero Values``: Once locked has been set to 1, subsequent toggles between 1 and another non-zero value (e.g., 2) will incur the lower cost.

This approach assumes that the reentrancy lock will be used multiple times over the lifecycle of each pool. The initial higher cost can be considered an investment that leads to savings in gas costs over time, as each subsequent lock and unlock operation will be cheaper

```diff
FILE: 2023-11-panoptic/contracts/SemiFungiblePositionManager.sol

 struct PoolAddressAndLock {
        IUniswapV3Pool pool;
-        bool locked;
+       uint32 locked;
    }


- /// @notice Modifier that prohibits reentrant calls for a specific pool
-    /// @dev We piggyback the reentrancy lock on the (pool id => pool) mapping to save gas
-    /// @dev (there's an extra 96 bits of storage available in the mapping slot and it's almost always warm)
-    /// @param poolId The poolId of the pool to activate the reentrancy lock on
-    modifier ReentrancyLock(uint64 poolId) {
-        // check if the pool is already locked
-        // init lock if not
-        beginReentrancyLock(poolId);
-
-        // execute function
-        _;
-
-        // remove lock
-        endReentrancyLock(poolId);
-    }
-
-    /// @notice Add reentrancy lock on pool
-    /// @dev reverts if the pool is already locked
-    /// @param poolId The poolId of the pool to add the reentrancy lock to
-    function beginReentrancyLock(uint64 poolId) internal {
-        // check if the pool is already locked, if so, revert
-        if (s_poolContext[poolId].locked) revert Errors.ReentrantCall();
-
-        // activate lock
-        s_poolContext[poolId].locked = true;
-    }
-
-    /// @notice Remove reentrancy lock on pool
-    /// @param poolId The poolId of the pool to remove the reentrancy lock from
-    function endReentrancyLock(uint64 poolId) internal {
-        // gas refund is triggered here by returning the slot to its original value
-        s_poolContext[poolId].locked = false;
-    }

+ modifier ReentrancyLock(uint64 poolId) {
+    // Inline the logic of beginReentrancyLock to save on function call overhead
+    // Check if the pool is already locked, if so, revert
+    if (s_poolContext[poolId].locked != 1) { // Assuming 'locked' is initialized to 1
+        s_poolContext[poolId].locked = 2; // Use different non-zero values to toggle
+    } else {
+        revert Errors.ReentrantCall();
+    }
+
+    _; // Execute function
+
+    // Inline the logic of endReentrancyLock
+    s_poolContext[poolId].locked = 1; // Toggle back
+  }

```
https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L302-L334

##

## [G-2]  ``totalLiquidity * 2 ** 64`` , ``netLiquidity ** 2`` , ``totalLiquidity ** 2`` repeated calculations results should be cached 

Caching the results of repeated calculations like ``totalLiquidity * 2 ** 64``, ``netLiquidity ** 2``, and ``totalLiquidity ** 2`` in a Solidity contract can be a good practice for optimizing gas usage. This will approximately saves ``1000 - 1500 GAS`` 

```diff
FILE: 2023-11-panoptic/contracts/SemiFungiblePositionManager.sol

{
                uint128 collected0 = uint128(collectedAmounts.rightSlot());
                uint128 collected1 = uint128(collectedAmounts.leftSlot());
                
+               uint256 totalLiquidityResult_  = totalLiquidity * 2 ** 64 ;
+               uint256 netLiquidity_ = netLiquidity ** 2 ; 

                // compute the base premium as collected * total / net^2 (from Eqn 3)
                premium0X64_base = Math
-                    .mulDiv(collected0, totalLiquidity * 2 ** 64, netLiquidity ** 2)
+                    .mulDiv(collected0, totalLiquidityResult_ , netLiquidity_ )
                    .toUint128();
                premium1X64_base = Math
-                    .mulDiv(collected1, totalLiquidity * 2 ** 64, netLiquidity ** 2)
+                    .mulDiv(collected1, totalLiquidityResult_, netLiquidity_)
                    .toUint128();
         }


 {

+              uint256  totalLiquidity_ = totalLiquidity ** 2 ;
                    // compute the gross premium (from Eqn 4)
-                    uint256 numerator = totalLiquidity ** 2 -
+                    uint256 numerator = totalLiquidity_  -
                        totalLiquidity *
                        removedLiquidity +
                        ((removedLiquidity ** 2) / 2 ** (VEGOID));
                    premium0X64_gross = Math
-                        .mulDiv(premium0X64_base, numerator, totalLiquidity ** 2)
+                        .mulDiv(premium0X64_base, numerator, totalLiquidity_)
                        .toUint128();
                    premium1X64_gross = Math
-                        .mulDiv(premium1X64_base, numerator, totalLiquidity ** 2)
+                        .mulDiv(premium1X64_base, numerator, totalLiquidity_)
                        .toUint128();
                    deltaPremiumGross = uint256(0).toRightSlot(premium0X64_gross).toLeftSlot(
                        premium1X64_gross
                    );
                }

```
https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1281-L1286


## [G-3] Caching ``univ3pool.tickSpacing()`` to eliminate repeated external calls in loop iterations 

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

## [G-4] Caching Repeated ``liquidityChunk.tickLower()`` and ``liquidityChunk.tickUpper()``function Calls 

Caching values like liquidityChunk.tickLower() and liquidityChunk.tickUpper() inside the loop can significantly improve gas efficiency, especially these methods are called multiple times within the loop. This optimization reduces the number of function calls, each of which consumes gas.

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

## [G-5] ``amount0Delta > 0 `` check should be cached

 ``amount0Delta > 0`` result should be cached with memory variable instead of checking multiple times 

```diff
FILE: 2023-11-panoptic/contracts/SemiFungiblePositionManager.sol

+  bool amount0Delta_ = amount0Delta > 0 ;
// Extract the address of the token to be sent (amount0 -> token0, amount1 -> token1)
-        address token = amount0Delta > 0
+        address token = amount0Delta_ 
            ? address(decoded.poolFeatures.token0)
            : address(decoded.poolFeatures.token1);

        // Transform the amount to pay to uint256 (take positive one from amount0 and amount1)
-        uint256 amountToPay = amount0Delta > 0 ? uint256(amount0Delta) : 
+        uint256 amountToPay = amount0Delta_  ? uint256(amount0Delta) : uint256(amount1Delta);

```
https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L451-L457

##

## [G-6] Use bitwise operations directly instead of calling these functions separately

The ``getPoolId()`` functions can be optimized by using bitwise operations directly in the function that calls them, instead of calling these functions separately. This will save the gas used for function calls.

This saves ``200-400 gas`` per function call (depends on the EVM version and gas schedule)

```diff
FILE: 2023-11-panoptic/contracts/SemiFungiblePositionManager.sol

- 368:   uint64 poolId = PanopticMath.getPoolId(univ3pool);

+        uint64 poolId= uint64(uint160(univ3pool) >> 96)

```
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L368

##

## [G-7] Memory variables should created outside of loop 

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

##

## [G-8] Don't cache memory variables 

Function parameters in Solidity are stored in memory, not in contract storage.Assigning the parameter univ3pool to another memory variable _univ3pool doesn't fundamentally change how the data is accessed or stored. It's still a memory-to-memory operation, which is relatively low in gas cost. The additional assignment might even add a small overhead.

```solidity
FILE: 2023-11-panoptic/contracts/SemiFungiblePositionManager.sol

752: IUniswapV3Pool _univ3pool = univ3pool;

867:  IUniswapV3Pool _univ3pool = univ3pool;

``` 
https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L752

##

## [G-9] ``movedInLeg.rightSlot()`` and ``movedInLeg.leftSlot()`` functions should be cached  

function calls can have a non-negligible gas cost, especially if they are external calls or if they read from contract storage. If ``movedInLeg.rightSlot()`` and ``movedInLeg.leftSlot()`` are either external calls or involve reading from storage, and they are called multiple times, caching their results in a local variable could save gas.

```diff
FILE: 2023-11-panoptic/contracts/SemiFungiblePositionManager.sol

 // moved will be negative if the leg was long (funds left the caller, don't count it in collected fees)
            uint128 collected0;
            uint128 collected1;
+      int128 rightSlot_ = movedInLeg.rightSlot();
+      int128 leftSlot_ = movedInLeg.leftSlot();
            unchecked {
-                collected0 = movedInLeg.rightSlot() < 0
+                collected0 = rightSlot_ < 0
-                    ? receivedAmount0 - uint128(-movedInLeg.rightSlot())
+                    ? receivedAmount0 - uint128(-rightSlot_)
                    : receivedAmount0;
-                collected1 = movedInLeg.leftSlot() < 0
+                collected1 = leftSlot_  < 0
-                    ? receivedAmount1 - uint128(-movedInLeg.leftSlot())
+                    ? receivedAmount1 - uint128(-leftSlot_)
                    : receivedAmount1;
            }

```
https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1230-L1240

##

## [G-10] Optimizing the ``add()`` function for enhanced gas efficiency 

- The addition of left and right slots is performed directly in int128. This approach minimizes the number of type conversions and uses the native type of the slots
- The overflow and underflow checks are simplified. The conditions check if the sign of the sum is different from what it should be if no overflow or underflow occurred. This is a more efficient way to perform these checks
- The final result z is constructed by shifting the left sum (leftSum) to the left by 128 bits and then combining it with the right sum (rightSum) using a bitwise OR operation. This approach avoids additional function calls and directly manipulates the bits

```diff
FILE: 2023-11-panoptic/contracts/types/LeftRight.sol

- function add(int256 x, int256 y) internal pure returns (int256 z) {
-        unchecked {
-            int256 left256 = int256(x.leftSlot()) + y.leftSlot();
-            int128 left128 = int128(left256);
-
-            int256 right256 = int256(x.rightSlot()) + y.rightSlot();
-            int128 right128 = int128(right256);
-
-            if (left128 != left256 || right128 != right256) revert Errors.UnderOverFlow();
-
-            return z.toRightSlot(right128).toLeftSlot(left128);
-        }
-    }

+ function add(int256 x, int256 y) internal pure returns (int256 z) {
+    unchecked {
+        int128 leftSum = int128(x.leftSlot()) + int128(y.leftSlot());
+        int128 rightSum = int128(x.rightSlot()) + int128(y.rightSlot());
+
+        // Overflow/underflow checks
+        if ((leftSum < x.leftSlot()) != (y.leftSlot() < 0) || 
+            (rightSum < x.rightSlot()) != (y.rightSlot() < 0)) {
+            revert Errors.UnderOverFlow();
+        }
+
+        // Combine left and right sums into one 256-bit integer
+        z = (int256(leftSum) << 128) | int256(uint128(rightSum));
+    }
+ }

```
https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/types/LeftRight.sol#L159-L171

##

## [G-11] Optimizing the ``sub()`` function for enhanced gas efficiency 

Same like add() function sub() also can be more gas optimized 

 
```diff
FILE: 2023-11-panoptic/contracts/types/LeftRight.sol

- function sub(int256 x, int256 y) internal pure returns (int256 z) {
-        unchecked {
-            int256 left256 = int256(x.leftSlot()) - y.leftSlot();
-            int128 left128 = int128(left256);
-
-            int256 right256 = int256(x.rightSlot()) - y.rightSlot();
-            int128 right128 = int128(right256);
-
-            if (left128 != left256 || right128 != right256) revert Errors.UnderOverFlow();
-
-            return z.toRightSlot(right128).toLeftSlot(left128);
-        }
-    }

+ function optimizedSub(int256 x, int256 y) internal pure returns (int256 z) {
+    unchecked {
+        int128 leftDifference = int128(x.leftSlot()) - int128(y.leftSlot());
+        int128 rightDifference = int128(x.rightSlot()) - int128(y.rightSlot());
+
+        // Overflow/underflow checks
+        if (int256(leftDifference) != x.leftSlot() - y.leftSlot() || 
+            int256(rightDifference) != x.rightSlot() - y.rightSlot()) {
+            revert Errors.UnderOverFlow();
+        }
+
+        // Combine left and right differences into one 256-bit integer
+        z = (int256(leftDifference) << 128) | int256(uint128(rightDifference));
+    }
+  }


```
https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/types/LeftRight.sol#L177-L189










