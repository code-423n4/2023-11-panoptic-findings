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