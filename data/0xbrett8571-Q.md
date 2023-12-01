# L-01 FUNCTION RETURNS TYPE AND NO RETURN

## Impact

These functions will always return a default `uint256` value of 0, regardless of the calculations performed inside the function body.

Specifically: [Line 68](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/LiquidityChunk.sol#L68)

- The function signature specifies that it returns a `uint256` value:

```solidity
returns (uint256)
```

- However, there is no explicit `return` statement with a specified return value inside the function body.

- So when this function gets called, the following happens:

1. The input parameters `self`, `_tickLower`, `_tickUpper`, `amount` get used in some calculations like `addLiquidity`, `addTickLower` etc.

2. But the return value of those calculations is not stored or returned explicitly.

3. So by default, the function returns 0, which gets typecast to a `uint256`.

This means all the computation happening inside `createChunk` has no impact on the actual return value. The return value will always be 0.

Essentially, it wastes gas performing pointless calculations that don't have any observable effect for the caller of this function.

The impact is broken and meaningless logic in the contract. This can disrupt expected behavior for users and makes the contract behave in an unreliable or unexpected way.

The resolution would be to explicitly `return` the computed value that is intended to be returned by this function.

https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/types/LiquidityChunk.sol#L63-L72
```solidity
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

_**Similar issues with these functions I listed below, they declare a return value in the function signature but do not explicitly return anything. Declaring that they return a `uint256` or some other type, but do not have a `return x;` statement inside the function body.**_

https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/types/LiquidityChunk.sol#L78-L82
```solidity
    function addLiquidity(uint256 self, uint128 amount) internal pure returns (uint256) {
        unchecked {
            return self + uint256(amount);
        }
    }
```
https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/types/LiquidityChunk.sol#L88-L92
```solidity
    function addTickLower(uint256 self, int24 _tickLower) internal pure returns (uint256) {
        unchecked {
            return self + (uint256(uint24(_tickLower)) << 232);
        }
    }
```

https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/types/LiquidityChunk.sol#L98-L103
```solidity
    function addTickUpper(uint256 self, int24 _tickUpper) internal pure returns (uint256) {
        unchecked {
            // convert tick upper to uint24 as explicit conversion from int24 to uint256 is not allowed
            return self + ((uint256(uint24(_tickUpper))) << 208);
        }
    }
```
https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/types/LiquidityChunk.sol#L112-L116
```solidity
    function tickLower(uint256 self) internal pure returns (int24) {
        unchecked {
            return int24(int256(self >> 232));
        }
    }
```
https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/types/LiquidityChunk.sol#L121-L125
```solidity
    function tickUpper(uint256 self) internal pure returns (int24) {
        unchecked {
            return int24(int256(self >> 208));
        }
    }
```
https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/types/LiquidityChunk.sol#L130-L134
```solidity
    function liquidity(uint256 self) internal pure returns (uint128) {
        unchecked {
            return uint128(self);
        }
    }
```
https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L44-L48
```solidity
    function toRightSlot(uint256 self, uint128 right) internal pure returns (uint256) {
        unchecked {
            return self + uint256(right);
        }
    }
```
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/LeftRight.sol#L54-L59
```solidity
    function toRightSlot(uint256 self, int128 right) internal pure returns (uint256) {
        if (right < 0) revert Errors.LeftRightInputError();
        unchecked {
            return self + uint256(int256(right));
        }
    }
```

https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L65-L69
```solidity
    function toRightSlot(int256 self, uint128 right) internal pure returns (int256) {
        unchecked {
            return self + int256(uint256(right));
        }
    }
```
https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L75-L80
```solidity
    function toRightSlot(int256 self, int128 right) internal pure returns (int256) {
        // bit mask needed in case rightHalfBitPattern < 0 due to 2's complement
        unchecked {
            return self + (int256(right) & RIGHT_HALF_BIT_MASK);
        }
    }
```
https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L108-L112
```solidity
    function toLeftSlot(uint256 self, uint128 left) internal pure returns (uint256) {
        unchecked {
            return self + (uint256(left) << 128);
        }
    }
```
https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L118-L122
```solidity
    function toLeftSlot(int256 self, uint128 left) internal pure returns (int256) {
        unchecked {
            return self + (int256(int128(left)) << 128);
        }
    }
```
https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L128-L132
```solidity
    function toLeftSlot(int256 self, int128 left) internal pure returns (int256) {
        unchecked {
            return self + (int256(left) << 128);
        }
    }
```
https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L23-L27
```solidity
    function absUint(int256 x) internal pure returns (uint256) {
        unchecked {
            return x > 0 ? uint256(x) : uint256(-x);
        }
    }
```

https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/PanopticMath.sol#L48-L59
```solidity
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

https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/PanopticMath.sol#L145-L161
```solidity
    function convert0to1(int256 amount, uint160 sqrtPriceX96) internal pure returns (int256) {
        unchecked {
            // the tick 443636 is the maximum price where (price) * 2**192 fits into a uint256 (< 2**256-1)
            // above that tick, we are forced to reduce the amount of decimals in the final price by 2**64 to 2**128
            if (sqrtPriceX96 < 340275971719517849884101479065584693834) {
                int256 absResult = Math
                    .mulDiv192(Math.absUint(amount), uint256(sqrtPriceX96) ** 2)
                    .toInt256();
                return amount < 0 ? -absResult : absResult;
            } else {
                int256 absResult = Math
                    .mulDiv128(Math.absUint(amount), Math.mulDiv64(sqrtPriceX96, sqrtPriceX96))
                    .toInt256();
                return amount < 0 ? -absResult : absResult;
            }
        }
    }
```

https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/PanopticMath.sol#L168-L188
```solidity
    function convert1to0(int256 amount, uint160 sqrtPriceX96) internal pure returns (int256) {
        unchecked {
            // the tick 443636 is the maximum price where (price) * 2**192 fits into a uint256 (< 2**256-1)
            // above that tick, we are forced to reduce the amount of decimals in the final price by 2**64 to 2**128
            if (sqrtPriceX96 < 340275971719517849884101479065584693834) {
                int256 absResult = Math
                    .mulDiv(Math.absUint(amount), 2 ** 192, uint256(sqrtPriceX96) ** 2)
                    .toInt256();
                return amount < 0 ? -absResult : absResult;
            } else {
                int256 absResult = Math
                    .mulDiv(
                        Math.absUint(amount),
                        2 ** 128,
                        Math.mulDiv64(sqrtPriceX96, sqrtPriceX96)
                    )
                    .toInt256();
                return amount < 0 ? -absResult : absResult;
            }
        }
    }
```

https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L80-L84
```solidity
    function univ3pool(uint256 self) internal pure returns (uint64) {
        unchecked {
            return uint64(self);
        }
    }
```

https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L93-L97
```solidity
    function asset(uint256 self, uint256 legIndex) internal pure returns (uint256) {
        unchecked {
            return uint256((self >> (64 + legIndex * 48)) % 2);
        }
    }
```


https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L103-L107
```solidity
    function optionRatio(uint256 self, uint256 legIndex) internal pure returns (uint256) {
        unchecked {
            return uint256((self >> (64 + legIndex * 48 + 1)) % 128);
        }
    }
```



https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L113-L117
```solidity
    function optionRatio(uint256 self, uint256 legIndex) internal pure returns (uint256) {
        unchecked {
            return uint256((self >> (64 + legIndex * 48 + 1)) % 128);
        }
    }
```



https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L123-L127
```solidity
    function tokenType(uint256 self, uint256 legIndex) internal pure returns (uint256) {
        unchecked {
            return uint256((self >> (64 + legIndex * 48 + 9)) % 2);
        }
    }
```


https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L149-L153
https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L139-L143
```solidity
    function riskPartner(uint256 self, uint256 legIndex) internal pure returns (uint256) {
        unchecked {
            return uint256((self >> (64 + legIndex * 48 + 10)) % 4);
        }
    }
```



https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L160-L164
```solidity
    function width(uint256 self, uint256 legIndex) internal pure returns (int24) {
        unchecked {
            return int24(int256((self >> (64 + legIndex * 48 + 36)) % 4096));
        } // "% 4096" = take last (2 ** 12 = 4096) 12 bits
    }
```



https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L173-L177
```solidity
    function width(uint256 self, uint256 legIndex) internal pure returns (int24) {
        unchecked {
            return int24(int256((self >> (64 + legIndex * 48 + 36)) % 4096));
        } // "% 4096" = take last (2 ** 12 = 4096) 12 bits
    }
```

https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/TokenId.sol#L189-L197
```solidity
    function addAsset(
        uint256 self,
        uint256 _asset,
        uint256 legIndex
    ) internal pure returns (uint256) {
        unchecked {
            return self + (uint256(_asset % 2) << (64 + legIndex * 48));
        }
    }
```

https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L204-L212
```solidity
    function addOptionRatio(
        uint256 self,
        uint256 _optionRatio,
        uint256 legIndex
    ) internal pure returns (uint256) {
        unchecked {
            return self + (uint256(_optionRatio % 128) << (64 + legIndex * 48 + 1));
        }
    }
```


https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L220-L228
```solidity
    function addIsLong(
        uint256 self,
        uint256 _isLong,
        uint256 legIndex
    ) internal pure returns (uint256) {
        unchecked {
            return self + ((_isLong % 2) << (64 + legIndex * 48 + 8));
        }
    }
```


https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L234-L242
```solidity
    function addTokenType(
        uint256 self,
        uint256 _tokenType,
        uint256 legIndex
    ) internal pure returns (uint256) {
        unchecked {
            return self + (uint256(_tokenType % 2) << (64 + legIndex * 48 + 9));
        }
    }
```


https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L248-L256
```solidity
    function addRiskPartner(
        uint256 self,
        uint256 _riskPartner,
        uint256 legIndex
    ) internal pure returns (uint256) {
        unchecked {
            return self + (uint256(_riskPartner % 4) << (64 + legIndex * 48 + 10));
        }
    }
```


https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L262-L270
```solidity
    function addStrike(
        uint256 self,
        int24 _strike,
        uint256 legIndex
    ) internal pure returns (uint256) {
        unchecked {
            return self + uint256((int256(_strike) & BITMASK_INT24) << (64 + legIndex * 48 + 12));
        }
    }
```


https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L276-L285
```solidity
    function addWidth(
        uint256 self,
        int24 _width,
        uint256 legIndex
    ) internal pure returns (uint256) {
        // % 4096 -> take 12 bits from the incoming 24 bits (there's no uint12)
        unchecked {
            return self + (uint256(uint24(_width) % 4096) << (64 + legIndex * 48 + 36));
        }
    }
```


https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L327-L355
```solidity
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



https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L361-L365
```solidity
    function countLongs(uint256 self) internal pure returns (uint256) {
        unchecked {
            return self.isLong(0) + self.isLong(1) + self.isLong(2) + self.isLong(3);
        }
    }
```

## Recommended Mitigation

Validate that every code path inside a functions declared as `returns (someType)` explicitly returns a value matching that type using a `return x;` statement.

For functions meant to return a value, ensure a return statement is present on ALL code paths. Do not rely on the default autosuggested return value.

For functions NOT meant to return a value, use the `returns()` signature with empty brackets instead of specifying a return type.

Use explicit `return` statements instead of relying on leftover memory values or call return values. Return expected and well-defined output.

# L-02 MISSING EVENTS

## Impact

The `safeTransferFrom` function does not emit events in that it reduces transparency and makes tracking/debugging token transfers more difficult.

- Without events, off-chain services have no way to reliably track when tokens are transferred via this function.

- There is no audit trail or log showing source, destination, amount for transfers.

- Troubleshooting failed transfers or accounting for token movements relies solely on on-chain contract storage rather than event logs. This makes historical tracking/analysis hard.

- Usage of this transfer function across contracts that rely on the SafeTransferLib would suffer from the same loss of transparency. No events emitted in any case.

- Can hide suspicious token draining/movement from monitoring tools looking for common transfer event signatures.

- Hurts gas optimization of consuming contracts that need to manually log transfers via this function.

https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/SafeTransferLib.sol#L16-L41
```solidity
    function safeTransferFrom(address token, address from, address to, uint256 amount) internal {
        bool success;


        assembly ("memory-safe") {
            // Get free memory pointer - we will store our calldata in scratch space starting at the offset specified here.
            let p := mload(0x40)


            // Write the abi-encoded calldata into memory, beginning with the function selector.
            mstore(p, 0x23b872dd00000000000000000000000000000000000000000000000000000000)
            mstore(add(4, p), from) // Append the "from" argument.
            mstore(add(36, p), to) // Append the "to" argument.
            mstore(add(68, p), amount) // Append the "amount" argument.


            success := and(
                // Set success to whether the call reverted, if not we check it either
                // returned exactly 1 (can't just be non-zero data), or had no return data.
                or(and(eq(mload(0), 1), gt(returndatasize(), 31)), iszero(returndatasize())),
                // We use 100 because that's the total length of our calldata (4 + 32 * 3)
                // Counterintuitively, this call() must be positioned after the or() in the
                // surrounding and() because and() evaluates its arguments from right to left.
                call(gas(), token, 0, p, 100, 0, 32)
            )
        }


        if (!success) revert Errors.TransferFailed();
    }
```

Lack of events reduces transaction traceability, transparency, and debuggability. Hurts integration with external services relying on event logs for analytics or notifications. Mandates extra work to manually log transfers.

https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/multicall/Multicall.sol#L12-L36

https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L321-L327

https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L331-L334

https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L407-L431

https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L441-L461

https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L544-L556

https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L563-L570

https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L578-L635

https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L663-L716

https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L743-L837

https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L848-L919

https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L936-L1067

https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L1073-L1087

https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L1129-L1162

https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L1169-L1190

https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L1200-L1248


## Recommended Mitigation
Consider emitting standard events like `TransferSingle` for fungible tokens or `TransferSingle` for non-fungibles immediately after sensitive state changes in critical functions.