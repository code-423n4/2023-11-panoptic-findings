## [N-01] Duplicated code to count legs in `countLegs` and `flipToBurnToken`.
L331-L347 in `flipToBurnToken` are counting legs, and have the same effect as `countLegs` function. Call `countLegs` instead in `flipToBurnToken` for simplicity.
```solidity
331:            // Strip all bits except for the option ratios
332:            uint256 optionRatios = self & OPTION_RATIO_MASK;
333:
334:            // The legs are filled in from least to most significant
335:            // Each comparison here is to the start of the next leg's option ratio
336:            // Since only the option ratios remain, we can be sure that no bits above the start of the inactive legs will be 1
337:            if (optionRatios < 2 ** 64) {
338:                optionRatios = 0;
339:            } else if (optionRatios < 2 ** 112) {
340:                optionRatios = 1;
341:            } else if (optionRatios < 2 ** 160) {
342:                optionRatios = 2;
343:            } else if (optionRatios < 2 ** 208) {
344:                optionRatios = 3;
345:            } else {
346:                optionRatios = 4;
347:            }
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L331-L347

## [N-02] Misspellings in comment.
The first word at L739 should be "If" instead of "It".
```solidity
739:    ///   It that position is burnt, then we remove a mix of the two tokens and swap one of them so that the user receives only one.
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L739