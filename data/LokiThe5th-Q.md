## Low-01: MultiCall::multicall() should not be payable  

https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/multicall/Multicall.sol#L12

## Impact  

There are no payable functions within the `ERC1155Minimal.sol` or the `SemiFungiblePositionManager.sol` contracts. There are also no functions that would allow the retrieval of any `ether` that's sent to these contracts. Thus any ether accidentally sent to these contracts via the `payable` `multicall` would be stuck in the contract.  

## Recommended Mitigation  

Remove the `payable` keyword from `Multicall::multicall()`.

--- 

## Low-02: Deterministic computation of addresses may fail on some zkEVMs  

## Impact  

Some zkEVMs derive addresses using a different method than what is used on Mainnet and most L2's. 

Given this, the code in the contest repo which relies on deterministic computation of addresses (`CalbackLib::validateCallback()` specifically) is unlikely to function correctly in such zkEVM contexts.

## Recommended Mitigation  

It is impossible (currently) to support all L2's relying on deterministic deployments. Please take care in the future when deploying to zk-L2's to ensure code compatibility.