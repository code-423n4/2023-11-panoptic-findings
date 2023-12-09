## Title:  The ERC1155Minimal contract does not follow ERC1155 spec

## **Severity:**

- Low Risk

## **Relevant GitHub Links:**

- https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/tokens/ERC1155Minimal.sol#L90
- https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/tokens/ERC1155Minimal.sol#L128

## **Vulnerability Details:**

The ERC155Minimal contract does not fully comply with the EIP1155 standard, specifically it misses the rules listed below.

***safeTransferFrom rules:***

- MUST revert if `_to` is the zero address.

```
function safeTransferFrom(address from, address to, uint256 id, uint256 amount, bytes calldata data) public {
        if (!(msg.sender == from || isApprovedForAll[from][msg.sender])) revert NotAuthorized();

        // @audit-info doesn't check if to is the zero address

        balanceOf[from][id] -= amount;

        // balance will never overflow
        unchecked {
            balanceOf[to][id] += amount;
        }

        ...
    }
```

***safeBatchTransferFrom rules:*** 

- MUST revert if `_to` is the zero address.
- MUST revert if length of `_ids` is not the same as length of `_values`.

```solidity
function safeBatchTransferFrom(
        address from,
        address to,
        uint256[] calldata ids,
        uint256[] calldata amounts,
        bytes calldata data
    ) public virtual {
        if (!(msg.sender == from || isApprovedForAll[from][msg.sender])) revert NotAuthorized();

        // Storing these outside the loop saves ~15 gas per iteration.
        uint256 id;
        uint256 amount;

	// @audit-info doesn't check if to is the zero address

        // @audit-info does not revert if ids and amounts are of unequal length, per eip specifications

        for (uint256 i = 0; i < ids.length;) {
            id = ids[i];
            amount = amounts[i];

            ...
        }

        ...
    }
```

## **Recommendation:**

Add the following checks:

```
function safeTransferFrom(address from, address to, uint256 id, uint256 amount, bytes calldata data) public {
        if (!(msg.sender == from || isApprovedForAll[from][msg.sender])) revert NotAuthorized();

        // add here
	if (to == address(0)) revert ("...");

        balanceOf[from][id] -= amount;

        // balance will never overflow
        unchecked {
            balanceOf[to][id] += amount;
        }

        ...
    }
```

```solidity
function safeBatchTransferFrom(
        address from,
        address to,
        uint256[] calldata ids,
        uint256[] calldata amounts,
        bytes calldata data
    ) public virtual {
        if (!(msg.sender == from || isApprovedForAll[from][msg.sender])) revert NotAuthorized();

        // Storing these outside the loop saves ~15 gas per iteration.
        uint256 id;
        uint256 amount;

	// add here
	if (to == address(0)) revert ("...");

        // add here
	if (ids.length != amounts.length) revert ("...");

        for (uint256 i = 0; i < ids.length;) {
            id = ids[i];
            amount = amounts[i];

            ...
            unchecked {
                ++i;
            }
        }

        ...
    }
```