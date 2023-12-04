## Initialize counter before loop saves gas
[SemiFungiblePositionManager #550](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L550)

## Use else statement to save gas
use else statement instead of another unnecessary if check
[SemiFungiblePositionManager #1043](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L1043)


## Loops can be implemented more efficiently 
*  [ERC1155Minimal #141](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L141)


*  [ERC1155Minimal #187](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L187)

### Mitigation
 Recommended implementation:

```

uint length = arr.length;

for (uint i; i < length; i++) {

//Operations not affecting the length of the array.

}
```
