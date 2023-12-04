## L-01: Minting positions might revert when user only has one-side token.
For writing options, call writers only require asset(ETH) to deposit and put writers only require cash(USDC) to deposit, they don't have to hold both asset and cash to write an option.
However, by the nature of the minting logic, when the positin is in the money, call writers need to hold both asset and cash even though one side is returned after swap.

Recommendation: Implementation could be upgraded by using flashloan, without requiring option writers to hold both asset and cash.

## L-02: Invalid check for upper/lower bound strike price
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/TokenId.sol#L482-L485

Recommendation: Use `<=` and `>=` for comparison.
