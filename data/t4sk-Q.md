# Set specific compiler version.

Contract under the following folders are set to Solidity `pragma solidity ^0.8.0;`
- libraries
- tokens (this is included in the bot report)
- types

Compiler version should be set a specific version, such as `0.8.18` to avoid bugs that may be present in certain compiler version. 

# Missing check on length of arrays in `ERC115.balanceOfBatch`

https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/tokens/ERC1155Minimal.sol#L179-L180

### Consequence
`balanceOfBatch` when `owners.length < ids.length` will result in incomplete query of balances.

### Recommendation

```solidity
# balanceOfBatch
require(owners.length == ids.length)
```

# SemiFungiblePositionManager - incorrect comment on int size

# Incorrect comments on fee calculation
Comments on fee and premia equations use `feeGrowthX128` but both inside this code and Uniswap v3, fee calculations are done with `feeGrowthInsideX128``so the comments should be fixed to use `feeGrowthInsideX128`.

https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L195-L275

https://github.com/Uniswap/v3-core/blob/d8b1c635c275d2a9450bd6a78f3fa2484fef73eb/contracts/libraries/Position.sol#L61-L76

