# PANOPTIC GAS OPTIMIZATIONS


## INTRODUCTION
Highlighted below are optimizations exclusively targeting state-mutating functions and view/pure functions invoked by state-mutating functions. In the discussion that follows, only runtime gas is emphasized, given its inevitable dominance over deployment gas costs throughout the protocol's lifetime. 

Please be aware that some code snippets may be shortened to conserve space, and certain code snippets may include @audit tags in comments to facilitate issue explanations."



## [G-01] Cache external calls outside of loop to avoid re-calling function on each iteration
Performing external calls that do not depend on variables incremented in loops should always try to be avoided within the loop. In the following instances, we are able to cache the external calls outside of the loop to save a STATICCALL (100 gas) per loop iteration.

### 2 Instances
1. #### Cache `univ3pool.tickSpacing()` outside of loop.
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L590

The external call `univ3pool.tickSpacing()` should be made outside the loop and the result cached since the value it returns is not dependent on the loop iterations.

```solidity
file: contracts/SemiFungiblePositionManager.sol

578:    function registerTokenTransfer(address from, address to, uint256 id, uint256 amount) internal {
579:        // Extract univ3pool from the poolId map to Uniswap Pool
580:        IUniswapV3Pool univ3pool = s_poolContext[id.validate()].pool;
581:
582:        uint256 numLegs = id.countLegs();
583:        for (uint256 leg = 0; leg < numLegs; ) {
584:            // for this leg index: extract the liquidity chunk: a 256bit word containing the liquidity amount and upper/lower tick
585:            // @dev see `contracts/types/LiquidityChunk.sol`
586:            uint256 liquidityChunk = PanopticMath.getLiquidityChunk(
587:                id,
588:                leg,
589:                uint128(amount),
590:                univ3pool.tickSpacing()     //@audit cache external call outside of loop
591:            );
.
.
.
635:    }
```

```diff
diff --git a/contracts/SemiFungiblePositionManager.sol b/contracts/SemiFungiblePositionManager.sol
index 0ea05b4..5766620 100644
--- a/contracts/SemiFungiblePositionManager.sol
+++ b/contracts/SemiFungiblePositionManager.sol
@@ -578,6 +578,7 @@ contract SemiFungiblePositionManager is ERC1155, Multicall {
     function registerTokenTransfer(address from, address to, uint256 id, uint256 amount) internal {
         // Extract univ3pool from the poolId map to Uniswap Pool
         IUniswapV3Pool univ3pool = s_poolContext[id.validate()].pool;
+        int24 tickSpacing = univ3pool.tickSpacing();

         uint256 numLegs = id.countLegs();
         for (uint256 leg = 0; leg < numLegs; ) {
@@ -587,7 +588,7 @@ contract SemiFungiblePositionManager is ERC1155, Multicall {
                 id,
                 leg,
                 uint128(amount),
-                univ3pool.tickSpacing()
+                tickSpacing
             );

             //construct the positionKey for the from and to addresses
```
```
Estimated Gas saved: 97 gas units per loop iteration
```


2. #### Cache `univ3pool.tickSpacing()` outside of loop.
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L886

The external call `univ3pool.tickSpacing()` should be made outside the loop and the result cached since the value it returns is not dependent on the loop iterations.

