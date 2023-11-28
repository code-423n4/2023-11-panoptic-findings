[L-01] - usage of ``slot0`` to get spot data of Univ3 pool is prone to manipulation:
``` // Get the current tick of the Uniswap pool, check slippage
        (, newTick, , , , , ) = univ3pool.slot0();

        if ((newTick >= tickLimitHigh) || (newTick <= tickLimitLow)) revert Errors.PriceBoundFail();
```
Use ``observe()`` instead

[L-02] - rebasing tokens would work against the users in events of negative rebases, as stated by the uniswap docs. This detail was probably missed, but due to not being mentioned in the README - Low