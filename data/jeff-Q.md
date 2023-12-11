## L-1 Reentrancy Vulnerability in `Multicall` Contract
https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/multicall/Multicall.sol#L8

The Multicall contract is susceptible to a reentrancy attack, where a malicious contract may re-enter the `multicall` function before the original call finishes execution. This vulnerability arises when external calls made within the `multicall` function trigger functions in other contracts that, in turn, allow reentrant calls back into the `Multicall` contract.An attacker could potentially manipulate the state of the contract, leading to unexpected behavior
Implement a reentrancy guard at the beginning of the `multicall` function to prevent reentrant calls during the execution of external code.

## L-2 Lack of Array Size checks
`safeBatchTransferFrom` and `balanceOfBatch` in `erc1155.sol` lack proper array size checks in inputs leading to array index out of bound issues.