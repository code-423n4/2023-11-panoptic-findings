## **[L-01] lack of input validation of arrays**

- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L178C3-L192C1

```solidity
File: contracts/tokens/ERC1155Minimal.sol
178: function balanceOfBatch(
        address[] calldata owners,
        uint256[] calldata ids
    ) public view returns (uint256[] memory balances) {
        balances = new uint256[](owners.length);

        // Unchecked because the only math done is incrementing
        // the array index counter which cannot possibly overflow.
        unchecked {
            for (uint256 i = 0; i < owners.length; ++i) {
                balances[i] = balanceOf[owners[i]][ids[i]];
            }
        }
    }
```

**if the array length of `ids.length` and `owenrs.length` is not equal, it can lead to an error. See dev comment on line [174](**https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L174**)**

**Recommend checking the input array length.**