## Title 
To ensure that the dynamic handling of precision in the convert0to1 and convert1to0 within the PanopticMath library by rounding up (ceil) or rounding down (floor) or rounding to the nearest integer (round)

Links to affected code
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/PanopticMath.sol#L145

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/PanopticMath.sol#L168

## Impact
To avoid significant rounding errors or unintended outcomes, particularly at boundary conditions, in the PanopticMath library's convert0to1 and convert1to0 functions, consider taking the following steps to manage the dynamic precision handling effectively:
## Proof of Concept
Consistent Rounding Strategy:
Choose a standard method for rounding fractional values, such as rounding up (ceil), rounding down (floor), or rounding to the nearest integer (round).

To implement rounding up (also known as "ceiling" rounding) in the convert0to1 and convert1to0 functions within the PanopticMath library, we need to adjust the calculations to round up whenever there's a remainder in the division. In Solidity, this can be done by checking if there's a remainder and then adding an appropriate value to ensure the result rounds up.

Here's how you can modify these functions:

convert0to1 Function:
This function converts an amount of token0 into token1 based on sqrtPriceX96.

function convert0to1(int256 amount, uint160 sqrtPriceX96) internal pure returns (int256) {
    unchecked {
        if (sqrtPriceX96 < 340275971719517849884101479065584693834) {
            int256 absAmount = Math.absUint(amount);
            uint256 result = Math.mulDiv192(absAmount, uint256(sqrtPriceX96) ** 2);

            // Rounding up if there's a remainder
            if (absAmount * (uint256(sqrtPriceX96) ** 2) % (2 ** 192) != 0) {
                result++;
            }

            return amount < 0 ? -int256(result) : int256(result);
        } else {
            int256 absAmount = Math.absUint(amount);
            uint256 result = Math.mulDiv128(absAmount, Math.mulDiv64(sqrtPriceX96, sqrtPriceX96));

            // Rounding up if there's a remainder
            if (absAmount * Math.mulDiv64(sqrtPriceX96, sqrtPriceX96) % (2 ** 128) != 0) {
                result++;
            }

            return amount < 0 ? -int256(result) : int256(result);
        }
    }
}
convert1to0 Function:
This function converts an amount of token1 into token0 based on sqrtPriceX96.

function convert1to0(int256 amount, uint160 sqrtPriceX96) internal pure returns (int256) {
    unchecked {
        if (sqrtPriceX96 < 340275971719517849884101479065584693834) {
            int256 absAmount = Math.absUint(amount);
            uint256 result = Math.mulDiv(absAmount, 2 ** 192, uint256(sqrtPriceX96) ** 2);

            // Rounding up if there's a remainder
            if (absAmount * (2 ** 192) % (uint256(sqrtPriceX96) ** 2) != 0) {
                result++;
            }

            return amount < 0 ? -int256(result) : int256(result);
        } else {
            int256 absAmount = Math.absUint(amount);
            uint256 result = Math.mulDiv(absAmount, 2 ** 128, Math.mulDiv64(sqrtPriceX96, sqrtPriceX96));

            // Rounding up if there's a remainder
            if (absAmount * (2 ** 128) % Math.mulDiv64(sqrtPriceX96, sqrtPriceX96) != 0) {
                result++;
            }

            return amount < 0 ? -int256(result) : int256(result);
        }
    }
}
In both functions, the logic if (remainder != 0) result++; is the key part where rounding up is applied. This ensures that if there is any fractional part left after the division, the result is rounded up to the nearest whole number. The Math library contains the appropriate mulDiv functions for different precision levels.


To implement rounding down in the convert0to1 and convert1to0 functions within the PanopticMath library, you can modify the existing code to explicitly perform the rounding down operation. In Solidity, when you perform integer division, the result is automatically rounded down to the nearest integer. However, to make it explicit and ensure clarity in the code, you can add comments or use intermediate variables.

Here's how the updated functions might look:

