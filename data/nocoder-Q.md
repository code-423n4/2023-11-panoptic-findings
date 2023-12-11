**1) Fix notes to the `asTicks` function to clarify that it does not round down in the case of a negative number**

`asTicks` in `tokenId.sol` divides CONSTANTS.MIN_V3POOL_TICK by tickSpacing and then multiplies by tickSpacing and the notes state that this is for the purpose of rounding down and forcing minTick to be a multiple of tickSpacing. But MIN_V3POOL_TICK is negative, so this calculation actually ends up rounding up, toward 0, rather than down. Here is the relevant code and notes (link: https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/types/TokenId.sol#L385)
```
// The max/min ticks that can be initialized are the closest multiple of tickSpacing to the actual max/min tick abs()=887272
            // Dividing and multiplying by tickSpacing rounds down and forces the tick to be a multiple of tickSpacing
            int24 minTick = (Constants.MIN_V3POOL_TICK / tickSpacing) * tickSpacing;
            int24 maxTick = (Constants.MAX_V3POOL_TICK / tickSpacing) * tickSpacing;
``` 


Take tickSpacing = 9. 
```
function testRounding() public pure returns(int24) {
    int24 MIN_V3POOL_TICK = -887272;
    int24 tickSpacing = 9;
    return ((MIN_V3POOL_TICK/tickSpacing) * tickSpacing);
}

result: -887265
```
But -887272/9 = -98585.777…. Rounding down would be -98586, but this calculation actually rounds to -98585. This makes sense given the symmetry of minTick and maxTick (because minTick will now be -maxTick). So it might just be something to fix in the notes – just state that it rounds toward 0 rather than rounds down.


**2) You don't need to mod by 2^n (where n=# of bits) in all the tokenId functions**

All the functions that add the various portions (option ratio, risk partner, etc) of the tokenId to the tokenId first shift the bits by the appropriate amount and then mod by 2^n (where n is the number of bits of number being added). For example here is the function for adding the risk partner, and you see that it mods by 4 (the risk partner is 2 bits). Similarly, the option ratio is 7bits and its mods by 128 (2^7). (link: https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/types/TokenId.sol#L139C5-L142C10)

```
function riskPartner(uint256 self, uint256 legIndex) internal pure returns (uint256) {
        unchecked {
            return uint256((self >> (64 + legIndex * 48 + 10)) % 4);
        }
```

But the maximum number that can fit into n bits is (2^n - 1). This means that the number you are adding to the tokenId will always be less than what you are modding by (e.g., the max risk partner is 3 - modding 0, 1, 2, or 3 by 4 will always just result in the number itself since you are dividing by a larger number). Similarly, the largest option ratio is 127 and, whether the option ratio is 4 or 35 or 127, modding by 128 will just get you 4, 35, or 127, respectively. 

Technically, you still get the right result doing it the way it is in the code, but you are introducing extra unnecessary complexity (may also cost more gas).


**3) chunkLiquidity is subtracted from removedLiquidity in an unchecked block without a check that chunkLiquidity is less than removedLiquidity, so it could lead to an underflow**

The `_createLegInAMM` function creates a position in the AMM for each leg and updates the internal tracking of liquidity relating to the position. If the leg is short and the position was burnt, a long position is being closed, so the amount of short liquidity should decrease, which the function executes by subtracting the `chunkLiquidity` from `removedLiquidity`. But there is no check that `removedLiquidity` is greater than `chunkLiquidity`. 

And this calculation is within an unchecked block, so if `removedLiquidity` is in fact smaller than `chunkLiquidity` (even if that is just due to rounding or an error), then `removedLiquidity` could underflow to a very large inaccurate number. This number wouldn't align with the amount of liquidity in the user's actual position in Uniswap. The user could potentially burn this very high amount of liquidity and try to get those tokens back from Uniswap.

When the function calculates the `startingLiquidity` for a long leg by subtracting `chunkLiquidity`, it checks that `startingLiquidity` is not smaller than `chunkLiquidity`. The same check should be performed when subtracting `chunkLiquidity` from `removedLiquidity` (even if the assumption is that there should be sufficient liquidity, given that the block is unchecked so underflow is possible).

