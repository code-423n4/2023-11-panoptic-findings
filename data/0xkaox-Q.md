## Location

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L78

## Impact
An unfaithful implementation of ERC1155 on the ERC1155Minimal.sol contract could lead a user to mistakenly approve the 0x0 address, allowing an attacker to steal user's tokens.

## Proof of Concept
An unfaithful implementation of ERC1155 could lead a user to mistakenly approve the 0x0 address, allowing an attacker to call from the constructor of a malicious contract the safeTransferFrom() or safeBatchTransferFrom() functions, leading to the loss of the user's tokens.
```
    function setApprovalForAll(address operator, bool approved) public {
        isApprovedForAll[msg.sender][operator] = approved; 
        emit ApprovalForAll(msg.sender, operator, approved);
    }

```


## Tools Used
Manual analysis.

## Recommended Mitigation Steps
Check for the zero address like in openzeppelin's implementation.
```
        if (operator == address(0)) {
            revert ERC1155InvalidOperator(address(0));
        }
```