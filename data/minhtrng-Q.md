### #[01]  No validation of legs being sorted with longs first

The SFPM has the following code:

```js
unchecked {
    // Reverse the order of the legs if this call is burning a position (LIFO)
    // We loop in reverse order if burning a position so that any dependent long liquidity is returned
    // to the pool first,
    // allowing the corresponding short liquidity to be removed
    _leg = _isBurn ? numLegs - leg - 1 : leg;
}
```

However when validating a position it is not enforced that longs or shorts are in any particular order, see [TokenId.validate](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/types/TokenId.sol#L463)

# [02] Tokens that contain legs with same tokenType and TickRange can not be transferred

[registerTransfer](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L578) reverts if the liquidity of `positionKey_to` is not 0:

```js
if (
    (s_accountLiquidity[positionKey_to] != 0) ||
    (s_accountFeesBase[positionKey_to] != 0)
) revert Errors.TransferFailed();
```

Since the position key is determined by target address, tokenType and tickRange. This will disallow a token transfer of tokenIds that contain legs with same tokenType and TickRange. As the transfer of 1 leg in the loop will set that value, causing a revert of the next iteration

# [03] Denial of tokenTransfers by frontRunning

Related to issue [2], a transfer of tokens can be denied by frontrunning, since [registerTransfer](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L578) revert if the `positionKey_to` already has liquidy.

# [04] Negative amountToCollect will be casted to large number

`_collectAndWritePositionData` casts the left and right slot of `amountToCollect` to uint128:

```js
(uint128 receivedAmount0, uint128 receivedAmount1) = univ3pool.collect(
    msg.sender,
    liquidityChunk.tickLower(),
    liquidityChunk.tickUpper(),
    uint128(amountToCollect.rightSlot()),
    uint128(amountToCollect.leftSlot())
);
```

As explained in `_getFeesBase`. The values can be negative:

```js
// (feegrowth * liquidity) / 2 ** 128
/// @dev here we're converting the value to an int128 even though all values (feeGrowth, liquidity, Q128) 
// are strictly positive.
/// That's because of the way feeGrowthInside works in Uniswap v3, where it can underflow when stored for the
// first time.
/// This is not a problem in Uniswap v3 because the fees are always calculated by taking the difference of th
// feeGrowths,
/// so that the net different is always positive.
/// So by using int128 instead of uint128, we remove the need to handle extremely large underflows and simply 
//allow it to be negative
feesBase = int256(0)
    .toRightSlot(int128(int256(Math.mulDiv128(feeGrowthInside0LastX128, liquidity))))
    .toLeftSlot(int128(int256(Math.mulDiv128(feeGrowthInside1LastX128, liquidity))));
```

In practice there shouldnt be a negative impact, because collecting if the collected fees are negative will just yield 0. To be safe, only casting if the slot values are > 0 would probably be a good idea.

