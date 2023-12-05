# QA Report

| Id                                                                                          | Title                                                                       |
| ------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| [N-001](#n-001-keep-the-pattern-of-implicitly-passing-data-to-library-functions-consistent) | Keep the pattern of implicitly passing data to library functions consistent |
| [N-002](#n-002-incorrect-reference-to-bytes-in-comment)                                     | Incorrect reference to bytes in comment                                     |
| [N-003](#n-003-incorrect-reference-to-short-liquidity-in-comment)                           | Incorrect reference to short liquidity in comment                           |

---

### N-001 Keep the pattern of implicitly passing data to library functions consistent

- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L943

```solidity
 943:       uint256 _tokenType = TokenId.tokenType(_tokenId, _leg);
 944:       // unique key to identify the liquidity chunk in this uniswap pool
 945:       bytes32 positionKey = keccak256(
```

Here, `TokenId.tokenType(_tokenId, _leg)` should ideally follow the implicit pattern as in other places in the codebase - `_tokenId.tokenType(_leg)`.

## N-002 Incorrect reference to bytes in comment

- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L365

```solidity
365:     // Set the base poolId as last 8 bytes of the address (the first 16 hex characters)
366:     // @dev in the unlikely case that there is a collision between the first 8 bytes of two different Uni v3 pools
```

In `L365` it should be `first 8 bytes` instead of `last`. It is written correctly below on `L366`.

## N-003 Incorrect reference to short liquidity in comment

- https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L997

```solidity
 997:               /// @dev so the amount of short liquidity should increase.
 998:               if (!_isBurn) {
 999:                   removedLiquidity += chunkLiquidity;
 1000:              }
```

Here, it should be `long liquidity` instead of `short` since removal of liquidity is taking place on a long position.