```solidity
file: contracts/SemiFungiblePositionManager.sol

848:    function _createPositionInAMM(
849:        IUniswapV3Pool univ3pool,
850:        uint256 tokenId,
851:        uint128 positionSize,
852:        bool isBurn
853:    ) internal returns (int256 totalMoved, int256 totalCollected, int256 itmAmounts) {
854:        // upper bound on amount of tokens contained across all legs of the position at any given tick
855:        uint256 amount0;
856:        uint256 amount1;
857:
858:        uint256 numLegs = tokenId.countLegs();
859:        // loop through up to the 4 potential legs in the tokenId
860:        for (uint256 leg = 0; leg < numLegs; ) {
861:            int256 _moved;
862:            int256 _itmAmounts;
863:            int256 _totalCollected;
864:
865:            {
866:                // cache the univ3pool, tokenId, isBurn, and _positionSize variables to get rid of stack too deep error
867:                IUniswapV3Pool _univ3pool = univ3pool;
868:                uint256 _tokenId = tokenId;
869:                bool _isBurn = isBurn;
870:                uint128 _positionSize = positionSize;
871:                uint256 _leg;
872:
873:                unchecked {
874:                    // Reverse the order of the legs if this call is burning a position (LIFO)
875:                    // We loop in reverse order if burning a position so that any dependent long liquidity is returned to the pool first,
876:                    // allowing the corresponding short liquidity to be removed
877:                    _leg = _isBurn ? numLegs - leg - 1 : leg;
878:                }
879:
880:                // for this _leg index: extract the liquidity chunk: a 256bit word containing the liquidity amount and upper/lower tick
881:                // @dev see `contracts/types/LiquidityChunk.sol`
882:                uint256 liquidityChunk = PanopticMath.getLiquidityChunk(
883:                    _tokenId,
884:                    _leg,
885:                    _positionSize,
886:                    _univ3pool.tickSpacing()    //@audit cache external call outside of loop
887:                );
.
.
.
919:    }
```

```diff
diff --git a/contracts/SemiFungiblePositionManager.sol b/contracts/SemiFungiblePositionManager.sol
index 0ea05b4..cfb3c2f 100644
--- a/contracts/SemiFungiblePositionManager.sol
+++ b/contracts/SemiFungiblePositionManager.sol
@@ -877,13 +877,15 @@ contract SemiFungiblePositionManager is ERC1155, Multicall {
                     _leg = _isBurn ? numLegs - leg - 1 : leg;
                 }

+                int24 tickSpacing = _univ3pool.tickSpacing();
+
                 // for this _leg index: extract the liquidity chunk: a 256bit word containing the liquidity amount and upper/lower tick
                 // @dev see `contracts/types/LiquidityChunk.sol`
                 uint256 liquidityChunk = PanopticMath.getLiquidityChunk(
                     _tokenId,
                     _leg,
                     _positionSize,
-                    _univ3pool.tickSpacing()
+                    tickSpacing
                 );

                 (_moved, _itmAmounts, _totalCollected) = _createLegInAMM(
```
```
Estimated Gas saved: 97 gas units per loop iteration
```




## [G-02] Computations should be memoized rather than having to re-compute them
In computing, memoization or memoisation is an optimization technique used primarily to speed up computer programs by storing the results of expensive function calls to pure functions and returning the cached result when the same inputs occur again.

### 2 Instances
1. #### We can memoize the computations of `id.tokenType(leg)`, `liquidityChunk.tickLower()` and `liquidityChunk.tickUpper()`.
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L598-#L600

We can reduce the gas usage of the `registerTokenTransfer()` function by memoizing the computations of `id.tokenType(leg)`, `liquidityChunk.tickLower()` and `liquidityChunk.tickUpper()` i.e cache the results of their computations so that for subsequent computations we can use the cached values rather than having to recompute their values. The diff below shows how the code could be refactored:

```solidity
file: contracts/SemiFungiblePositionManager.sol

578:    function registerTokenTransfer(address from, address to, uint256 id, uint256 amount) internal {
.
.
.
593:            //construct the positionKey for the from and to addresses
594:            bytes32 positionKey_from = keccak256(
595:                abi.encodePacked(
596:                    address(univ3pool),
597:                    from,
598:                    id.tokenType(leg),           //@audit memoize id.tokenType(leg)
599:                    liquidityChunk.tickLower(),  //@audit memoize liquidityChunk.tickLower()
600:                    liquidityChunk.tickUpper()   //@audit memoize liquidityChunk.tickUpper()
601:                )
602:            );
603:            bytes32 positionKey_to = keccak256(
604:                abi.encodePacked(
605:                    address(univ3pool),
606:                    to,
607:                    id.tokenType(leg),
609:                    liquidityChunk.tickLower(),
610:                    liquidityChunk.tickUpper()
611:                )
612:            );
.
.
.
635:    }
```