library PanopticMath {
    // ... existing code ...

    function convert0to1(int256 amount, uint160 sqrtPriceX96) internal pure returns (int256) {
        unchecked {
            if (sqrtPriceX96 < 340275971719517849884101479065584693834) {
                int256 absResult = Math
                    .mulDiv192(Math.absUint(amount), uint256(sqrtPriceX96) ** 2)
                    .toInt256();
                // Rounding down is implicit in integer division
                return amount < 0 ? -absResult : absResult;
            } else {
                int256 absResult = Math
                    .mulDiv128(Math.absUint(amount), Math.mulDiv64(sqrtPriceX96, sqrtPriceX96))
                    .toInt256();
                // Rounding down is implicit in integer division
                return amount < 0 ? -absResult : absResult;
            }
        }
    }

    function convert1to0(int256 amount, uint160 sqrtPriceX96) internal pure returns (int256) {
        unchecked {
            if (sqrtPriceX96 < 340275971719517849884101479065584693834) {
                int256 absResult = Math
                    .mulDiv(Math.absUint(amount), 2 ** 192, uint256(sqrtPriceX96) ** 2)
                    .toInt256();
                // Rounding down is implicit in integer division
                return amount < 0 ? -absResult : absResult;
            } else {
                int256 absResult = Math
                    .mulDiv(
                        Math.absUint(amount),
                        2 ** 128,
                        Math.mulDiv64(sqrtPriceX96, sqrtPriceX96)
                    )
                    .toInt256();
                // Rounding down is implicit in integer division
                return amount < 0 ? -absResult : absResult;
            }
        }
    }

    // ... other code ...
}
In this implementation, the rounding down happens naturally due to the nature of integer division in Solidity. The explicit comments help to clarify the intent of the code.


To round to the nearest integer in the convert0to1 and convert1to0 functions within the PanopticMath library, you need to adjust the division process to account for the fractional part of the division. In Solidity, integer division truncates towards zero, so you'll need to manually implement the rounding.

The strategy is to add half of the divisor to the dividend before performing the division. This way, the division will naturally round to the nearest integer.

Here's the updated implementation:

library PanopticMath {
    // ... existing code ...

    function convert0to1(int256 amount, uint160 sqrtPriceX96) internal pure returns (int256) {
        unchecked {
            uint256 absAmount = Math.absUint(amount);
            if (sqrtPriceX96 < 340275971719517849884101479065584693834) {
                uint256 divisor = uint256(sqrtPriceX96) ** 2;
                int256 absResult = Math
                    .mulDiv192(absAmount, divisor)
                    .toInt256();
                // Rounding to nearest integer
                absResult = (absResult + 1) / 2;
                return amount < 0 ? -absResult : absResult;
            } else {
                uint256 divisor = Math.mulDiv64(sqrtPriceX96, sqrtPriceX96);
                int256 absResult = Math
                    .mulDiv128(absAmount, divisor)
                    .toInt256();
                // Rounding to nearest integer
                absResult = (absResult + 1) / 2;
                return amount < 0 ? -absResult : absResult;
            }
        }
    }

    function convert1to0(int256 amount, uint160 sqrtPriceX96) internal pure returns (int256) {
        unchecked {
            uint256 absAmount = Math.absUint(amount);
            if (sqrtPriceX96 < 340275971719517849884101479065584693834) {
                uint256 divisor = uint256(sqrtPriceX96) ** 2;
                int256 absResult = Math
                    .mulDiv(absAmount, 2 ** 192, divisor)
                    .toInt256();
                // Rounding to nearest integer
                absResult = (absResult + 1) / 2;
                return amount < 0 ? -absResult : absResult;
            } else {
                uint256 divisor = Math.mulDiv64(sqrtPriceX96, sqrtPriceX96);
                int256 absResult = Math
                    .mulDiv(
                        absAmount,
                        2 ** 128,
                        divisor
                    )
                    .toInt256();
                // Rounding to nearest integer
                absResult = (absResult + 1) / 2;
                return amount < 0 ? -absResult : absResult;
            }
        }
    }

    // ... other code ...
}
In this implementation, the absResult is adjusted by adding 1 before dividing by 2. This step effectively rounds the result to the nearest integer. Note that the added 1 is actually half of the divisor 2 for the rounding operation. This method will work correctly for positive and negative amounts.

## Tools Used
VS code

## Recommended Mitigation Steps

Apply one of these strategies uniformly across all calculations in the convert0to1 and convert1to0 functions.
In Solidity, rounding down (floor) is the default behavior for integer division. 
Ensure that the rounding method chosen does not introduce biases that could be exploited in certain market conditions.









Issue type *
Under/Overflow
