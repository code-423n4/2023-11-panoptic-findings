# [N-01] REWRITE ON ASSEMBLY CALL

## Impact
The usage of `call` in inline assembly here carries the risk of calling an arbitrary contract address controlled by the `token` input parameter. This opens up possibilities for reentrancy or hijacking execution flow if the return value is not validated properly.

- If a malicious contract is passed as the `token` address, it can execute custom fallback code during the `call` that could call back into the calling contract and modify state while the assembly block still continues execution. This reentrancy risk can lead to loss of funds, incorrect state updates, etc.

- The return data from the `call` sits in scratch space pointed to by `p`. If the malicious contract writes custom raw bytes here, subsequent logic that assumes validated return data could be tricked and progress with an invalid state.

- The hardcoded expectation of a 1 return value checks if transfer succeeded. But a malicious contract at `token` address could ignore this convention and return any data or encoding it wants.

- If output data from the `call` is longer than expected, it can potentially overwrite parts of the input data sitting below scratch space. This could corrupt variables or calldata parameters for subsequent operations.

```solidity
call(gas(), token, 0, p, 100, 0, 32)
```

https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/SafeTransferLib.sol#L36-L36

## Recommendation

Follow checks-effects-interactions pattern by doing all validation before the `call` and revalidation after.

When expecting a specific return value like 1, mask and extract only the expected bytes. Avoid assumptions.

Disable oppcodes that intensify risks like `delegatecall`. Favor `staticcall` which protects state. 


# [N-02] REQUIRE WITH EMPTY MESSAGE

## Impact
Lack of descriptive error messages in `require` statements reduces debuggability.  It takes two parameters and the message part is optional. This is shown to the user when and if the `require` statement evaluates to `false`. Users see opaque reverts instead of insights into root causes. Adding customized strings aids tracing and resolves issues faster by clearly signaling to users what conditions or assumptions failed.

```solidity
    require(denominator > 0);
```
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/Math.sol#L207-L207