```diff
diff --git a/contracts/SemiFungiblePositionManager.sol b/contracts/SemiFungiblePositionManager.sol    
index 0ea05b4..6586d72 100644                                                                         
--- a/contracts/SemiFungiblePositionManager.sol                                                       
+++ b/contracts/SemiFungiblePositionManager.sol                                                       
@@ -590,23 +590,27 @@ contract SemiFungiblePositionManager is ERC1155, Multicall {                    
                 univ3pool.tickSpacing()                                                              
             );                                                                                       
                                                                                                      
+            uint256 idTokenType = id.tokenType(leg);                                                 
+            int24 liquidityChunkTickLower = liquidityChunk.tickLower();                              
+            int liquidityChunkTickUpper = liquidityChunk.tickUpper();                                
+                                                                                                     
             //construct the positionKey for the from and to addresses                                
             bytes32 positionKey_from = keccak256(                                                    
                 abi.encodePacked(                                                                    
                     address(univ3pool),                                                              
                     from,                                                                            
-                    id.tokenType(leg),                                                               
-                    liquidityChunk.tickLower(),                                                      
-                    liquidityChunk.tickUpper()                                                       
+                    idTokenType,                                                                     
+                    liquidityChunkTickLower,                                                         
+                    liquidityChunkTickUpper                                                          
                 )                                                                                    
             );                                                                                       
             bytes32 positionKey_to = keccak256(                                                      
                 abi.encodePacked(                                                                    
                     address(univ3pool),                                                              
                     to,                                                                              
-                    id.tokenType(leg),                                                               
-                    liquidityChunk.tickLower(),                                                      
-                    liquidityChunk.tickUpper()                                                       
+                    idTokenType,                                                                     
+                    liquidityChunkTickLower,                                                         
+                    liquidityChunkTickUpper                                                          
                 )                                                                                    
             );                                                                                       
```


2. #### We can memoize the computations of `self.strike(i)`.
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L483

We can reduce the gas usage of the `validate()` function by memoizing the computation of `self.strike(i)` i.e cache the result of the computations so that for subsequent computations we can use the cached value rather than having to recompute the value. The diff below shows how the code could be refactored:

```solidity
file: contracts/types/TokenId.sol

463:    function validate(uint256 self) internal pure returns (uint64) {
.
.
.
483:                // Strike cannot be MIN_TICK or MAX_TICK
484:                if (
485:                    (self.strike(i) == Constants.MIN_V3POOL_TICK) ||    
486:                    (self.strike(i) == Constants.MAX_V3POOL_TICK)
487:                ) revert Errors.InvalidTokenIdParameter(4);
488:
489:                // In the following, we check whether the risk partner of this leg is itself
490:                // or another leg in this position.
491:                // Handles case where riskPartner(i) != i ==> leg i has a risk partner that is another leg
.
.
.
524:    }
```

```diff
diff --git a/contracts/types/TokenId.sol b/contracts/types/TokenId.sol
index 6f3517c..9156725 100644
--- a/contracts/types/TokenId.sol
+++ b/contracts/types/TokenId.sol
@@ -478,10 +478,12 @@ library TokenId {

                 // The width cannot be 0; the minimum is 1
                 if ((self.width(i) == 0)) revert Errors.InvalidTokenIdParameter(5);
+
+                int24 selfStrike = self.strike(i);
                 // Strike cannot be MIN_TICK or MAX_TICK
                 if (
-                    (self.strike(i) == Constants.MIN_V3POOL_TICK) ||
-                    (self.strike(i) == Constants.MAX_V3POOL_TICK)
+                    selfStrike == Constants.MIN_V3POOL_TICK) ||
+                    selfStrike == Constants.MAX_V3POOL_TICK)
                 ) revert Errors.InvalidTokenIdParameter(4);

                 // In the following, we check whether the risk partner of this leg is itself
```




## [G-03] Refactor `SemiFungiblePositionManager.uniswapV3SwapCallback()` function to avoid performing `amount0Delta > 0` conditional twice.

- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L452-#L457

In the `uniswapV3SwapCallback` function rather than having to check if `amount0Delta` is greater than `0` using twice in the function we can refactor the code so that this check is done once thereby saving gas that would have been used in the second check. The diff below shows how the code could be refactored:

```solidity
file: contracts/SemiFungiblePositionManager.sol

441:    function uniswapV3SwapCallback(
442:        int256 amount0Delta,
443:        int256 amount1Delta,
444:        bytes calldata data
445:    ) external {
446:        // Decode the swap callback data, checks that the UniswapV3Pool has the correct address.
447:        CallbackLib.CallbackData memory decoded = abi.decode(data, (CallbackLib.CallbackData));
448:        // Validate caller to ensure we got called from the AMM pool
449:        CallbackLib.validateCallback(msg.sender, address(FACTORY), decoded.poolFeatures);
450:
451:        // Extract the address of the token to be sent (amount0 -> token0, amount1 -> token1)
452:        address token = amount0Delta > 0
453:            ? address(decoded.poolFeatures.token0)
454:            : address(decoded.poolFeatures.token1);
455:
456:        // Transform the amount to pay to uint256 (take positive one from amount0 and amount1)
457:        uint256 amountToPay = amount0Delta > 0 ? uint256(amount0Delta) : uint256(amount1Delta);
458:
459:        // Pay the required token from the payer to the caller of this contract
460:        SafeTransferLib.safeTransferFrom(token, decoded.payer, msg.sender, amountToPay);
461:    }
```

```diff
diff --git a/contracts/SemiFungiblePositionManager.sol b/contracts/SemiFungiblePositionManager.sol
index 0ea05b4..c26021a 100644
--- a/contracts/SemiFungiblePositionManager.sol
+++ b/contracts/SemiFungiblePositionManager.sol
@@ -449,12 +449,18 @@ contract SemiFungiblePositionManager is ERC1155, Multicall {
         CallbackLib.validateCallback(msg.sender, address(FACTORY), decoded.poolFeatures);

         // Extract the address of the token to be sent (amount0 -> token0, amount1 -> token1)
-        address token = amount0Delta > 0
-            ? address(decoded.poolFeatures.token0)
-            : address(decoded.poolFeatures.token1);
-
         // Transform the amount to pay to uint256 (take positive one from amount0 and amount1)
-        uint256 amountToPay = amount0Delta > 0 ? uint256(amount0Delta) : uint256(amount1Delta);
+
+        address token;
+        uint256 amountToPay;
+        if(amount0Delta > 0) {
+            token = address(decoded.poolFeatures.token0);
+            amountToPay = uint256(amount0Delta);
+        } else {
+            token = address(decoded.poolFeatures.token1);
+            amountToPay = uint256(amount1Delta);
+        }
+

         // Pay the required token from the payer to the caller of this contract
         SafeTransferLib.safeTransferFrom(token, decoded.payer, msg.sender, amountToPay);
```




## [G-04] Multiple accesses of a mapping should use a local variable cache
The instances below point to the second+ access of a value inside a mapping, within a function. Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations. Caching an array's struct avoids recalculating the array offsets into memory/calldata.
#### Please note this instance was not included in the bots report.

### 1 Instance
1. #### Cache `s_poolContext[poolId]` into a local storage variable.
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L370-#L378
```solidity
file: contracts/SemiFungiblePositionManager.sol

351:    function initializeAMMPool(address token0, address token1, uint24 fee) external {
.
.
.
370:        while (address(s_poolContext[poolId].pool) != address(0)) {     //@audit cache s_poolContext[poolId]
371:            poolId = PanopticMath.getFinalPoolId(poolId, token0, token1, fee);
372:        }
373:        // store the poolId => UniswapV3Pool information in a mapping
374:        // `locked` can be initialized to false because the pool address makes the slot nonzero
375:        s_poolContext[poolId] = PoolAddressAndLock({
376:            pool: IUniswapV3Pool(univ3pool),
377:            locked: false
378:        });
.
.
.
396:    }
```

