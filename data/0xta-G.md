# Gas Optimization

## [G-01] Use inline assembly for balance math

Balance operations like additions and subtractions currently use Solidity math which involves opcode gas costs.

Replace Solidity math with inline assembly for direct SLOAD and SSTORE to storage

Gas Savings:

- SLOAD/SSTORE is approx 3x cheaper than Solidity additions/subtractions
- Estimated savings of ~30 gas per balance operation

```solidity
File: contracts/tokens/ERC1155Minimal.sol

    function safeTransferFrom(
        address from,
        address to,
        uint256 id,
        uint256 amount,
        bytes calldata data
    ) public {
        if (!(msg.sender == from || isApprovedForAll[from][msg.sender])) revert NotAuthorized();

        balanceOf[from][id] -= amount;

        // balance will never overflow
        unchecked {
            balanceOf[to][id] += amount;
        }

        afterTokenTransfer(from, to, id, amount);

        emit TransferSingle(msg.sender, from, to, id, amount);

        if (to.code.length != 0) {
            if (
                ERC1155Holder(to).onERC1155Received(msg.sender, from, id, amount, data) !=
                ERC1155Holder.onERC1155Received.selector
            ) {
                revert UnsafeRecipient();
            }
        }
    }
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L90-L118

### Implementation:

```solidity
function safeTransferFrom(from, to, id, amount) {

  // Get balances
  assembly {
    let fromBalance := sload(balanceOf, from, id)
    let toBalance := sload(balanceOf, to, id)
  }

  // Adjust balances
  assembly {
    sstore(balanceOf, from, id, sub(fromBalance, amount))
    sstore(balanceOf, to, id, add(toBalance, amount))
  }

  // Continue function
}
```

## [G-02] Returning a single packed uint rather than array from ERC1155Minimal::balanceOfBatch.

ERC1155Minimal::balanceOfBatch currently returns an array of balances

### Pack balances into a single unpacked uint.

Gas Savings:

- Array allocation/return is expensive
- Packed format reduces overhead

```solidity
File: contracts/tokens/ERC1155Minimal.sol

    function balanceOfBatch(
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

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L178-L191

### Implementation:

```solidity
function balanceOfBatch(owners, ids) external view returns(uint256 balances) {

  for(uint i = 0; i < owners.length; i++) {
    balances |= (balancesOf[owners[i]][ids[i]] << (i * BITS_PER_BALANCE));
  }

  return balances;
}

function getBalance(balances, index) internal pure returns(uint) {
  uint mask = (~uint256(0)) >> (BITS_PER_BALANCE * index);
  return (balances & mask) >> (BITS_PER_BALANCE * index);
}
```

## [G-03] Use inline assembly for bit manipulation operations LeftRight contract

Bitwise operations like shifting, masking currently use Solidity opcodes.

Replace with inline assembly for direct bit manipulation.

Gas Savings:

- Solidity bit ops are approximate 5x-10x more expensive than assembly
- Estimated 10-50 gas saved per bit op

```solidity
File: contracts/types/LeftRight.sol

89    function leftSlot(uint256 self) internal pure returns (uint128) {
90        return uint128(self >> 128);
91    }
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L89-L91

### Implementation:

```solidity
function leftSlot(x) internal pure returns (uint128) {

  assembly {
    let slot := shr(128, x)
    return(slot)
  }

}
```

## [G-04] LeftRight::toInt256() that perform the same conversion could cache outputs to avoid recomputation

Functions like toInt256() recompute the same conversions repeatedly.

Memoize converted values in a mapping to avoid recomputation.

Gas Savings:

- Conversion functions cost 100-200 gas each
- Memoization saves re-doing conversions

```solidity
File: contracts/types/LeftRight.sol

    function toInt256(uint256 self) internal pure returns (int256) {
        if (self > uint256(type(int256).max)) revert Errors.CastingError();
        return int256(self);
    }
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L212-L215

### Implementation:

```solidity
mapping(uint256 => int256) private uint2intCache;

function toInt256(uint256 x) internal view returns (int256) {
  if (uint2intCache[x] == 0) {
    uint2intCache[x] = int256(x);
  }
  return uint2intCache[x];
}
```

## [G-05] Concatenate instead of math addition

Chunk encoding currently uses addition to combine properties

Concatenate properties with bitwise OR instead of addition

Gas Savings:

- Addition is much more expensive than bitwise operations
- OR is approx 5x cheaper than addition

```solidity
File: contracts/types/LiquidityChunk.sol

   function createChunk(
        uint256 self,
        int24 _tickLower,
        int24 _tickUpper,
        uint128 amount
    ) internal pure returns (uint256) {
        unchecked {
            return self.addLiquidity(amount).addTickLower(_tickLower).addTickUpper(_tickUpper);
        }
    }
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LiquidityChunk.sol#L63-L72

### Implementation:

```solidity
function createChunk(..., tickLower, tickUpper, amount)
  returns (chunk)
{

  chunk = amount;

  chunk |= (uint256(tickLower) << 232);

  chunk |= (uint256(tickUpper) << 208);

  return chunk;

}
```

## [G-06] TokenId::flipToBurnToken function use comparisons that can be optimized

flipToBurnToken uses multiple comparisons to determine how many legs are active. Precompute the number of active legs instead of comparing on every call.

```solidity
File: contracts/types/TokenId.sol

    function flipToBurnToken(uint256 self) internal pure returns (uint256) {
        unchecked {
            // NOTE: This is a hack to avoid blowing up the contract size.
            // We copy the logic from the countLegs function, using it here adds 5K to the contract size with IR for some reason
            // Strip all bits except for the option ratios
            uint256 optionRatios = self & OPTION_RATIO_MASK;

            // The legs are filled in from least to most significant
            // Each comparison here is to the start of the next leg's option ratio
            // Since only the option ratios remain, we can be sure that no bits above the start of the inactive legs will be 1
            if (optionRatios < 2 ** 64) {
                optionRatios = 0;
            } else if (optionRatios < 2 ** 112) {
                optionRatios = 1;
            } else if (optionRatios < 2 ** 160) {
                optionRatios = 2;
            } else if (optionRatios < 2 ** 208) {
                optionRatios = 3;
            } else {
                optionRatios = 4;
            }

            // We need to ensure that only active legs are flipped
            // In order to achieve this, we shift our long bit mask to the right by (4-# active legs)
            // i.e the whole mask is used to flip all legs with 4 legs, but only the first leg is flipped with 1 leg so we shift by 3 legs
            // We also clear the poolId area of the mask to ensure the bits that are shifted right into the area don't flip and cause issues
            return self ^ ((LONG_MASK >> (48 * (4 - optionRatios))) & CLEAR_POOLID_MASK);
        }
    }
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L327-L355

### Implementation:

```solidity
// Cache number of active legs
uint8 numLegs;

function countLegs(uint256 tokenId) internal {
  // existing logic
  numLegs = result;
}

function flipToBurnToken(uint256 tokenId) internal returns (uint256) {

  uint256 mask = LONG_MASK;

  if(numLegs < 4) {
    mask = mask >> (48 * (4 - numLegs));
  }

  return tokenId ^ (mask & CLEAR_POOLID_MASK);

}
```

Avoid multiple comparisons on every call to determine active legs