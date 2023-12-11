
## 1. No revert condition on burning short options. [1]
- mintTokenizedPosition() and burnTokenizedPosition() are external, and in case of burnTokenizedPosition() is called first, **removedLiquidty**(the amount of liquidity currently bought) is 0 and wil be reverted.
- I think checking if **removedLiquidty** is greater than **chunkLiquidty** and revert **NotEnoughLiquidty** message will helpful.
    ```sh
    if (removedLiquidty < chunkLiquidity) {
        revert Errors.NotEnoughLiquidity();
    }
    ```

## 2. Definition of ITM amount.
"ITM" definition is little different in code implementation.
According to definition of ITM [here]
> In-the-Money (ITM) refers to a situation when the strike price of a call option is below the current market price of the underlying asset, or when the strike price of a put option is above the current market price of the underlying asset. In these scenarios, the option has intrinsic value, as exercising the option would result in a profit.
  - But in code implementation here([2]), ITM is calculated like **currentPrice** is between **tickLower** and **tickUpper**, no matter **currentPrice** is above **strikePrice**(in case of CALL option).
  
  


   [1]: <https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L979>
   [2]: <https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1037>
   [here]: <https://panoptic.xyz/docs/terms/in_the_money>