```diff
diff --git a/contracts/SemiFungiblePositionManager.sol b/contracts/SemiFungiblePositionManager.sol              
index 0ea05b4..3f6a9a3 100644                                                                                   
--- a/contracts/SemiFungiblePositionManager.sol                                                                 
+++ b/contracts/SemiFungiblePositionManager.sol                                                                 
@@ -367,15 +367,17 @@ contract SemiFungiblePositionManager is ERC1155, Multicall {                              
         // @dev increase the poolId by a pseudo-random number                                                  
         uint64 poolId = PanopticMath.getPoolId(univ3pool);                                                     
                                                                                                                
-        while (address(s_poolContext[poolId].pool) != address(0)) {                                            
+        PoolAddressAndLock storage  contextData = s_poolContext[poolId];                                       
+                                                                                                               
+        while (address(contextData.pool) != address(0)) {                                                      
             poolId = PanopticMath.getFinalPoolId(poolId, token0, token1, fee);                                 
         }                                                                                                      
         // store the poolId => UniswapV3Pool information in a mapping                                          
         // `locked` can be initialized to false because the pool address makes the slot nonzero                
-        s_poolContext[poolId] = PoolAddressAndLock({                                                           
-            pool: IUniswapV3Pool(univ3pool),                                                                   
-            locked: false                                                                                      
-        });                                                                                                    
+        contextData.pool = IUniswapV3Pool(univ3pool);                                                          
+        contextData.locked = false;                                                                            
+                                                                                                               
+                                                                                                               
                                                                                                                
         // store the UniswapV3Pool => poolId information in a mapping                                          
         // add a bit on the end to indicate that the pool is initialized                                       
```
```
Estimated gas saved: 40 gas units
```




## [G-05] Use custom errors instead of require
Consider the use of a custom error, as it leads to a cheaper deploy cost and run time cost. The run time cost is only relevant when the revert condition is met. For more explanation read [Solidity's Documentation](https://docs.soliditylang.org/en/v0.8.22/control-structures.html#revert)


### Instances
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L207
```solidity
file: contracts/libraries/Math.sol

186:    function mulDiv(
187:        uint256 a,
188:        uint256 b,
189:        uint256 denominator
190:    ) internal pure returns (uint256 result) {
.
.
.
205:            // Handle non-overflow cases, 256 by 256 division
206:            if (prod1 == 0) {
207:                require(denominator > 0);  //@audit use custom error instead
208:                assembly ("memory-safe") {
209:                    result := div(prod0, denominator)
210:                }
211:                return result;
212:            }
.
.
.
280    }
```

- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L216
```solidity
file: contracts/libraries/Math.sol

186:    function mulDiv(
187:        uint256 a,
188:        uint256 b,
189:        uint256 denominator
190:    ) internal pure returns (uint256 result) {
.
.
.
214:            // Make sure the result is less than 2**256.
215:            // Also prevents denominator == 0
216:            require(denominator > prod1);  //@audit use custom error instead
217:
218:            ///////////////////////////////////////////////
219:            // 512 by 256 division.
220            ///////////////////////////////////////////////
.
.
.
280:    }
```

- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L311
```solidity
file: contracts/libraries/Math.sol


286:    function mulDiv64(uint256 a, uint256 b) internal pure returns (uint256 result) {
.
.
.
310:            // Make sure the result is less than 2**256.
311:            require(2 ** 64 > prod1);  //@audit use custom error instead
312:
313:            ///////////////////////////////////////////////
314:            // 512 by 256 division.
315:            ///////////////////////////////////////////////
.
.
.
342:    }
```

- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L373
```solidity
file: contracts/libraries/Math.sol

348:    function mulDiv96(uint256 a, uint256 b) internal pure returns (uint256 result) {
.
.
.
372:            // Make sure the result is less than 2**256.
373:            require(2 ** 96 > prod1);  //@audit use custom error instead
374:
375:            ///////////////////////////////////////////////
376:            // 512 by 256 division.
377:            ///////////////////////////////////////////////
.
.
.
404:    }
```

- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L435
```solidity
file: contracts/libraries/Math.sol

410:    function mulDiv128(uint256 a, uint256 b) internal pure returns (uint256 result) {
.
.
.
434:            // Make sure the result is less than 2**256.
435:            require(2 ** 128 > prod1);  //@audit use custom error instead
436:
437:            ///////////////////////////////////////////////
438:            // 512 by 256 division.
439:            ///////////////////////////////////////////////
.
.
.
466:    }
```

- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L497
```solidity
file: contracts/libraries/Math.sol

472:    function mulDiv192(uint256 a, uint256 b) internal pure returns (uint256 result) {
.
.
.
496:            // Make sure the result is less than 2**256.
497:            require(2 ** 192 > prod1);  //@audit use custom error instead
498:
499:            ///////////////////////////////////////////////
500:            // 512 by 256 division.
501:            ///////////////////////////////////////////////
.
.
.
528:    }
```





## [G-06]  Use named returns for local variables of pure functions where it is possible 

### Proof of Concept
```solidity
library NoNamedReturnArithmetic {
    
    function sum(uint256 num1, uint256 num2) internal pure returns(uint256){
        return num1 + num2;
    }
}

contract NoNamedReturn {
    using NoNamedReturnArithmetic for uint256;

    uint256 public stateVar;

    function add2State(uint256 num) public {
        stateVar = stateVar.sum(num);
    }
}
```
```solidity
test for test/NoNamedReturn.t.sol:NamedReturnTest
[PASS] test_Increment() (gas: 27639)
```

```solidity
library NamedReturnArithmetic {
    
    function sum(uint256 num1, uint256 num2) internal pure returns(uint256 theSum){
        theSum = num1 + num2;
    }
}

contract NamedReturn {
    using NamedReturnArithmetic for uint256;

    uint256 public stateVar;

    function add2State(uint256 num) public {
        stateVar = stateVar.sum(num);
    }
}
```
```solidity
test for test/NamedReturn.t.sol:NamedReturnTest
[PASS] test_Increment() (gas: 27613)
```

### 42 Instances
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L25
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L32
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L44
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L54
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L65
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L75
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L89
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L108
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L118
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L212
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LiquidityChunk.sol#L68
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LiquidityChunk.sol#L78
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LiquidityChunk.sol#L88
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LiquidityChunk.sol#L98
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LiquidityChunk.sol#L112
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LiquidityChunk.sol#L121
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LiquidityChunk.sol#L130
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L80
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L93
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L103
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L123
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L139
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L149
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L160
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L173
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L193
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L208
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L224
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L238
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L252
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L266
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L280
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L327
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L361
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L410
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L442
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L463
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L23
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/PanopticMath.sol#L38
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/PanopticMath.sol#L53
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/PanopticMath.sol#L145
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/PanopticMath.sol#L168





## [G-07] memory variable should created outside of loop
memory variable should created outside of loop, and get overriden with each iteration of loop, By doing so we save gas cost for memory variable creation in each iteration.

### Proof of Concept
```solidity
contract VarInLoop {

    uint256[] numbers = [1,2,3,4,5];
    uint256 total;

    function double(uint256 num) public pure returns(uint256 ans) {
        ans = num * 2;
    }

    function doubleNumbers() public {
        uint256 len = numbers.length;

        for(uint256 i; i < len; ++i) {
            uint256 doubleNum =  double(numbers[i]);
            total = total + doubleNum;
        }
    }
}
```
```
test for test/VarInLoop.t.sol:VarInLoopTest
[PASS] test_doubleNumbers() (gas: 38185)
```


```
contract VarOutLoop {

    uint256[] numbers = [1,2,3,4,5];
    uint256 total;

    function double(uint256 num) public pure returns(uint256 ans) {
        ans = num * 2;
    }

    function doubleNumbers() public {
        uint256 len = numbers.length;

        uint256 doubleNum;
        for(uint256 i; i < len; ++i) {
            doubleNum =  double(numbers[i]);
            total = total + doubleNum;
        }
    }
}
```
```
test for test/VarOutLoop.t.sol:VarOutLoopTest
[PASS] test_doubleNumbers() (gas: 38170)
```


### Instances
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L586
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L594
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L603
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L620
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L623
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L861
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L862
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L863
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L867
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L868
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L869
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L870
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L871
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L882

- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L490
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L503
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L504
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L507
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L508
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/multicall/Multicall.sol#L15







## CONCLUSION
As you embark on incorporating the recommended optimizations, we want to emphasize the utmost importance of proceeding with vigilance and dedicating thorough efforts to comprehensive testing. It is of paramount significance to ensure that the proposed alterations do not inadvertently introduce fresh vulnerabilities, while also successfully achieving the anticipated enhancements in performance.

We strongly advise conducting a meticulous and exhaustive evaluation of the modifications made to the codebase. This rigorous scrutiny and exhaustive assessment will play a pivotal role in affirming both the security and efficacy of the refactored code. Your careful attention to detail, coupled with the implementation of a robust testing framework, will provide the necessary assurance that the refined code aligns with your security objectives and effectively fulfills the intended performance optimizations.










