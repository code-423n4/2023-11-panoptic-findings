# GAS Report

## Summary

Total **73 instances** over **7 issues**with **2715** gas saved:

|ID|Issue|Instances|Gas|
|:--:|:---|:--:|:--:|
| [[G-01]](#g-01-the-arraylength-should-be-cached-outside-of-the-for-loop) | The `<array>.length` should be cached outside of the `for`-loop | 3 | 9 |
| [[G-02]](#g-02-dont-initialize-bool-variables-with-default-value) | Don't initialize `bool` variables with default value | 1 | - |
| [[G-03]](#g-03-using-assembly-to-check-for-zero-can-save-gas) | Using assembly to check for zero can save gas | 41 | 246 |
| [[G-04]](#g-04-newer-versions-of-solidity-are-more-gas-efficient) | Newer versions of solidity are more gas efficient | 13 | - |
| [[G-05]](#g-05-use-assembly-to-compute-hashes-to-save-gas) | Use assembly to compute hashes to save gas | 2 | 160 |
| [[G-06]](#g-06-use-calldata-instead-of-memory-for-immutable-arguments) | Use `calldata` instead of `memory` for immutable arguments | 5 | 1500 |
| [[G-07]](#g-07-the-result-of-a-function-call-should-be-cached-rather-than-re-calling-the-function) | The result of a function call should be cached rather than re-calling the function | 8 | 800 |

## Gas Optimizations

### [G-01] The `<array>.length` should be cached outside of the `for`-loop

The overheads outlined below are PER LOOP, excluding the first loop:
- storage arrays incur a Gwarmaccess (**100 gas**)
- memory arrays use `MLOAD` (**3 gas**)
- calldata arrays use `CALLDATALOAD` (**3 gas**)
Caching the length changes each of these to a `DUP<N>` (**3 gas**), and gets rid of the extra `DUP<N>` needed to store the stack offset.

There are 3 instances:

- *Multicall.sol* ( [14](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/multicall/Multicall.sol#L14) ):

```solidity
14:         for (uint256 i = 0; i < data.length; ) {
```

- *ERC1155Minimal.sol* ( [141](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/tokens/ERC1155Minimal.sol#L141), [187](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/tokens/ERC1155Minimal.sol#L187) ):

```solidity
141:         for (uint256 i = 0; i < ids.length; ) {

187:             for (uint256 i = 0; i < owners.length; ++i) {
```

### [G-02] Don't initialize `bool` variables with default value

It is redundant and wasting gas to initialize `bool` variables with default value.

There is 1 instance:

- *SemiFungiblePositionManager.sol* ( [127](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L127) ):

```solidity
127:     bool internal constant MINT = false;
```

### [G-03] Using assembly to check for zero can save gas

Using assembly to check for zero can save gas by allowing more direct access to the evm and reducing some of the overhead associated with high-level operations in solidity.

<details>
<summary>There are 41 instances (click to show):</summary>

- *SemiFungiblePositionManager.sol* ( [356](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L356), [363](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L363), [671](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L671), [683](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L683), [810](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L810), [820](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L820), [971](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L971), [1031](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L1031), [1043](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L1043), [1217](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L1217), [1394](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L1394) ):

```solidity
356:         if (address(univ3pool) == address(0)) revert Errors.UniswapPoolNotInitialized();

363:         if (s_AddrToPoolIdData[univ3pool] != 0) return;

671:         if (positionSize == 0) revert Errors.OptionsBalanceZero();

683:         if (univ3pool == IUniswapV3Pool(address(0))) revert Errors.UniswapPoolNotInitialized();

810:             } else if (itm0 != 0) {

820:             if (swapAmount == 0) return int256(0);

971:             if (isLong == 0) {

1031:             _moved = isLong == 0

1043:             if (_tokenType == 0) {

1217:         if (amountToCollect != 0) {

1394:             if (netLiquidity != 0) {
```

- *Math.sol* ( [43](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/Math.sol#L43), [47](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/Math.sol#L47), [49](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/Math.sol#L49), [51](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/Math.sol#L51), [53](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/Math.sol#L53), [55](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/Math.sol#L55), [57](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/Math.sol#L57), [59](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/Math.sol#L59), [61](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/Math.sol#L61), [63](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/Math.sol#L63), [65](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/Math.sol#L65), [67](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/Math.sol#L67), [69](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/Math.sol#L69), [71](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/Math.sol#L71), [73](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/Math.sol#L73), [75](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/Math.sol#L75), [77](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/Math.sol#L77), [79](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/Math.sol#L79), [81](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/Math.sol#L81), [83](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/Math.sol#L83), [89](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/Math.sol#L89), [206](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/Math.sol#L206), [302](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/Math.sol#L302), [364](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/Math.sol#L364), [426](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/Math.sol#L426), [488](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/Math.sol#L488) ):

```solidity
43:             uint256 sqrtR = absTick & 0x1 != 0

47:             if (absTick & 0x2 != 0) sqrtR = (sqrtR * 0xfff97272373d413259a46990580e213a) >> 128;

49:             if (absTick & 0x4 != 0) sqrtR = (sqrtR * 0xfff2e50f5f656932ef12357cf3c7fdcc) >> 128;

51:             if (absTick & 0x8 != 0) sqrtR = (sqrtR * 0xffe5caca7e10e4e61c3624eaa0941cd0) >> 128;

53:             if (absTick & 0x10 != 0) sqrtR = (sqrtR * 0xffcb9843d60f6159c9db58835c926644) >> 128;

55:             if (absTick & 0x20 != 0) sqrtR = (sqrtR * 0xff973b41fa98c081472e6896dfb254c0) >> 128;

57:             if (absTick & 0x40 != 0) sqrtR = (sqrtR * 0xff2ea16466c96a3843ec78b326b52861) >> 128;

59:             if (absTick & 0x80 != 0) sqrtR = (sqrtR * 0xfe5dee046a99a2a811c461f1969c3053) >> 128;

61:             if (absTick & 0x100 != 0) sqrtR = (sqrtR * 0xfcbe86c7900a88aedcffc83b479aa3a4) >> 128;

63:             if (absTick & 0x200 != 0) sqrtR = (sqrtR * 0xf987a7253ac413176f2b074cf7815e54) >> 128;

65:             if (absTick & 0x400 != 0) sqrtR = (sqrtR * 0xf3392b0822b70005940c7a398e4b70f3) >> 128;

67:             if (absTick & 0x800 != 0) sqrtR = (sqrtR * 0xe7159475a2c29b7443b29c7fa6e889d9) >> 128;

69:             if (absTick & 0x1000 != 0) sqrtR = (sqrtR * 0xd097f3bdfd2022b8845ad8f792aa5825) >> 128;

71:             if (absTick & 0x2000 != 0) sqrtR = (sqrtR * 0xa9f746462d870fdf8a65dc1f90e061e5) >> 128;

73:             if (absTick & 0x4000 != 0) sqrtR = (sqrtR * 0x70d869a156d2a1b890bb3df62baf32f7) >> 128;

75:             if (absTick & 0x8000 != 0) sqrtR = (sqrtR * 0x31be135f97d08fd981231505542fcfa6) >> 128;

77:             if (absTick & 0x10000 != 0) sqrtR = (sqrtR * 0x9aa508b5b7a84e1c677de54f3e99bc9) >> 128;

79:             if (absTick & 0x20000 != 0) sqrtR = (sqrtR * 0x5d6af8dedb81196699c329225ee604) >> 128;

81:             if (absTick & 0x40000 != 0) sqrtR = (sqrtR * 0x2216e584f5fa1ea926041bedfe98) >> 128;

83:             if (absTick & 0x80000 != 0) sqrtR = (sqrtR * 0x48a170391f7dc42444e8fa2) >> 128;

89:             sqrtPriceX96 = uint160((sqrtR >> 32) + (sqrtR % (1 << 32) == 0 ? 0 : 1));

206:             if (prod1 == 0) {

302:             if (prod1 == 0) {

364:             if (prod1 == 0) {

426:             if (prod1 == 0) {

488:             if (prod1 == 0) {
```

- *TokenId.sol* ( [443](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/TokenId.sol#L443), [464](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/TokenId.sol#L464), [469](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/TokenId.sol#L469), [473](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/TokenId.sol#L473) ):

```solidity
443:         if (i == 0)

464:         if (self.optionRatio(0) == 0) revert Errors.InvalidTokenIdParameter(1);

469:                 if (self.optionRatio(i) == 0) {

473:                     if ((self >> (64 + 48 * i)) != 0) revert Errors.InvalidTokenIdParameter(1);
```

</details>

### [G-04] Newer versions of solidity are more gas efficient

The solidity language continues to pursue more efficient gas optimization schemes. Adopting a [newer version of solidity](https://github.com/ethereum/solc-js/tags) can be more gas efficient.

<details>
<summary>There are 13 instances (click to show):</summary>

- *SemiFungiblePositionManager.sol* ( [2](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L2) ):

```solidity
2: pragma solidity =0.8.18;
```

- *CallbackLib.sol* ( [2](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/CallbackLib.sol#L2) ):

```solidity
2: pragma solidity ^0.8.0;
```

- *Constants.sol* ( [2](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/Constants.sol#L2) ):

```solidity
2: pragma solidity ^0.8.0;
```

- *Errors.sol* ( [2](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/Errors.sol#L2) ):

```solidity
2: pragma solidity ^0.8.0;
```

- *FeesCalc.sol* ( [2](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/FeesCalc.sol#L2) ):

```solidity
2: pragma solidity ^0.8.0;
```

- *Math.sol* ( [2](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/Math.sol#L2) ):

```solidity
2: pragma solidity ^0.8.0;
```

- *PanopticMath.sol* ( [2](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/PanopticMath.sol#L2) ):

```solidity
2: pragma solidity ^0.8.0;
```

- *SafeTransferLib.sol* ( [2](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/SafeTransferLib.sol#L2) ):

```solidity
2: pragma solidity ^0.8.0;
```

- *Multicall.sol* ( [2](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/multicall/Multicall.sol#L2) ):

```solidity
2: pragma solidity =0.8.18;
```

- *ERC1155Minimal.sol* ( [2](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/tokens/ERC1155Minimal.sol#L2) ):

```solidity
2: pragma solidity ^0.8.0;
```

- *LeftRight.sol* ( [2](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/LeftRight.sol#L2) ):

```solidity
2: pragma solidity ^0.8.0;
```

- *LiquidityChunk.sol* ( [2](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/LiquidityChunk.sol#L2) ):

```solidity
2: pragma solidity ^0.8.0;
```

- *TokenId.sol* ( [2](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/TokenId.sol#L2) ):

```solidity
2: pragma solidity ^0.8.0;
```

</details>

### [G-05] Use assembly to compute hashes to save gas

If the arguments to the encode call can fit into the scratch space (two words or fewer), then it's more efficient to use assembly to generate the hash (**80 gas**):

`keccak256(abi.encodePacked(x, y))` -> `assembly {mstore(0x00, a); mstore(0x20, b); let hash := keccak256(0x00, 0x40); }`

There are 2 instances:

- *SemiFungiblePositionManager.sol* ( [1104-1110](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L1104-L1110) ):

```solidity
1104:                 keccak256(
1105:                     abi.encodePacked(
1106:                         address(this),
1107:                         liquidityChunk.tickLower(),
1108:                         liquidityChunk.tickUpper()
1109:                     )
1110:                 )
```

- *PanopticMath.sol* ( [57](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/PanopticMath.sol#L57) ):

```solidity
57:                 (uint64(uint256(keccak256(abi.encodePacked(token0, token1, fee)))) >> 32);
```

### [G-06] Use `calldata` instead of `memory` for immutable arguments

Mark data types as `calldata` instead of `memory` where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as `calldata`. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies `memory` storage.

<details>
<summary>There are 5 instances (click to show):</summary>

- *SemiFungiblePositionManager.sol* ( [544-549](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L544-L549) ):

```solidity
/// @audit ids
/// @audit amounts
544:     function afterTokenTransfer(
545:         address from,
546:         address to,
547:         uint256[] memory ids,
548:         uint256[] memory amounts
549:     ) internal override {
```

- *CallbackLib.sol* ( [28-32](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/CallbackLib.sol#L28-L32) ):

```solidity
/// @audit features
28:     function validateCallback(
29:         address sender,
30:         address factory,
31:         PoolFeatures memory features
32:     ) internal pure {
```

- *ERC1155Minimal.sol* ( [252-257](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/tokens/ERC1155Minimal.sol#L252-L257) ):

```solidity
/// @audit ids
/// @audit amounts
252:     function afterTokenTransfer(
253:         address from,
254:         address to,
255:         uint256[] memory ids,
256:         uint256[] memory amounts
257:     ) internal virtual;
```

</details>

### [G-07] The result of a function call should be cached rather than re-calling the function

The function calls in solidity are expensive. If the same result of the same function calls are to be used several times, the result should be cached to reduce the gas consumption of repeated calls.

There are 8 instances:

- *SemiFungiblePositionManager.sol* ( [578](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L578), [936](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L936), [1200](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L1200) ):

```solidity
/// @audit `id.tokenType(leg)` called on lines: 598, 607
/// @audit `liquidityChunk.tickLower()` called on lines: 599, 608
/// @audit `liquidityChunk.tickUpper()` called on lines: 600, 609
578:     function registerTokenTransfer(address from, address to, uint256 id, uint256 amount) internal {

/// @audit `currentLiquidity.rightSlot()` called on lines: 1050, 967
936:     function _createLegInAMM(

/// @audit `movedInLeg.rightSlot()` called on lines: 1234, 1235
/// @audit `movedInLeg.leftSlot()` called on lines: 1237, 1238
1200:     function _collectAndWritePositionData(
```

- *TokenId.sol* ( [463](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/TokenId.sol#L463) ):

```solidity
/// @audit `self.optionRatio(i)` called on lines: 469, 499
/// @audit `self.strike(i)` called on lines: 483, 484
463:     function validate(uint256 self) internal pure returns (uint64) {
```

