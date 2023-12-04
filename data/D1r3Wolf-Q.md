## Missing some common practices

1. In SafeTransferLib contract, in the safeTransferFrom function, the `from`, `to` addresses are not masked. You can find more about it here: https://github.com/transmissions11/solmate/pull/347

2. In `ERC1155Minimal` contract, the `_mint` and `_burn` functions are not calling the `afterTokenTransfer` function after the functionality. As this was expected from the abstract erc contracts, as the the afterTransfer hooks are used for some functionalities.

3. In `ERC1155Minimal` contract, the `safeBatchTransferFrom`, `balanceOfBatch` are not check the lists that came into the function have same length or not.