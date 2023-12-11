# Low
## [L-01] Incorrect checking for uninitialized `univ3pool`.
To check `univ3pool` is uninitialized, use `s_AddrToPoolIdData[univ3pool] == 0` instead of `univ3pool == IUniswapV3Pool(address(0)`.
```solidity
683:        if (univ3pool == IUniswapV3Pool(address(0))) revert Errors.UniswapPoolNotInitialized();
```
[L683](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L683)

## [L-02] The `addXXX` function familys may be used in a wrong way, reuslting unexpected `tokenId` value.
Check that the properties of `tokenId` are zero before addding that properties to `tokenId`.
```solidity
173:    function addUniv3pool(uint256 self, uint64 _poolId) internal pure returns (uint256) {
189:    function addAsset
204:    function addOptionRatio(
220:    function addIsLong(
234:    function addTokenType(
248:    function addRiskPartner(
262:    function addStrike(
276:    function addWidth(
```
[L173](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L173) | [L189](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L189) | [L204](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L204) | [L220](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L220) | [L234](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L234) | [L248](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L248) | [L262](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L262) | [L276](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L276)

## [L-03] `TokenId.countLongs` should count legs first and then count the longs for active legs
`countLongs` should count the longs only for active legs and ignores the inactive legs.
```solidity
363:            return self.isLong(0) + self.isLong(1) + self.isLong(2) + self.isLong(3);
```
[L363](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L363)

## [L-04] Strike price is not guaranteed to be multiples of `tickSpacing` when adding a tokenId.
Strike price is not guaranteed to be multiples of `tickSpacing` when adding a tokenI, resulting the tokenId cannot be used to create position (`SemiFungiblePositionManager._createPositionInAMM` => `PanopticMath.getLiquidityChunk` => `TokenId.asTicks` => revert). Validate the strike price early while adding a tokenId.
```solidity
388:            // The width is from lower to upper tick, the one-sided range is from strike to upper/lower
389:            int24 oneSidedRange = (selfWidth * tickSpacing) / 2;
390:
391:            (legLowerTick, legUpperTick) = (selfStrike - oneSidedRange, selfStrike + oneSidedRange);
392:
393:            // Revert if the upper/lower ticks are not multiples of tickSpacing
394:            // Revert if the tick range extends from the strike outside of the valid tick range
395:            // These are invalid states, and would revert silently later in `univ3Pool.mint`
396:            if (
397:                legLowerTick % tickSpacing != 0 ||
398:                legUpperTick % tickSpacing != 0 ||
399:                legLowerTick < minTick ||
400:                legUpperTick > maxTick
401:            ) revert Errors.TicksNotInitializable();
```
[L388-L401](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L388-L401)


## [L-05] Misspellings in comment.
The first word at L739 should be "If" instead of "It".
```solidity
739:    ///   It that position is burnt, then we remove a mix of the two tokens and swap one of them so that the user receives only one.
```
[L739](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L739)

## [L-06] `tokenType` is also used to calculate `positionKey`, which is missed in the comments for `s_accountLiquidity` and `s_accountFeesBase`.
```solidity
175:    /// @dev mapping that stores the liquidity data of keccak256(abi.encodePacked(address poolAddress, address owner, int24 tickLower, int24 tickUpper)) // @audit comment miss tokenType

292:    /// @dev mapping that stores a LeftRight packing of feesBase of  keccak256(abi.encodePacked(address poolAddress, address owner, int24 tickLower, int24 tickUpper))
```
[L175](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L175) | [L292](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L292)

## [L-07] Redundant words (`of the`) in comment.
```solidity
350:    /// @param fee The fee level of the of the underlying Uniswap v3 pool, denominated in hundredths of bips
```
[L350](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L350)


# Non-Critical
[N-01] Use `<=` and `>=` instead of `==` when checking strike price of option legs.
```solidity
482:                if (
483:                    (self.strike(i) == Constants.MIN_V3POOL_TICK) ||
484:                    (self.strike(i) == Constants.MAX_V3POOL_TICK)
485:                ) revert Errors.InvalidTokenIdParameter(4);
```
[L482-L485](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L482-L485)