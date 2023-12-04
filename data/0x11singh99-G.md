## Gas Optimizations

| Number | Issue | Instances |
|-|:-|:-:|
| [[G-01](#g-01-make-variables-outside-of-the-loops)] | Make variables outside of the loops | 21 |
| [[G-02](#g-02-cache-complex-conditions)] | Cache complex conditions | 1 |
| [[G-03](#g-03-dont-cache-state-variable-if-it-is-used-only-once)] | Don't cache state variable if it is used only once | 1 |
| [[G-04](#g-04-check-for-zero-before-dividing)] | Check for zero before dividing | 1 |
| [[G-05](#g-05-missing-zero-address-check-in-constructor)] | Missing `zero-address` check in `constructor` | 1 |

**Note:  [G-03] and [G-04] findings name are similar as bot report but instances covered here in these findings are  different and missed by bot**

## [G-01] Make variables outside of the loops

When a variable is declared inside a loop, it may cause the memory to be allocated and deallocated repeatedly, leading to higher gas consumption. By declaring the variable outside the loop, memory is allocated only once, reducing gas costs.

_21 instances in 3 Files_

```solidity
File : contracts/multicall/Multicall.sol

13:     results = new bytes[](data.length);
14:     for (uint256 i = 0; i < data.length; ) {
15:         (bool success, bytes memory result) = address(this).delegatecall(data[i]);

```
[13-15](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/multicall/Multicall.sol#L13C9-L15C87)

```diff
File : contracts/multicall/Multicall.sol

        results = new bytes[](data.length);
+       bool success;
+       bytes memory result;
        for (uint256 i = 0; i < data.length; ) {
-            (bool success, bytes memory result) = address(this).delegatecall(data[i]);
+            (success, result) = address(this).delegatecall(data[i]);

```

```solidity
File : contracts/types/TokenId.sol

468:           for (uint256 i = 0; i < 4; ++i) {
469:                if (self.optionRatio(i) == 0) {
           ... 

508:           uint256 tokenTypeP = self.tokenType(riskPartnerIndex);

```
[468-508](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L468C2-L508C75)

```diff
File : contracts/types/TokenId.sol

+          uint256 riskPartnerIndex;
+          uint256 isLong;  
+          uint256 isLongP;
+          uint256 tokenType;
+          uint256 tokenTypeP;
           for (uint256 i = 0; i < 4; ++i) {
                if (self.optionRatio(i) == 0) {
                    // final leg in this position identified;
                    // make sure any leg above this are zero as well
                    // (we don't allow gaps eg having legs 1 and 4 active without 2 and 3 is not allowed)
                    if ((self >> (64 + 48 * i)) != 0) revert Errors.InvalidTokenIdParameter(1);

                    break; // we are done iterating over potential legs
                }
                // now validate this ith leg in the position:

                // The width cannot be 0; the minimum is 1
                if ((self.width(i) == 0)) revert Errors.InvalidTokenIdParameter(5);
                // Strike cannot be MIN_TICK or MAX_TICK
                if (
                    (self.strike(i) == Constants.MIN_V3POOL_TICK) ||
                    (self.strike(i) == Constants.MAX_V3POOL_TICK)
                ) revert Errors.InvalidTokenIdParameter(4);

                // In the following, we check whether the risk partner of this leg is itself
                // or another leg in this position.
                // Handles case where riskPartner(i) != i ==> leg i has a risk partner that is another leg
-                uint256 riskPartnerIndex = self.riskPartner(i);
+                riskPartnerIndex = self.riskPartner(i);
                 if (riskPartnerIndex != i) {
                    // Ensures that risk partners are mutual
                    if (self.riskPartner(riskPartnerIndex) != i)
                        revert Errors.InvalidTokenIdParameter(3);

                    // Ensures that risk partners have 1) the same asset, and 2) the same ratio
                    if (
                        (self.asset(riskPartnerIndex) != self.asset(i)) ||
                        (self.optionRatio(riskPartnerIndex) != self.optionRatio(i))
                    ) revert Errors.InvalidTokenIdParameter(3);

                    // long/short status of associated legs
-                    uint256 isLong = self.isLong(i);
+                    isLong = self.isLong(i);
-                    uint256 isLongP = self.isLong(riskPartnerIndex);
+                    isLongP = self.isLong(riskPartnerIndex);

                    // token type status of associated legs (call/put)
-                    uint256 tokenType = self.tokenType(i);
+                    tokenType = self.tokenType(i);
-                    uint256 tokenTypeP = self.tokenType(riskPartnerIndex);
+                    tokenTypeP = self.tokenType(riskPartnerIndex);

```

```solidity
File : contracts/SemiFungiblePositionManager.sol

583:    for (uint256 leg = 0; leg < numLegs; ) {
       ... 

624:    int256 fromBase = s_accountFeesBase[positionKey_from];

```
[583-624](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L583C9-L624C1)

```diff
File : contracts/SemiFungiblePositionManager.sol

+   uint256 liquidityChunk;
+   bytes32 positionKey_from;
+   bytes32 positionKey_to;
+   uint256 fromLiq;
+   int256 fromBase;
    for (uint256 leg = 0; leg < numLegs; ) {
            // for this leg index: extract the liquidity chunk: a 256bit word containing the liquidity amount and upper/lower tick
            // @dev see `contracts/types/LiquidityChunk.sol`
-           uint256 liquidityChunk = PanopticMath.getLiquidityChunk(
+           liquidityChunk = PanopticMath.getLiquidityChunk(
                id,
                leg,
                uint128(amount),
                univ3pool.tickSpacing()
            );

            //construct the positionKey for the from and to addresses
-            bytes32 positionKey_from = keccak256(
+            positionKey_from = keccak256(
                abi.encodePacked(
                    address(univ3pool),
                    from,
                    id.tokenType(leg),
                    liquidityChunk.tickLower(),
                    liquidityChunk.tickUpper()
                )
            );
-            bytes32 positionKey_to = keccak256(
+            positionKey_to = keccak256(
                abi.encodePacked(
                    address(univ3pool),
                    to,
                    id.tokenType(leg),
                    liquidityChunk.tickLower(),
                    liquidityChunk.tickUpper()
                )
            );

            // Revert if recipient already has that position
            if (
                (s_accountLiquidity[positionKey_to] != 0) ||
                (s_accountFeesBase[positionKey_to] != 0)
            ) revert Errors.TransferFailed();

            // Revert if not all balance is transferred
-           uint256 fromLiq = s_accountLiquidity[positionKey_from];
+           fromLiq = s_accountLiquidity[positionKey_from];
            if (fromLiq.rightSlot() != liquidityChunk.liquidity()) revert Errors.TransferFailed();

-           int256 fromBase = s_accountFeesBase[positionKey_from];
+           fromBase = s_accountFeesBase[positionKey_from];

```

```solidity
File : contracts/SemiFungiblePositionManager.sol

860:      for (uint256 leg = 0; leg < numLegs; ) {
         
    ...
              
882:      uint256 liquidityChunk = PanopticMath.getLiquidityChunk(

```
[860-882](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L860C3-L882C73)

```diff
File : contracts/SemiFungiblePositionManager.sol

+       int256 _moved;
+       int256 _itmAmounts;
+       int256 _totalCollected;
+       IUniswapV3Pool _univ3pool;
+       uint256 _tokenId;
+       bool _isBurn;
+       uint128 _positionSize;
+       uint256 _leg;
+       uint256 liquidityChunk;
        for (uint256 leg = 0; leg < numLegs; ) {
-            int256 _moved;
-            int256 _itmAmounts;
-            int256 _totalCollected;
+            _moved;
+            _itmAmounts;
+            _totalCollected;

            {
                // cache the univ3pool, tokenId, isBurn, and _positionSize variables to get rid of stack too deep error
-               IUniswapV3Pool _univ3pool = univ3pool;
-               uint256 _tokenId = tokenId;
-               bool _isBurn = isBurn;
-               uint128 _positionSize = positionSize;
-               uint256 _leg;
+               _univ3pool = univ3pool;
+               _tokenId = tokenId;
+               _isBurn = isBurn;
+               _positionSize = positionSize;
+               _leg;

                unchecked {
                    // Reverse the order of the legs if this call is burning a position (LIFO)
                    // We loop in reverse order if burning a position so that any dependent long liquidity is returned to the pool first,
                    // allowing the corresponding short liquidity to be removed
                    _leg = _isBurn ? numLegs - leg - 1 : leg;
                }

                // for this _leg index: extract the liquidity chunk: a 256bit word containing the liquidity amount and upper/lower tick
                // @dev see `contracts/types/LiquidityChunk.sol`
-               uint256 liquidityChunk = PanopticMath.getLiquidityChunk(
+               liquidityChunk = PanopticMath.getLiquidityChunk(

```

## [G-02] Cache complex conditions 

When a complex condition is used multiple times within a function, caching it into a local variable can reduce gas consumption by preventing redundant evaluations.

_1 instance in 1 File_

```soldity
File : contracts/SemiFungiblePositionManager.sol

917:        if (amount0 > uint128(type(int128).max) || amount1 > uint128(type(int128).max))

```
[917](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L917C9-L917C88)

```diff
File : contracts/SemiFungiblePositionManager.sol

+       uint128 int128Max = uint128(type(int128).max);
-       if (amount0 > uint128(type(int128).max) || amount1 > uint128(type(int128).max))
+       if (amount0 > int128Max || amount1 > int128Max)

```

## [G-03] Don't cache state variable if it is used only once

_1 instance in 1 File_

**Note: Here only those instances are covered that were missed by bot-report**

```solidity
File : contracts/SemiFungiblePositionManager.sol

623:           int256 fromBase = s_accountFeesBase[positionKey_from]; //@audit don't cache s_accountFeesBase[positionKey_from]
624:
625:            //update+store liquidity and fee values between accounts
626:            s_accountLiquidity[positionKey_to] = fromLiq;
627:            s_accountLiquidity[positionKey_from] = 0;
628:
629:            s_accountFeesBase[positionKey_to] = fromBase;
630:            s_accountFeesBase[positionKey_from] = 0;

```
[623-230](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L623C2-L630C53)

## [G-04] Check for zero before dividing

By performing this check, you ensure that the division operation only occurs when the denominator is non-zero, thus preventing unnecessary computations and potential errors that might consume additional gas or lead to unexpected behavior.

**Note: Here only those instances are covered that were missed by bot-report**

_1 instance in 1 File_

```solidity
File : contracts/libraries/Math.sol

108:                mulDiv(
109:                    uint256(liquidityChunk.liquidity()) << 96,
110:                    highPriceX96 - lowPriceX96,
111:                    highPriceX96
112:                ) / lowPriceX96;  //@audit check lowPriceX96

```
[108-112](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L108C1-L112C33)

## [G-05] Missing `zero-address` check in `constructor`

_1 instance in 1 File_

Missing checks for zero-addresses may lead to infunctional protocol, if the variable addresses are updated incorrectly. It also waste gas as it requires the redeployment of the contract.

```solidity
File : contracts/SemiFungiblePositionManager.sol

342:    constructor(IUniswapV3Factory _factory) {
343:        FACTORY = _factory;
344:    }

```
[342-344](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L342C1-L344C6)