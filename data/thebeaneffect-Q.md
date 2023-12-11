# QA Report

## Summary

- L-01: Two different functions with the same name

### [L-01] Two different functions with the same name

Each variable and each function should have a separate name to avoid confusion. `function afterTokenTransfer` (called after a single token transfer) and `function afterTokenTransfer` (called after a batch token transfer) share the same name.


*Instances (2):*

    File: contracts/tokens/ERC1155Minimal.sol

    252:     function afterTokenTransfer(
    253:         address from,
    254:         address to,
    255:         uint256[] memory ids,
    256:         uint256[] memory amounts
    257:     ) internal virtual;

    265:     function afterTokenTransfer(
    266:         address from,
    267:         address to,
    268:         uint256 id,
    269:         uint256 amount
    270:     ) internal virtual;

[Link to code](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol)

    File: contracts/SemiFungiblePositionManager.sol

    544:     function afterTokenTransfer(
    545:         address from,
    546:         address to,
    547:         uint256[] memory ids,
    548:         uint256[] memory amounts
    549:     ) internal override {

    563:     function afterTokenTransfer(
    564:         address from,
    565:         address to,
    566:         uint256 id,
    567:         uint256 amount
    568:     ) internal override {

[Link to code](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol)

**Recommendation:** Change the name of the function called after a batch token transfer to `afterBatchTokenTransfer`, following the same logic as `safeTransferFrom` and `safeBatchTransferFrom` as shown below.

    File: contracts/tokens/ERC1155Minimal.sol

    83:      /// @notice Transfer a single token from one user to another

    90:      function safeTransferFrom(

    120:     /// @notice Transfer multiple tokens from one user to another

    128:     function safeBatchTransferFrom(