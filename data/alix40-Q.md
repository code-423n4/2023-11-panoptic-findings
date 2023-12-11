## Non-Critical: Consistency in Comparison Operators for Tick Conditions

#### Issue Identified

In the function _getAMMSwapFeesPerLiquidityCollected, there is inconsistency in the usage of comparison operators for the conditions related to tick values. Specifically, the operators < and >= are used interchangeably in the if statements comparing currentTick with tickLower and tickUpper.
#### Code Location
 [Link to code](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/libraries/FeesCalc.sol#L89-L167)
The issue can be found in the following section of the code:

```solidity

unchecked {
    if (currentTick < tickLower) {
        // ... (code block)
    } else if (currentTick >= tickUpper) {
        // ... (code block)
    } else {
        // ... (code block)
    }
}
```

#### Observation

The condition currentTick < tickLower is used to check if the current tick is below the lower tick, and currentTick >= tickUpper is used to check if the current tick is above or equal to the upper tick. The inconsistency arises from the mix of < and >= operators, and it may be beneficial to maintain consistency for clarity and readability.
#### Recommendation

For consistency, consider using currentTick <= tickLower instead of currentTick < tickLower in the first if statement. This ensures uniformity in the comparison operators used for tick-related conditions in the function.
#### Updated Code

```solidity

unchecked {
    if (currentTick <= tickLower) {
        // ... (code block)
    } else if (currentTick >= tickUpper) {
        // ... (code block)
    } else {
        // ... (code block)
    }
}
```

This change aligns the conditions, making the code more consistent and potentially improving readability.

Feel free to implement this change if it aligns with your intended logic. If you have any further questions or concerns, please let me know.