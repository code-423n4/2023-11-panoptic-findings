# L-01 `swapATMint` call is not called when `tickLimitLow` <= `tickLimitHigh` which isn't evident by the comments

## Lines of code

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L706

## Impact
The comment [here](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L737) doesn't mention the fact that `swapInAMM` should only be called when `swapATMint` is true and when `tickLimitLow` > `tickLimitHigh`

## Tools Used
manual review
## Recommended Mitigation Steps
either update the comment or the if condition

# L-02 No events emitted in some critical functions

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


# L-04 Event emition before the function complete

## Lines of code
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L510-L533

## Impact
Event is emitted before the function ends, so a event can be emitted and still the function could revert. If there is any of-chain node or server that is looking for the event emittion then they can mistakenly consider that the function is completed but instead the function could revert

## Tools Used
manual review
## Recommended Mitigation Steps
move the event emittion to the end of funcion.

# L-05 usage of `uint256`

## Lines of code
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L578-L635

## Impact
The code is using `uint256` alot of times, it is not good to use, the maximum number of legs are 4 and still the code is using `uint256` to save the number of legs, you can see it [here](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L582-L583), this is not efficient way use lower value of uint as more uint can be stored in single slot

## Tools Used
manual review
## Recommended Mitigation Steps
use lower value for uint