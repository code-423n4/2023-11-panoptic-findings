## [L-1] Avoid variable shadowing

1. `isLong` local variable shadows `isLong()` function 
- 503:  uint256 isLong = self.isLong(i);
- 113: function isLong(uint256 self, uint256 legIndex) internal pure returns (uint256) {
Files: 
- https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/TokenId.sol#L503
- https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/TokenId.sol#L113

2. `tokenType` local variable shadows `tokenType()` function
- 507: uint256 tokenType = self.tokenType(i);
- 123: function tokenType(uint256 self, uint256 legIndex) internal pure returns (uint256) {

Files: 
- https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/TokenId.sol#L507
- https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/TokenId.sol#L123
Recommendation: Rename the local variables.

## [L-2] Avoid unreachable code.
The assembly code at the end of the initializeAMMPool function can be reached because of the return statement before it.
File: 

```
function initializeAMMPool(address token0, address token1, uint24 fee) external {
        ....

        return;

        // this disables `memoryguard` when compiling this contract via IR
        // it is classed as a potentially unsafe assembly block by the compiler, but is in fact safe
        // we need this because enabling `memoryguard` and therefore StackLimitEvader increases the size of the contract significantly beyond the size limit
        assembly {
            mstore(0, 0xFA20F71C)
        }
    }
```
Recommendation: Remove unreachable code.

## [L-3] SemiFungiblePositionManager.sol does not follow standard upgradeability schema.
SemiFungiblePositionManager.sol does not follow upgradeability standard like:
1. using initialize function instead of naming it initializeAMMPools
2. using the openzeppelin's initializable contract and the initialize modifier

This contract does not follow the standard and will make it difficult to run analytic tools on it even when you want to updgrade to a new contract. Following standards makes so many other reviews easier and quicker.
Upgradeability standards that need to be followed are here: https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable

Standards help analytic tools to quickly check some issues like 
1. function-id-collision
2. function-shadowing
3. missing-init-modifier
4. initializer-missing
5. Detect variables that should be constant
6. Missing Variables

Recommendation: Follow the upgradeability standard and run 
```
slither-check-upgradeability contracts/SemiFungiblePositionManager.sol SemiFungiblePositionManager
```



