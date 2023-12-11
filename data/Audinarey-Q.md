## [1] Wrong NATSPEC comment is used for `s_accountFeesBase`
https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L175

The dev description for `s_accountFeesBase` is wrongly described as:
/// @dev mapping that stores a LeftRight packing of feesBase of  `keccak256(abi.encodePacked(address poolAddress, address owner, int24 tickLower, int24 tickUpper))`
missing out the token type

## Recommendation
change comment to:
/// @dev mapping that stores a LeftRight packing of feesBase of  `keccak256(abi.encodePacked(address poolAddress, address owner, uint256 tokenType, int24 tickLower, int24 tickUpper))`


## [2] `univ3pool` is cached after flipping is `isLong` bits
https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L674-L680

When `burnTokenizedPosition(...)` is called, the `isBurn` flag is set to `true` and the `isLong` bit for each `leg` of the `tokenId` is flipped thereby changing the `tokenId`.
```solidity
    function _validateAndForwardToAMM(
        uint256 tokenId,
        uint128 positionSize, // liquidity
        int24 tickLimitLow,
        int24 tickLimitHigh,
        bool isBurn
    ) internal returns (int256 totalCollectedFromAMM, int256 totalMoved, int24 newTick) {
        // Reverts if positionSize is 0 and user did not own the position before minting/burning
        if (positionSize == 0) revert Errors.OptionsBalanceZero();

        /// @dev the flipToBurnToken() function flips the isLong bits for burn it makes short long and longs short
        if (isBurn) {
            tokenId = tokenId.flipToBurnToken();
        }

        // Validate tokenId
        // Extract univ3pool from the poolId map to Uniswap Pool
   
        IUniswapV3Pool univ3pool = s_poolContext[tokenId.validate()].pool;

        // Revert if the pool not been previously initialized
        if (univ3pool == IUniswapV3Pool(address(0))) revert Errors.UniswapPoolNotInitialized();

        ...
}
```
Although the `tokenId` is a 256 bit word with different variables encoded into it, and it does not neccesarily affect the pool address


## Recommendation
consider caching the `univ3pool` before flipping the `isLong` bits


## [3] Wrong NATSPEC comment is used for `_getPremiaDeltas(...)`
https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1253-L1254

The `return` description for `deltaPremiumOwed` are `deltaPremiumGross` are wrongly described as:
    /// @return `deltaPremiumOwed` The extra premium (per liquidity X64) to be added to the owed accumulator for token0 (right) and token1 (right)
    /// @return `deltaPremiumGross` The extra premium (per liquidity X64) to be added to the gross accumulator for token0 (right) and token1 (right)

## Recommendation
change comment to:
    /// @return `deltaPremiumOwed` The extra premium (per liquidity X64) to be added to the owed accumulator for token0 (right) and token1 (left)
    /// @return `deltaPremiumGross` The extra premium (per liquidity X64) to be added to the gross accumulator for token0 (right) and token1 (left)