Here is the `_createLegInAMM` function which provides that if a long position is being burned, then `chunkLiquidity` is subtracted from `removedLiquidity`. This also shows that there is a check performed when subtracting `chunkLiquidity` from `startingLiquidity` but the same check is not performed when subtracting `chunkLiquidity` from `removedLiquidity` (link: https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L978)
```
function _createLegInAMM(
        IUniswapV3Pool _univ3pool,
        uint256 _tokenId,
        uint256 _leg,
        uint256 _liquidityChunk,
        bool _isBurn
    ) internal returns (int256 _moved, int256 _itmAmounts, int256 _totalCollected) {
        uint256 _tokenType = TokenId.tokenType(_tokenId, _leg);
        // unique key to identify the liquidity chunk in this uniswap pool
        bytes32 positionKey = keccak256(
            abi.encodePacked(
                address(_univ3pool),
                msg.sender,
                _tokenType,
                _liquidityChunk.tickLower(),
                _liquidityChunk.tickUpper()
            )
        );

        // update our internal bookkeeping of how much liquidity we have deployed in the AMM
        // for example: if this _leg is short, we add liquidity to the amm, make sure to add that to our tracking
        uint128 updatedLiquidity;
        uint256 isLong = TokenId.isLong(_tokenId, _leg);
        uint256 currentLiquidity = s_accountLiquidity[positionKey]; //cache

        unchecked {
            // did we have liquidity already deployed in Uniswap for this chunk range from some past mint?

            // s_accountLiquidity is a LeftRight. The right slot represents the liquidity currently sold (added) in the AMM owned by the user
            // the left slot represents the amount of liquidity currently bought (removed) that has been removed from the AMM - the user owes it to a seller
            // the reason why it is called "removedLiquidity" is because long options are created by removing -ie.short selling LP positions
            uint128 startingLiquidity = currentLiquidity.rightSlot();
            uint128 removedLiquidity = currentLiquidity.leftSlot();
            uint128 chunkLiquidity = _liquidityChunk.liquidity();

            if (isLong == 0) {
                // selling/short: so move from msg.sender *to* uniswap
                // we're minting more liquidity in uniswap: so add the incoming liquidity chunk to the existing liquidity chunk
                updatedLiquidity = startingLiquidity + chunkLiquidity;

                /// @dev If the isLong flag is 0=short but the position was burnt, then this is closing a long position
                /// @dev so the amount of short liquidity should decrease.
                if (_isBurn) {
                    removedLiquidity -= chunkLiquidity;
                }
           } else {
                // the _leg is long (buying: moving *from* uniswap to msg.sender)
                // so we seek to move the incoming liquidity chunk *out* of uniswap - but was there sufficient liquidity sitting in uniswap
                // in the first place?
                if (startingLiquidity < chunkLiquidity) {
                    // the amount we want to move (liquidityChunk.legLiquidity()) out of uniswap is greater than
                    // what the account that owns the liquidity in uniswap has (startingLiquidity)
                    // we must ensure that an account can only move its own liquidity out of uniswap
                    // so we revert in this case
                    revert Errors.NotEnoughLiquidity();
```

Add a check that `chunkLiquidity` is not greater than `removedliquidity`

```
if (_isBurn) { 
             if (removedLiquidity < chunkLiquidity) {
                revert Errors.NotEnoughLiquidity();}
                    else {
                    removedLiquidity -= chunkLiquidity;
                }
            }
```


**4) You will lose precision by dividing before you multiply in `asTicks`**

You will lose precision by first dividing `CONSTANTS.MIN_V3POOL_TICK` and `CONSTANT.MAX_V3POOL_TICK` by `tickSpacing` and then multiplying by tickspacing even though it is the equivalent of multiplying by 1 as shown here (link: https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/types/TokenId.sol#L385):
```
            int24 minTick = (Constants.MIN_V3POOL_TICK / tickSpacing) * tickSpacing;
            int24 maxTick = (Constants.MAX_V3POOL_TICK / tickSpacing) * tickSpacing;
```


For example, consider tickSpacing of 887. Logically you would expect MAX_V3POOL_TICK divided by 887 times 887 to equal MAX_V3POOL_TICK (887272), but you actually get 887,000 (see below).

```
function testRounding() public pure returns(int24) {
    int24 MAX_V3POOL_TICK = 887272;
    int24 tickSpacing = 887;
    return ((MAX_V3POOL_TICK/tickSpacing) * tickSpacing);
}
```
Result: 887000

This seems to be intentional so that the result is a multiple of the tick, but users should still be aware that it could result in the calculations being run on their transactions losing accuracy.