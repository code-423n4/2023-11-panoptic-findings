1. Incorrect comment.
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/Math.sol#L150
```diff
-     /// @notice Calculates the amount of liquidity for a given amount of token0 and liquidityChunk
+     /// @notice Calculates the amount of liquidity for a given amount of token1 and liquidityChunk
```
2. Dust amount of tokens (1 wei) is always substracted from user's funds in a mint-burn cycle. Ideally, burn should return the exact same amount of tokens that was spent. Below you can see logs from a PoC provided in my `Loss of precision in Math.getLiquidityForAmount1()` report:
```log
  Minting... 0x0000000000000000000000000000000000123456
  Liquidity to mint 18767501621118
  Token0:
  -567114396
  Token1:
  -254483561326829482
  Removed liq:  0
  Added liq:  18767501621118
  SFPM:  999999999999991608

  Burning... 0x0000000000000000000000000000000000123456
  Liquidity to burn 18767501621118
  Token0:
  567114395
  Token1:
  254483561326829481
  Removed liq:  0
  Added liq:  0
  SFPM:  0
```
3. There's a possible reentrancy with external systems that rely on the `SFPM` balances. Users can mint any amount of any `SFPM` tokens before they'll actually pay for the liquidity because `_mint()` in `mintTokenizedPosition()` contains a callback right after the balances change. It is better to move the minting to the end of the execution.
```diff
    function mintTokenizedPosition(
        uint256 tokenId,
        uint128 positionSize,
        int24 slippageTickLimitLow,
        int24 slippageTickLimitHigh
    )
        external
        ReentrancyLock(tokenId.univ3pool())
        returns (int256 totalCollected, int256 totalSwapped, int24 newTick)
    {
        // create the option position via its ID in this erc1155
-       _mint(msg.sender, tokenId, positionSize); // @audit-info can mint any amount of any tokenId and receive a callback

        emit TokenizedPositionMinted(msg.sender, tokenId, positionSize);

        // validate the incoming option position, then forward to the AMM for minting/burning required liquidity chunks
        (totalCollected, totalSwapped, newTick) =
            _validateAndForwardToAMM(tokenId, positionSize, slippageTickLimitLow, slippageTickLimitHigh, MINT);

+       _mint(msg.sender, tokenId, positionSize);
    }
```
4. There's also a reentrancy in `uniswapV3MintCallback()` with ERC777 tokens (the project supports them). When Alice calls `mintTokenizedPosition()` the following will happen:
      1. She'll receive SFPM tokens from `_mint()`;
      2. `_createLegInAMM()` calculates and updates the liquidity held by Alice;
      3. `_mintLiquidity()` calls `UniswapV3Pool.mint()`;
      4. Uniswap calls back to `uniswapV3MintCallback()`;
      5. The callback calls the token to transfer funds from Alice to Uniswap;
      6. And if the token is ERC777, it'll call Alice before the transfer. At this point, Alice has both SFPM tokens and liquidity to use in external systems before actually paying any money. Even if these external systems rely on non-standard liquidity counters inside `SFPM` (`s_accountLiquidity` mapping) instead of the standard ERC1155 to determine the liquidity held by Alice, they will still be vulnerable to this reentrancy.
Solution here is to update the liquidity after the payment.
