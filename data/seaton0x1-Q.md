#### 1.Unbounded loop
The project contains many instances of code that can loop indefinitely, leading to exhaustion of gas and transaction failure if the number of loops is excessive.
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L178-L191
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/multicall/Multicall.sol#L14-L35
#### Recommendation
Limit the number of loops, recommend users to process in batches.

#### 2.Pragma float
Many contracts within the project are using a floating pragma version, which could lead to compatibility issues with different compiler versions. It's advisable to specify a fixed compiler version in the pragma directive for greater consistency and safety
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L2
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L2

#### 3.Lack of address(0) checks
Input addresses should be checked against address(0) to prevent unexpected behavior.
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L351-L396
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L77-L81

#### 4.Critical changes should use a two-step pattern and a timelock
Lack of two-step procedure for critical operations leaves them error-prone.

Consider adding a two-steps pattern and a timelock on critical changes to avoid modifying the system state.
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L77-L81

#### 5.Lack of event for parameters changes
Adding an event will facilitate offchain monitoring when changing system parameters.


https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L178-L191

#### 6.Lack of old and new value for events related to parameter updates
Events that mark critical parameter changes should contain both the old and the new value.

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L236-L240

#### 7.Usage of return named variables and explicit values
Some functions return named variables, others return explicit values.

Following function returns an explicit value.
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L200-L204
Following function return returns a named variable.
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L178-L191

#### 8.The solidity documentation recommends the following order for functions:

* constructor
* receive function (if exists)
* fallback function (if exists)
* external
* public
* internal
* private

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol


