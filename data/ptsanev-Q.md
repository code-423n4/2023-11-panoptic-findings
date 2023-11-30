[L-01] - usage of ``slot0`` to get spot data of Univ3 pool is prone to manipulation:
``` // Get the current tick of the Uniswap pool, check slippage
        (, newTick, , , , , ) = univ3pool.slot0();

        if ((newTick >= tickLimitHigh) || (newTick <= tickLimitLow)) revert Errors.PriceBoundFail();
```
Use ``observe()`` instead

[L-02] - rebasing tokens would work against the users in events of negative rebases, as stated by the uniswap docs. This detail was probably missed, but due to not being mentioned in the README - Low

[L-03] - the ERC1155Minimal does not implement a ``balanceOf()``, only a ``balanceOfBatch()`` function, possibly tampering with potential integrations

[L-04] - tokens with blacklists would not allow users to burn their already opened positions, consider creating a mechanism for removing such users' liquidity, compensating the protocol with their fees.