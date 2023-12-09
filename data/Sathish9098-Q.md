##

## [L-1] Collision risks in pool ID generation in ``getFinalPoolId()`` Function

### Impact
The probability of a collision depends on the number of pools and the variety of token0, token1, and fee combinations. The more pools and varied combinations, the higher the chance of a collision.

If a collision occurs, two different sets of parameters would lead to the same finalPoolId, potentially causing confusion or errors in pool identification and management.

```solidity
FILE: 2023-11-panoptic/contracts/libraries/PanopticMath.sol

function getFinalPoolId(
        uint64 basePoolId,
        address token0,
        address token1,
        uint24 fee
    ) internal pure returns (uint64) {
        unchecked {
            return
                basePoolId +
                (uint64(uint256(keccak256(abi.encodePacked(token0, token1, fee)))) >> 32);
        }
    }

```
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/PanopticMath.sol#L48-L59

### Recommended Mitigation
Incorporating additional distinguishing parameters into the hash could also reduce collision risks. The function could use more than 64 bits from the hash

##

## [L-2] Array Lengths not checked

### Impact
 If the lengths of the ids and amounts arrays are not explicitly checked and enforced to be equal, the function may behave unpredictably. This can lead to scenarios where the contract processes an unequal number of ids and amounts, potentially resulting in incorrect token transfers.

```solidity
FILE: 2023-11-panoptic/contracts/tokens/ERC1155Minimal.sol

function safeBatchTransferFrom(
        address from,
        address to,
        uint256[] calldata ids,
        uint256[] calldata amounts,
        bytes calldata data
    ) public virtual {

```
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/tokens/ERC1155Minimal.sol#L128-L171

### Recommended Mitigation

```diff
+  require(ids.length == amounts.length, "IDs and amounts length mismatch");
```

##

## [L-3] Hardcoded function selector in ``safeTransferFrom()`` function

Since the function selector is hardcoded, your ``safeTransferFrom`` function can only interact with contracts that have a ``transferFrom`` method matching the exact signature transferFrom(address,address,uint256). If you encounter a token contract with a slightly different transferFrom method (even if just the parameter names are different), this hardcoded selector won't work.

Even though this function does the same thing, the signature is slightly different (sender and recipient instead of from and to). The function selector for this version of transferFrom will be different from 0x23b872dd. Therefore, your safeTransferFrom function will not be able to interact with this token contract because the hardcoded selector won't match.

```solidity
FILE: 2023-11-panoptic/contracts/libraries/SafeTransferLib.sol

20: mstore(p, 0x23b872dd00000000000000000000000000000000000000000000000000000000)

```
https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/libraries/SafeTransferLib.sol#L24

### Recommended Mitigation
 Instead of hardcoding, calculate the function selector dynamically within your contract based on the actual function signature

##

## [L-4] 