[Math.sol#L216-L216](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L216-L216)

[Math.sol#L311-L311](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L311-L311)

[Math.sol#L373-L373](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L373-L373)

[Math.sol#L435-L435](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L435-L435)

[Math.sol#L497-L497](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L497-L497)

## Recommendation
Add a descriptive message, no longer than `32 bytes`, inside the `require` statement to give more detail to the user about why the condition failed.

# [N-03] In-line assembly bypasses several important safety features and checks of Solidity.

## Impact
It uses low-level EVM instructions to encode the `calldata` for the token transfer function, including the "from", "to", and "amount" arguments. This bypasses some safety checks.

 - It makes a call to the token contract at address "token". The call transfers the tokens.

 - It checks if the call succeeded by checking if it reverted or if it returned exactly 1 (success).

The bypass increases the risk of vulnerabilities if assembly is not properly validated.

 - If there is a mistake in the assembly, it could inadvertently drain tokens from the contract or user accounts.

 - Transactions using inline assembly may be more susceptible to front-running attacks compared to a Solidity high-level call.

 - Requires manual gas stipulation, risking out-of-gas exceptions.

```solidity
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
```

_IN-LINE ASSEMBLY ARE ALSO IN THE FOLLOWING LINES_

[SafeTransferLib.sol#L19-L38](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/SafeTransferLib.sol#L19-L38)


[Multicall.sol#L25-L27](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/multicall/Multicall.sol#L25-L27)


[Math.sol#L199-L203](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L199-L203)


[Math.sol#L208-L210](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L208-L210)


[Math.sol#L225-L227](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L225-L227)


[Math.sol#L229-L232](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L229-L232)


[Math.sol#L239-L241](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L239-L241)


[Math.sol#L244-L246](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L244-L246)





[Math.sol#L295-L299](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L295-L299)




[Math.sol#L320-L322](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L320-L322)



[Math.sol#L324-L327](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L324-L327)


[Math.sol#L330-L333](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L330-L333)


[Math.sol#L357-L361](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L357-L361)


[Math.sol#L365-L368](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L365-L368)



[Math.sol#L382-L384](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L382-L384)


[Math.sol#L386-L389](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L386-L389)


[Math.sol#L330-L333](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L330-L333)


[Math.sol#L357-L361](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L357-L361)



[Math.sol#L365-L368](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L365-L368)


[Math.sol#L382-L384](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L382-L384)


[Math.sol#L386-L389](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L386-L389)


[Math.sol#L330-L333](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L330-L333)



[Math.sol#L357-L361](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L357-L361)



[Math.sol#L365-L368](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L365-L368)



[Math.sol#L382-L384](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L382-L384)



[Math.sol#L386-L389](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L386-L389)

[Math.sol#L386-L389](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L386-L389)


[Math.sol#L392-L395](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L392-L395)


[Math.sol#L419-L423](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L419-L423)




[Math.sol#L427-L430](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L427-L430)


[Math.sol#L444-L446](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L444-L446)


[Math.sol#L454-L457](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L448-L451)

[Math.sol#L454-L457](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L454-L457)


[Math.sol#L481-L485](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L481-L485)


[Math.sol#L489-L492](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L489-L492)


[Math.sol#L506-L508](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L506-L508)

[Math.sol#L510-L513](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L510-L513)

[Math.sol#L516-L519](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L516-L519)

[SemiFungiblePositionManager.sol#L393-L395](https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L393-L395)

## Recommendation
Avoid using inline assembly instructions if possible because it might introduce certain issues in the code if not dealt with properly because it bypasses several safety features that are already implemented.

# [GAS-04] MAKE PUBLIC LIBRARY FUNCTIONS INTERNAL

## Impact
Library `FeesCalc.sol` which has defined its function as public. This can be optimized by changing the function visibility. Changing the visibility from public will remove the compiler-introduced checks for `msg.value` and decrease the contract’s method ID table size

```solidity
    function calculateAMMSwapFeesLiquidityChunk(
        IUniswapV3Pool univ3pool,
        int24 currentTick,
        uint128 startingLiquidity,
        uint256 liquidityChunk
    ) public view returns (int256 feesEachToken) {
        // extract the amount of AMM fees collected within the liquidity chunk`
        // note: the fee variables are *per unit of liquidity*; so more "rate" variables
        (
            uint256 ammFeesPerLiqToken0X128,
            uint256 ammFeesPerLiqToken1X128
        ) = _getAMMSwapFeesPerLiquidityCollected(
                univ3pool,
                currentTick,
                liquidityChunk.tickLower(),
                liquidityChunk.tickUpper()
            );


        // Use the fee growth (rate) variable to compute the absolute fees accumulated within the chunk:
        //   ammFeesToken0X128 * liquidity / (2**128)
        // to store the (absolute) fees as int128:
        feesEachToken = feesEachToken
            .toRightSlot(int128(int256(Math.mulDiv128(ammFeesPerLiqToken0X128, startingLiquidity))))
            .toLeftSlot(int128(int256(Math.mulDiv128(ammFeesPerLiqToken1X128, startingLiquidity))));
    }
```
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/FeesCalc.sol#L54-L78

## Recommendation
The `public` functions can be changed to `private`/`internal` to save some gas.

# [GAS-05] UNUSED NAMED RETURNS

## Impact
Using both named returns and a return statement isn't necessary. Removing unused named return variables can reduce gas usage and improve code clarity.

```solidity
    function mulDiv(
        uint256 a,
        uint256 b,
        uint256 denominator
    ) internal pure returns (uint256 result) {
        unchecked {
            // 512-bit multiply [prod1 prod0] = a * b
            // Compute the product mod 2**256 and mod 2**256 - 1
            // then use the Chinese Remainder Theorem to reconstruct
            // the 512 bit result. The result is stored in two 256
            // variables such that product = prod1 * 2**256 + prod0
            uint256 prod0; // Least significant 256 bits of the product
            uint256 prod1; // Most significant 256 bits of the product
            assembly ("memory-safe") {
                let mm := mulmod(a, b, not(0))
                prod0 := mul(a, b)
                prod1 := sub(sub(mm, prod0), lt(mm, prod0))
            }


            // Handle non-overflow cases, 256 by 256 division
            if (prod1 == 0) {
                require(denominator > 0);
                assembly ("memory-safe") {
                    result := div(prod0, denominator)
                }
                return result;
            }


            // Make sure the result is less than 2**256.
            // Also prevents denominator == 0
            require(denominator > prod1);


            ///////////////////////////////////////////////
            // 512 by 256 division.
            ///////////////////////////////////////////////


            // Make division exact by subtracting the remainder from [prod1 prod0]
            // Compute remainder using mulmod
            uint256 remainder;
            assembly ("memory-safe") {
                remainder := mulmod(a, b, denominator)
            }
            // Subtract 256 bit number from 512 bit number
            assembly ("memory-safe") {
                prod1 := sub(prod1, gt(remainder, prod0))
                prod0 := sub(prod0, remainder)
            }


            // Factor powers of two out of denominator
            // Compute largest power of two divisor of denominator.
            // Always >= 1.
            uint256 twos = (0 - denominator) & denominator;
            // Divide denominator by power of two
            assembly ("memory-safe") {
                denominator := div(denominator, twos)
            }


            // Divide [prod1 prod0] by the factors of two
            assembly ("memory-safe") {
                prod0 := div(prod0, twos)
            }
            // Shift in bits from prod1 into prod0. For this we need
            // to flip `twos` such that it is 2**256 / twos.
            // If twos is zero, then it becomes one
            assembly ("memory-safe") {
                twos := add(div(sub(0, twos), twos), 1)
            }
            prod0 |= prod1 * twos;


            // Invert denominator mod 2**256
            // Now that denominator is an odd number, it has an inverse
            // modulo 2**256 such that denominator * inv = 1 mod 2**256.
            // Compute the inverse by starting with a seed that is correct
            // correct for four bits. That is, denominator * inv = 1 mod 2**4
            uint256 inv = (3 * denominator) ^ 2;
            // Now use Newton-Raphson iteration to improve the precision.
            // Thanks to Hensel's lifting lemma, this also works in modular
            // arithmetic, doubling the correct bits in each step.
            inv *= 2 - denominator * inv; // inverse mod 2**8
            inv *= 2 - denominator * inv; // inverse mod 2**16
            inv *= 2 - denominator * inv; // inverse mod 2**32
            inv *= 2 - denominator * inv; // inverse mod 2**64
            inv *= 2 - denominator * inv; // inverse mod 2**128
            inv *= 2 - denominator * inv; // inverse mod 2**256


            // Because the division is now exact we can divide by multiplying
            // with the modular inverse of denominator. This will give us the
            // correct result modulo 2**256. Since the precoditions guarantee
            // that the outcome is less than 2**256, this is the final result.
            // We don't need to compute the high bits of the result and prod1
            // is no longer required.
            result = prod0 * inv;
            return result;
        }
    }
```

https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L186-L280

```solidity
    function mulDiv64(uint256 a, uint256 b) internal pure returns (uint256 result) {
        unchecked {
            // 512-bit multiply [prod1 prod0] = a * b
            // Compute the product mod 2**256 and mod 2**256 - 1
            // then use the Chinese Remainder Theorem to reconstruct
            // the 512 bit result. The result is stored in two 256
            // variables such that product = prod1 * 2**256 + prod0
            uint256 prod0; // Least significant 256 bits of the product
            uint256 prod1; // Most significant 256 bits of the product
            assembly ("memory-safe") {
                let mm := mulmod(a, b, not(0))
                prod0 := mul(a, b)
                prod1 := sub(sub(mm, prod0), lt(mm, prod0))
            }


            // Handle non-overflow cases, 256 by 256 division
            if (prod1 == 0) {
                assembly ("memory-safe") {
                    // Right shift by n is equivalent and 2 gas cheaper than division by 2^n
                    result := shr(64, prod0)
                }
                return result;
            }


            // Make sure the result is less than 2**256.
            require(2 ** 64 > prod1);


            ///////////////////////////////////////////////
            // 512 by 256 division.
            ///////////////////////////////////////////////


            // Make division exact by subtracting the remainder from [prod1 prod0]
            // Compute remainder using mulmod
            uint256 remainder;
            assembly ("memory-safe") {
                remainder := mulmod(a, b, 0x10000000000000000)
            }
            // Subtract 256 bit number from 512 bit number
            assembly ("memory-safe") {
                prod1 := sub(prod1, gt(remainder, prod0))
                prod0 := sub(prod0, remainder)
            }


            // Divide [prod1 prod0] by the factors of two (note that this is just 2**96 since the denominator is a power of 2 itself)
            assembly ("memory-safe") {
                // Right shift by n is equivalent and 2 gas cheaper than division by 2^n
                prod0 := shr(64, prod0)
            }
            // Shift in bits from prod1 into prod0. For this we need
            // to flip `twos` such that it is 2**256 / twos.
            // If twos is zero, then it becomes one
            // Note that this is just 2**192 since 2**256 over the fixed denominator (2**64) equals 2**192
            prod0 |= prod1 * 2 ** 192;


            return prod0;
        }
    }
```
https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L286-L342



```solidity
    function mulDiv96(uint256 a, uint256 b) internal pure returns (uint256 result) {
        unchecked {
            // 512-bit multiply [prod1 prod0] = a * b
            // Compute the product mod 2**256 and mod 2**256 - 1
            // then use the Chinese Remainder Theorem to reconstruct
            // the 512 bit result. The result is stored in two 256
            // variables such that product = prod1 * 2**256 + prod0
            uint256 prod0; // Least significant 256 bits of the product
            uint256 prod1; // Most significant 256 bits of the product
            assembly ("memory-safe") {
                let mm := mulmod(a, b, not(0))
                prod0 := mul(a, b)
                prod1 := sub(sub(mm, prod0), lt(mm, prod0))
            }


            // Handle non-overflow cases, 256 by 256 division
            if (prod1 == 0) {
                assembly ("memory-safe") {
                    // Right shift by n is equivalent and 2 gas cheaper than division by 2^n
                    result := shr(96, prod0)
                }
                return result;
            }


            // Make sure the result is less than 2**256.
            require(2 ** 96 > prod1);


            ///////////////////////////////////////////////
            // 512 by 256 division.
            ///////////////////////////////////////////////


            // Make division exact by subtracting the remainder from [prod1 prod0]
            // Compute remainder using mulmod
            uint256 remainder;
            assembly ("memory-safe") {
                remainder := mulmod(a, b, 0x1000000000000000000000000)
            }
            // Subtract 256 bit number from 512 bit number
            assembly ("memory-safe") {
                prod1 := sub(prod1, gt(remainder, prod0))
                prod0 := sub(prod0, remainder)
            }


            // Divide [prod1 prod0] by the factors of two (note that this is just 2**96 since the denominator is a power of 2 itself)
            assembly ("memory-safe") {
                // Right shift by n is equivalent and 2 gas cheaper than division by 2^n
                prod0 := shr(96, prod0)
            }
            // Shift in bits from prod1 into prod0. For this we need
            // to flip `twos` such that it is 2**256 / twos.
            // If twos is zero, then it becomes one
            // Note that this is just 2**160 since 2**256 over the fixed denominator (2**96) equals 2**160
            prod0 |= prod1 * 2 ** 160;


            return prod0;
        }
    }
```
https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L348-L404


```solidity
    function mulDiv128(uint256 a, uint256 b) internal pure returns (uint256 result) {
        unchecked {
            // 512-bit multiply [prod1 prod0] = a * b
            // Compute the product mod 2**256 and mod 2**256 - 1
            // then use the Chinese Remainder Theorem to reconstruct
            // the 512 bit result. The result is stored in two 256
            // variables such that product = prod1 * 2**256 + prod0
            uint256 prod0; // Least significant 256 bits of the product
            uint256 prod1; // Most significant 256 bits of the product
            assembly ("memory-safe") {
                let mm := mulmod(a, b, not(0))
                prod0 := mul(a, b)
                prod1 := sub(sub(mm, prod0), lt(mm, prod0))
            }


            // Handle non-overflow cases, 256 by 256 division
            if (prod1 == 0) {
                assembly ("memory-safe") {
                    // Right shift by n is equivalent and 2 gas cheaper than division by 2^n
                    result := shr(128, prod0)
                }
                return result;
            }


            // Make sure the result is less than 2**256.
            require(2 ** 128 > prod1);


            ///////////////////////////////////////////////
            // 512 by 256 division.
            ///////////////////////////////////////////////


            // Make division exact by subtracting the remainder from [prod1 prod0]
            // Compute remainder using mulmod
            uint256 remainder;
            assembly ("memory-safe") {
                remainder := mulmod(a, b, 0x100000000000000000000000000000000)
            }
            // Subtract 256 bit number from 512 bit number
            assembly ("memory-safe") {
                prod1 := sub(prod1, gt(remainder, prod0))
                prod0 := sub(prod0, remainder)
            }


            // Divide [prod1 prod0] by the factors of two (note that this is just 2**128 since the denominator is a power of 2 itself)
            assembly ("memory-safe") {
                // Right shift by n is equivalent and 2 gas cheaper than division by 2^n
                prod0 := shr(128, prod0)
            }
            // Shift in bits from prod1 into prod0. For this we need
            // to flip `twos` such that it is 2**256 / twos.
            // If twos is zero, then it becomes one
            // Note that this is just 2**160 since 2**256 over the fixed denominator (2**128) equals 2**128
            prod0 |= prod1 * 2 ** 128;


            return prod0;
        }
    }
```
https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L410-L466



```solidity
    function mulDiv192(uint256 a, uint256 b) internal pure returns (uint256 result) {
        unchecked {
            // 512-bit multiply [prod1 prod0] = a * b
            // Compute the product mod 2**256 and mod 2**256 - 1
            // then use the Chinese Remainder Theorem to reconstruct
            // the 512 bit result. The result is stored in two 256
            // variables such that product = prod1 * 2**256 + prod0
            uint256 prod0; // Least significant 256 bits of the product
            uint256 prod1; // Most significant 256 bits of the product
            assembly ("memory-safe") {
                let mm := mulmod(a, b, not(0))
                prod0 := mul(a, b)
                prod1 := sub(sub(mm, prod0), lt(mm, prod0))
            }


            // Handle non-overflow cases, 256 by 256 division
            if (prod1 == 0) {
                assembly ("memory-safe") {
                    // Right shift by n is equivalent and 2 gas cheaper than division by 2^n
                    result := shr(192, prod0)
                }
                return result;
            }


            // Make sure the result is less than 2**256.
            require(2 ** 192 > prod1);


            ///////////////////////////////////////////////
            // 512 by 256 division.
            ///////////////////////////////////////////////


            // Make division exact by subtracting the remainder from [prod1 prod0]
            // Compute remainder using mulmod
            uint256 remainder;
            assembly ("memory-safe") {
                remainder := mulmod(a, b, 0x1000000000000000000000000000000000000000000000000)
            }
            // Subtract 256 bit number from 512 bit number
            assembly ("memory-safe") {
                prod1 := sub(prod1, gt(remainder, prod0))
                prod0 := sub(prod0, remainder)
            }


            // Divide [prod1 prod0] by the factors of two (note that this is just 2**96 since the denominator is a power of 2 itself)
            assembly ("memory-safe") {
                // Right shift by n is equivalent and 2 gas cheaper than division by 2^n
                prod0 := shr(192, prod0)
            }
            // Shift in bits from prod1 into prod0. For this we need
            // to flip `twos` such that it is 2**256 / twos.
            // If twos is zero, then it becomes one
            // Note that this is just 2**64 since 2**256 over the fixed denominator (2**192) equals 2**64
            prod0 |= prod1 * 2 ** 64;


            return prod0;
        }
    }
```
https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L472-L528

## Recommendation
To save gas and improve code quality, consider using either the named returns or a return statement.

# GAS-06] DEFINE CONSTRUCTOR AS PAYABLE

## Impact
can save around 10 opcodes and some gas if the constructors are defined as payable.

```solidity
    constructor(IUniswapV3Factory _factory) {
        FACTORY = _factory;
    }
```
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L342-L344

## Recommendation
I suggest marking the constructors as payable to save some gas to make sure it does not lead to any adverse effects in case an upgrade pattern is involved.

# [GAS-07] ABI ENCODE IS LESS EFFICIENT THAN ABI ENCODEPACKED

## Impact
The contract is using `abi.encode()` in the `validateCallback` function. In `abi.encode()`, all elementary types are padded to 32 bytes and dynamic arrays include their length, whereas `abi.encodePacked()` will only use the minimal required memory to encode the data.

```solidity
                                keccak256(abi.encode(features)),
```
https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/libraries/CallbackLib.sol#L43-L43

```solidity
            data = abi.encode(
```
https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L760-L760

```solidity
        bytes memory mintdata = abi.encode(
```
https://github.com//code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L1134-L1134

## Recommendation
Unless explicitly needed , it is recommended to use `abi.encodePacked()` instead of `abi.encode()`.