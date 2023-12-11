# QA Report

| QA     | Issues                                                                                       | Instances |
|--------|----------------------------------------------------------------------------------------------|-----------|
| [L-01](#L-01) | Misleading NatSpec Comments                                                        | 2         |
| [L-02](#L-02) | Add meaningful revert messages to the require statements                                                    | 6         |
| [L-03](#L-03) | Use latest stable version of Solidity                                                    | 11         |
| [NC-01](#NC-01) | Missing NatSpec function @return tag               | 3         |
| [NC-02](#NC-02) | Missing NatSpec function @param tag                     | 8         |

**Total Low-severity issues:** 19 instances across 3 issues

**Total NoNC-Critical issues:** 11 instances across 2 issues

**Total issues:** 30 instances across 5 issues

## Low Risk Issues

### [L-01] Misleading NatSpec Comments

Comments should align with the code

*Instances (2):*

#### 1- [SemiFungiblePositionManager.sol#L469-L470](https://github.com/code-423n4/2023-11-panoptic/blob/2647928c33be4a58883110befd7fd065448478ef/contracts/SemiFungiblePositionManager.sol#L469-L470)

```text
File: contracts/SemiFungiblePositionManager.sol

469:    /// @param tokenId The tokenId of the minted position, which encodes information about up to 4 legs
470:    /// @param positionSize The number of contracts minted, expressed in terms of the asset
```

The above comments should be corrected as follows:

```text
File: contracts/SemiFungiblePositionManager.sol

469:    /// @param tokenId The tokenId of the position to be burned, which encodes information about up to 4 legs
470:    /// @param positionSize The number of contracts to be burned, expressed in terms of the asset
```

#### 2- [SemiFungiblePositionManager.sol#L1253-L1254](https://github.com/code-423n4/2023-11-panoptic/blob/2647928c33be4a58883110befd7fd065448478ef/contracts/SemiFungiblePositionManager.sol#L1253-L1254)

```text
File: contracts/SemiFungiblePositionManager.sol

1253:  /// @return deltaPremiumOwed The extra premium (per liquidity X64) to be added to the owed accumulator for token0 (right) and token1 (right)
1254   /// @return deltaPremiumGross The extra premium (per liquidity X64) to be added to the gross accumulator for token0 (right) and token1 (right)
```

The above comment should be corrected as follows:

```text
File: contracts/SemiFungiblePositionManager.sol

1253:  /// @return deltaPremiumOwed The extra premium (per liquidity X64) to be added to the owed accumulator for token0 (right) and token1 (left)
1254   /// @return deltaPremiumGross The extra premium (per liquidity X64) to be added to the gross accumulator for token0 (right) and token1 (left)
```

### [L-02] Add meaningful revert messages to the require statements

In the Math library, there are multiple occasions where require statements are used for conditional checks. But these require statements are not using revert messages.

Hence, if these require statements revert, it will be difficult to find the reason for the revert.
It is recommended to add meaningful revert messages to these require statements.

*Instances (6):*

```solidity
File: contracts/libraries/Math.sol

207:    require(denominator > 0);

216:    require(denominator > prod1);

311:    require(2 ** 64 > prod1);

373:    require(2 ** 96 > prod1);

435:    require(2 ** 128 > prod1);

497:    require(2 ** 192 > prod1);
```

### [L-03] Use latest stable version of Solidity

Many files use solidity version 0.8.0. Consider using at least version 0.8.18 solidity pragma version because newer versions have bug fixes and security updates.

*Instances (11):*

Files:

* [CallbackLib.sol](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/libraries/CallbackLib.sol#L2)
* [Constants.sol](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/libraries/Constants.sol#L2)
* [Errors.sol](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/libraries/Errors.sol#L2)
* [FeesCalc.sol](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/libraries/FeesCalc.sol#L2)
* [Math.sol](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/libraries/Math.sol#L2)
* [PanopticMath.sol](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/libraries/PanopticMath.sol#L2)
* [SafeTransferLib.sol](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/libraries/SafeTransferLib.sol#L2)
* [ERC1155Minimal.sol](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/tokens/ERC1155Minimal.sol#L2)
* [LeftRight.sol](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/types/LeftRight.sol#L2)
* [LiquidityChunk.sol](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/types/LiquidityChunk.sol#L2)
* [TokenId.sol](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/types/TokenId.sol#L2)

## Non-critical Issues

### [NC-01] Missing NatSpec function @return tag

*Instances (3):*

#### 1- [TokenId.sol#L99-L103](<https://github.com/code-423n4/2023-11-panoptic/blob/2647928c33be4a58883110befd7fd065448478ef/contracts/types/TokenId.sol#L99-L103>)

```solidity
File: contracts/types/TokenId.sol

99:     /// @notice Get the number of contracts per leg.
100:    /// @param self the option position Id.
101:    /// @param legIndex the leg index of this position (in {0,1,2,3})
102:    /// @dev The final mod: "% 2**7" = takes the rightmost (2 ** 7 = 128) 7 bits of the pattern.
103:    function optionRatio(uint256 self, uint256 legIndex) internal pure returns (uint256) {
```

#### 2- [TokenId.sol#L322-L327](<https://github.com/code-423n4/2023-11-panoptic/blob/2647928c33be4a58883110befd7fd065448478ef/contracts/types/TokenId.sol#L322-L327>)

```solidity
File: contracts/types/TokenId.sol

322:    /// @notice Flip all the `isLong` positions in the legs in the `tokenId` option position.
323:    /// @dev uses XOR on existing isLong bits.
324:    /// @dev useful during burning an option position. So we need to take an existing tokenId but now burn it.
325:    /// The way to do this is to simply flip it to a short instead.
326:    /// @param self the tokenId in the SFPM representing an option position.
327:    function flipToBurnToken(uint256 self) internal pure returns (uint256) {
```

#### 3- [SemiFungiblePositionManager.sol#L1089-L1097](<https://github.com/code-423n4/2023-11-panoptic/blob/2647928c33be4a58883110befd7fd065448478ef/contracts/SemiFungiblePositionManager.sol#L1089-L1097>)

```solidity
File: contracts/SemiFungiblePositionManager.sol

1089:   /// @notice Compute the feesGrowth * liquidity / 2**128 by reading feeGrowthInside0LastX128 and feeGrowthInside1LastX128 from univ3pool.positions.
1090:   /// @param univ3pool the Uniswap pool.
1091:   /// @param liquidity the total amount of liquidity in the AMM for the specific position
1092:   /// @param liquidityChunk has lower tick, upper tick, and liquidity amount to mint
1093:   function _getFeesBase(
1094:       IUniswapV3Pool univ3pool,
1095:       uint128 liquidity,
1096:       uint256 liquidityChunk
1097:   ) private view returns (int256 feesBase) {
```

Should add @return tags specifying what the functions return.
Below is an example for the first instance of this issue:

```diff
File: contracts/types/TokenId.sol

    /// @notice Get the number of contracts per leg.
    /// @param self the option position Id.
    /// @param legIndex the leg index of this position (in {0,1,2,3})
    /// @dev The final mod: "% 2**7" = takes the rightmost (2 ** 7 = 128) 7 bits of the pattern.
+   /// @return the option ratio (the number of contracts per leg), greater than 0 for an active leg.
    function optionRatio(uint256 self, uint256 legIndex) internal pure returns (uint256) {
```

### [NC-02] Missing NatSpec function @param tag

*Instances (8):*

#### 1- [TokenId.sol#L170-L173](<https://github.com/code-423n4/2023-11-panoptic/blob/2647928c33be4a58883110befd7fd065448478ef/contracts/types/TokenId.sol#L170-L173>)

```solidity
File: contracts/types/TokenId.sol

170:    /// @notice Add the Uniswap v3 Pool pointed to by this option position.
171:    /// @param self the option position Id.
172:    /// @return the tokenId with the Uniswap V3 pool added to it.
173:    function addUniv3pool(uint256 self, uint64 _poolId) internal pure returns (uint256) {
```

#### 2- [TokenId.sol#L183-L193](<https://github.com/code-423n4/2023-11-panoptic/blob/2647928c33be4a58883110befd7fd065448478ef/contracts/types/TokenId.sol#L183-L193>)

```solidity
File: contracts/types/TokenId.sol

183:    /// @notice Add the asset basis for this position.
184:    /// @param self the option position Id.
185:    /// @param legIndex the leg index of this position (in {0,1,2,3})
186:    /// @dev occupies the leftmost bit of the optionRatio 4 bits slot
187:    /// @dev The final mod: "% 2" = takes the rightmost bit of the pattern
188:    /// @return the tokenId with numerarire added to the incoming leg index
189:    function addAsset(
190:        uint256 self,
191:        uint256 _asset,
192:        uint256 legIndex
193:    ) internal pure returns (uint256) {
```

#### 3- [TokenId.sol#199-208](<https://github.com/code-423n4/2023-11-panoptic/blob/2647928c33be4a58883110befd7fd065448478ef/contracts/types/TokenId.sol#L199-L208>)

```solidity
File: contracts/types/TokenId.sol

199:    /// @notice Add the number of contracts to leg index `legIndex`.
200:    /// @param self the option position Id
201:    /// @param legIndex the leg index of the position (in {0,1,2,3})
202:    /// @dev The final mod: "% 128" = takes the rightmost (2 ** 7 = 128) 7 bits of the pattern.
203:    /// @return the tokenId with optionRatio added to the incoming leg index
204:    function addOptionRatio(
205:        uint256 self,
206:        uint256 _optionRatio,
207:        uint256 legIndex
208:    ) internal pure returns (uint256) {
```

#### 4- [TokenId.sol#230-238](<https://github.com/code-423n4/2023-11-panoptic/blob/2647928c33be4a58883110befd7fd065448478ef/contracts/types/TokenId.sol#L230-L238>)

```solidity
File: contracts/types/TokenId.sol

230:    /// @notice Add the type of token moved for a given leg (implies a call or put). Either Token0 or Token1.
231:    /// @param self the tokenId in the SFPM representing an option position
232:    /// @param legIndex the leg index of this position (in {0,1,2,3})
233:    /// @return the tokenId with tokenType added to its relevant leg.
234:    function addTokenType(
235:        uint256 self,
236:        uint256 _tokenType,
237:        uint256 legIndex
238:    ) internal pure returns (uint256) {
```

#### 5- [TokenId.sol#L244-L252](<https://github.com/code-423n4/2023-11-panoptic/blob/2647928c33be4a58883110befd7fd065448478ef/contracts/types/TokenId.sol#L244-L252>)

```solidity
File: contracts/types/TokenId.sol

244:    /// @notice Add the associated risk partner of the leg index (generally another leg in the overall position).
245:    /// @param self the tokenId in the SFPM representing an option position
246:    /// @param legIndex the leg index of this position (in {0,1,2,3})
247:    /// @return the tokenId with riskPartner added to its relevant leg.
248:    function addRiskPartner(
249:        uint256 self,
250:        uint256 _riskPartner,
251:        uint256 legIndex
252:    ) internal pure returns (uint256) {
```

#### 6- [TokenId.sol#L258-L266](<https://github.com/code-423n4/2023-11-panoptic/blob/2647928c33be4a58883110befd7fd065448478ef/contracts/types/TokenId.sol#L258-L266>)

```solidity
File: contracts/types/TokenId.sol

258:    /// @notice Add the strike price tick of the nth leg (index `legIndex`).
259:    /// @param self the tokenId in the SFPM representing an option position.
260:    /// @param legIndex the leg index of this position (in {0,1,2,3})
261:    /// @return the tokenId with strike price tick added to its relevant leg
262:    function addStrike(
263:        uint256 self,
264:        int24 _strike,
265:        uint256 legIndex
266:    ) internal pure returns (uint256) {
```

#### 7- [TokenId.sol#L272-L280](<https://github.com/code-423n4/2023-11-panoptic/blob/2647928c33be4a58883110befd7fd065448478ef/contracts/types/TokenId.sol#L272-L280>)

```solidity
File: contracts/types/TokenId.sol

272:    /// @notice Add the width of the nth leg (index `legIndex`). This is half the tick-range covered by the leg (tickUpper - tickLower)/2.
273:    /// @param self the tokenId in the SFPM representing an option position.
274:    /// @param legIndex the leg index of this position (in {0,1,2,3})
275:    /// @return the tokenId with width added to its relevant leg
276:    function addWidth(
277:        uint256 self,
278:        int24 _width,
279:        uint256 legIndex
280:    ) internal pure returns (uint256) {
```

#### 8- [SafeTransferLib.sol#L16](<https://github.com/code-423n4/2023-11-panoptic/blob/2647928c33be4a58883110befd7fd065448478ef/contracts/libraries/SafeTransferLib.sol#L16>)

```solidity
File: contracts/libraries/SafeTransferLib.sol

16: function safeTransferFrom(address token, address from, address to, uint256 amount) internal {
```

Should add a @param tags for all arguments of those functions.
Below is an example for the `_tokenType` argument of TokenId::addTokenType() (the 4th occurrence of this issue).

```diff
File: contracts/types/TokenId.sol

    /// @notice Add the type of token moved for a given leg (implies a call or put). Either Token0 or Token1.
    /// @param self the tokenId in the SFPM representing an option position
    /// @param legIndex the leg index of this position (in {0,1,2,3})
+   /// @param _tokenType the type of token moved for a the `legIndex + `th leg. Either Token0 or Token1.
    /// @return the tokenId with tokenType added to its relevant leg.
    function addTokenType(
        uint256 self,
        uint256 _tokenType,
        uint256 legIndex
    ) internal pure returns (uint256) {
```
