## GAS1: tokentype, tokentypep, isLong and isLongP should be initialized outside of the for loop
#### Issue Identified

The code exhibits a potential inefficiency related to the repeated initialization of variables within a loop. Specifically, the variables tokentype, tokentypep, isLong, and isLongP are declared inside a loop, potentially incurring unnecessary gas costs due to repeated initialization.
#### Code Location

The issue can be found in the following section of the code:
[Link to Code](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/types/TokenId.sol#L503C21-L503C21)

#### Recommendation

To optimize gas consumption, it is advisable to declare the four variables (isLong, isLongP, tokenType, and tokenTypeP) outside the for loop. This approach avoids unnecessary reinitialization and reduces gas costs.
Recommended Fix

Simply declare the four variables before the for loop, as illustrated below:

```solidity

unchecked {
    uint256 isLong;
    uint256 isLongP;
    uint256 tokenType;
    uint256 tokenTypeP;
    for (uint256 i = 0; i < 4; ++i) {
    // ... (rest of the code)
```

This adjustment ensures that the variables are initialized only once, thereby optimizing gas usage.

## GAS2: unnecessary variables return

#### Issue Identified

The function _validateAndForwardToAMM unnecessarily returns variables (totalCollectedFromAMM, totalMoved, newTick) that are already declared in the returns statement.

#### Code Location

The issue can be found in the following section of the code:
[Link to Code](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L663-L715)
```solidity
function _validateAndForwardToAMM(
    uint256 tokenId,
    uint128 positionSize,
    int24 tickLimitLow,
    int24 tickLimitHigh,
    bool isBurn
) internal returns (int256 totalCollectedFromAMM, int256 totalMoved, int24 newTick) {
    // ... (function body)
    return (totalCollectedFromAMM, totalMoved, newTick);
}
```
#### Recommendation

Returning the already declared variables in the returns statement is unnecessary and can be simplified for better readability and gas optimization.
#### Recommended Fix

Simply remove the explicit return statement and let the function implicitly return the declared variables. The modified code should look like this:

```solidity

function _validateAndForwardToAMM(
    uint256 tokenId,
    uint128 positionSize,
    int24 tickLimitLow,
    int24 tickLimitHigh,
    bool isBurn
) internal returns (int256 totalCollectedFromAMM, int256 totalMoved, int24 newTick) {
    // ... (function body)
}
```

This change ensures that the variables are implicitly returned without the need for an explicit return statement, enhancing the code's simplicity and potentially optimizing gas usage.