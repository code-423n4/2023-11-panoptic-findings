## [QA-1] Gas Exhaustion and Out-of-Gas Risk in multicall Function
The `Multicall` function allows users to execute multiple calls within a single transaction, potentially leading to high gas consumption. Unbounded operations within the Multicall function, such as iterating through large data structures, can cause gas exhaustion and transaction failures.

https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/multicall/Multicall.sol#L12-L16

## [QA-2] Lack of Array Size Verification in SafeBatchTransferFrom and BalanceOfBatch
The `safeBatchTransferFrom` function does not check if the `ids` and `amounts` arrays have the same length before proceeding with the transfer operations.

The `balanceOfBatch` function assumes that the `owners` and `ids` arrays passed as arguments are of equal length without explicit verification.

If the arrays have different lengths, the function may access out-of-bounds indices resulting in unexpected behavior.
https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/tokens/ERC1155Minimal.sol#L128-L191

## [QA-3] Lack of ERC1155 standard compliance in `SemiFungiblePositionManager` contract
The `SemiFungiblePositionManager` contract claims to implement the ERC1155 standard but fails to include the necessary implementation for crucial ERC1155 functions such as `safeTransferFrom`, `safeBatchTransferFrom`..

Implement the missing ERC1155 functions in the SemiFungiblePositionManager contract to ensure compliance with the ERC1155 standard
https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L72

## [QA-4] Missing 'msg.sender' in 'PoolInitialized' Event
The PoolInitialized event is emitted without including the `msg.sender` information. Including the msg.sender in the event provides transparency and helps track the initiator of the pool initialization.
```
        emit PoolInitialized(univ3pool);
```
## [QA-5] Lack of Withdrawal Mechanism for Received Ether in `Multicall`
The `Multicall` contract is designed to be payable, allowing it to receive Ether during delegate calls. However, there is a critical oversight in the absence of a mechanism to withdraw or manage the received Ether. This vulnerability may lead to Ether accumulation within the contract creating a potential risk of fund lockup and limiting the contract's ability to efficiently handle received Ether.
```
    function multicall(bytes[] calldata data) public payable returns (bytes[] memory results) {

```

```