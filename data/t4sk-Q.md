# Set specific compiler version.

Contract under the following folders are set to Solidity `pragma solidity ^0.8.0;`
- libraries
- tokens (this is included in the bot report)
- types

Compiler version should be set a specific version, such as `0.8.18` to avoid bugs that may be present in certain compiler version. 

# Missing check on length of arrays in `ERC1155` `safeBatchTransferFrom` and `balanceOfBatch`

https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/tokens/ERC1155Minimal.sol#L131-L132

https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/tokens/ERC1155Minimal.sol#L179-L180

### Consequence
`safeBatchTransferFrom` when `ids.length < amounts.length` will result in unprocessed transfer for `amounts[i]` with `i >= ids.length`.

`balanceOfBatch` when `owners.length < ids.length` will result in incomplete query of balances.

### Recommendation
```solidity
# safeBatchTransferFrom
require(ids.length == amounts.length);
```

```solidity
# balanceOfBatch
require(owners.length == ids.length)
```