## **[01] lack of input validation of arrays**

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

`require(owners.length == ids.length, "LENGTH_MISMATCH");`

### [02] Missing Zero address check for `operator`

checking whether the provided **`operator`** address is the zero address (**`address(0)`** or **`0x0`**). If the zero address is allowed as an operator, it might introduce security issues, as it could potentially grant approval to an uninitialized or unauthorized entity.

- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L77

```solidity
FIle: contracts/tokens/ERC1155Minimal.sol
77: function setApprovalForAll(address operator, bool approved) public {
78:        isApprovedForAll[msg.sender][operator] = approved; //@issue check for zero address operator
79:
80:        emit ApprovalForAll(msg.sender, operator, approved);
81:    }
```

```diff
function setApprovalForAll(address operator, bool approved) public {
+    require(operator != address(0), "Invalid operator address"); // Check for zero address

    isApprovedForAll[msg.sender][operator] = approved;

    emit ApprovalForAll(msg.sender, operator, approved);
}
```