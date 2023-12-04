# Infinite loop risk 
The loop may be stuck as it may not update the variable every iteration and run into an infinite loop
[SemiFungiblePositionManager.sol #370](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L370)

```js
while (address(s_poolContext[poolId].pool) != address(0)) {
	poolId = PanopticMath.getFinalPoolId(poolId, token0, token1, fee);
}
```


# Use scientific notation
use scientific notation instes of multiplication and powers
[SemiFungiblePositionManager #384](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L384)
```js
s_AddrToPoolIdData[univ3pool] = uint256(poolId) + 2 ** 255;
```
# Unreachable code after return statement
The function mentioned below contains an unreachable code after the return statement, even if the code is made for the compiler, it should be put behind the return statement to not cause confusion.
[SemiFungiblePositionManager #393](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L393)
```js
return;

        //This disables `memoryguard` when compiling this contract via IR
        // it is classed as a potentially unsafe assembly block by the compiler, but is in fact safe
        // we need this because enabling `memoryguard` and therefore StackLimitEvader increases the size of the contract significantly beyond the size limit
        assembly {
            mstore(0, 0xFA20F71C)
        }
    }
```
# Assure values greater than 0
to make sure the logic is working properly check the following variable values are greater than 0
## Instances
* ```tickingTimeLow``` ,  ```tickingTimeHigh``` >0
[SemiFungiblePositionManager #663](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L663)
* ```amount``` > 0
 [ERC1155Minimal #90](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L90)
* ```amount``` > 0
 [ERC1155Minimal #128](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L128)



# Unchecked ```address(0)```
The mint function does not check if the ```to``` address is the ```address(0)```
 [ERC1155Minimal #214](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#214)


# General 
* ### The codebase comments are too much
	The comments should be brief explanations of functions, not documentations of the logic, include short explanations with links to documentation for further explanation
* ### Contracts are not using their OZ upgradeable counterpart
*  ### The protocol is not pausable






