# Gas Optimization Report

## G-001 Redundant tick retrieval from liquidity chunks in amount to liquidity helpers

**2 instances**

- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L136
- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L155

File: `libraries/Math.sol`
```solidity
135:    function getLiquidityForAmount0(
136:        uint256 liquidityChunk,
137:        uint256 amount0
138:    ) internal pure returns (uint128 liquidity) {
139:        uint160 lowPriceX96 = getSqrtRatioAtTick(liquidityChunk.tickLower());
140:        uint160 highPriceX96 = getSqrtRatioAtTick(liquidityChunk.tickUpper());
.
.
.
154:  function getLiquidityForAmount1(
155:       uint256 liquidityChunk,
156:        uint256 amount1
157:    ) internal pure returns (uint128 liquidity) {
158:        uint160 lowPriceX96 = getSqrtRatioAtTick(liquidityChunk.tickLower());
159:        uint160 highPriceX96 = getSqrtRatioAtTick(liquidityChunk.tickUpper());
```

In these two functions, it is redundant to pass in `liquidityChunk` and re-retrieve the ticks. The ticks are already in the raw form in the only place these functions are called:

File: `PanopticMath.sol`: https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/PanopticMath.sol#L120C9-L130C10 
```solidity
120:   if (tokenId.asset(legIndex) == 0) {
121:         legLiquidity = Math.getLiquidityForAmount0(
122:              uint256(0).addTickLower(tickLower).addTickUpper(tickUpper),
123:              amount
124:         );
125:   } else {
126:          legLiquidity = Math.getLiquidityForAmount1(
127:               uint256(0).addTickLower(tickLower).addTickUpper(tickUpper),
128:               amount
129:           );
130:    }
```

**Resolution:** Modify the liquidity helpers to directly accept the ticks. Example shown below is only for one of the helpers. The resolution applies similarly for the other.

```diff
diff --git a/contracts/libraries/Math.sol b/contracts/libraries/Math.sol
index 3d9cbb1..b82ef54 100644
--- a/contracts/libraries/Math.sol
+++ b/contracts/libraries/Math.sol
@@ -133,11 +133,12 @@ library Math {
     /// @param amount0 The amount of token0
     /// @return liquidity The calculated amount of liquidity
     function getLiquidityForAmount0(
-        uint256 liquidityChunk,
+        int24 tickLower,
+        int24 tickUpper,
         uint256 amount0
     ) internal pure returns (uint128 liquidity) {
-        uint160 lowPriceX96 = getSqrtRatioAtTick(liquidityChunk.tickLower());
-        uint160 highPriceX96 = getSqrtRatioAtTick(liquidityChunk.tickUpper());
+        uint160 lowPriceX96 = getSqrtRatioAtTick(tickLower);
+        uint160 highPriceX96 = getSqrtRatioAtTick(tickUpper);
 
         unchecked {
             return
diff --git a/contracts/libraries/PanopticMath.sol b/contracts/libraries/PanopticMath.sol
index e3f9ce0..d1f3fcf 100644
--- a/contracts/libraries/PanopticMath.sol
+++ b/contracts/libraries/PanopticMath.sol
@@ -119,7 +119,8 @@ library PanopticMath {
         uint256 amount = uint256(positionSize) * tokenId.optionRatio(legIndex);
         if (tokenId.asset(legIndex) == 0) {
             legLiquidity = Math.getLiquidityForAmount0(
-                uint256(0).addTickLower(tickLower).addTickUpper(tickUpper),
+                tickLower,
+                tickUpper,
                 amount
             );
         } else {
```