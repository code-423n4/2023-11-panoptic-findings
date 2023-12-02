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