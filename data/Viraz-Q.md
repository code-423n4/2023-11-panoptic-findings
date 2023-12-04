# L-01 `swapATMint` call is not called when `tickLimitLow` <= `tickLimitHigh` which isn't evident by the comments

## Lines of code

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L706

## Impact
The comment [here](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L737) doesn't mention the fact that `swapInAMM` should only be called when `swapATMint` is true and when `tickLimitLow` > `tickLimitHigh`

## Tools Used
manual review
## Recommended Mitigation Steps
either update the comment or the if condition

#L-02 No events emitted in some critical functions

## Lines of code

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L407
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L441
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L1200
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L1073

## Impact
Events are necessary for these functions for better logging since there are multiple things happening in each method in the contract

## Tools Used
manual review
## Recommended Mitigation Steps
add missing events

# L-03 Lack of input validations

## Lines of code

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L351
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L482
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L516

## Impact
lack of input validation would delay reverting of a transaction for inavlid inputs so ideally a tx could be reverted early and gas could be saved

## Tools Used
manual review
## Recommended Mitigation Steps
add missing input validation checks