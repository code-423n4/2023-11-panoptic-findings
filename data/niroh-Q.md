# LOW: Users are forced to max-approve the sfpm contract on any tokens they use with it


## Github Links
[https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L417](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L417)

[https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L460](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L460)


## Description
Due to a lack of mechanism that enables users to either approve exact amounts on the sfpm contract per operation, or transfer exact amounts using a callback, they are forced to use `token.approve(address(sfpm), type(uint256).max);` type approvals which are insecure.

## Impact
Max approvals are insecure as they are often left open and expose the users to funds loss inccured by potential future hacks of the sfpm contract.

## Proof of Concept

## Tools Used
Manual Review

## Recommended Mitigation Steps
One option is to provide view functions such as mintTokenizedPositionAmounts/burnTokenizedPositionAmounts which report the exact funds that need to be approved for the functions to succeed on the current state. (not ideal as state might change between the check and the tx execution)

Another option is to add optional "second-tier" callbacks to uniswapV3MintCallback and uniswapV3SwapCallback. If the caller impements these callbacks, token transfers to the pool are delegated to the caller (i.e. `payer.sfpmMintCallback(tokens,amounts,pool)`), and if not, the current code transferring funds on behalf of the user is executed. 