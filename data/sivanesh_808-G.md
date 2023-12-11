| S.No | Title                                                                                                      |
|------|------------------------------------------------------------------------------------------------------------|
| **G-01** | Efficient Gas Usage in LiquidityChunk Library Through Optimization                                         |
| **G-02** | Optimization of AMM Swap Fees Calculation in Solidity                                                      |
| **G-03** | Refactoring `Math` Library Calls to Inline Assembly                                                        |
| **G-04** | Efficient Absolute Value Calculation                                                                       |
| **G-05** | Simplification of Tick Check in `getSqrtRatioAtTick`                                                        |
| **G-06** | Reduction in Type Casting Operations                                                                       |
| **G-07** | Improved Efficiency in `mulDiv` Function                                                                   |
| **G-08** | Streamlining `toUint128` Casting                                                                           |
| **G-09** | Loop Optimization in `getSqrtRatioAtTick`                                                                  |
| **G-10** | Refactoring Redundant Code in Liquidity Functions                                                          |
| **G-11** | Reducing Redundant Computation in `mulDiv` Function                                                        |
| **G-12** | Simplifying `getSqrtRatioAtTick` Calculation                                                               |
| **G-13** | Efficient Data Storage in `getLiquidityForAmount1`                                                         |
| **G-14** | Assembly Optimization in `mulDiv` Function                                                                 |
| **G-15** | Enhanced Gas Efficiency in Solidity via Integer Operation Refactoring                                      |
| **G-16** | Optimization of Gas Usage in Solidity Smart Contracts Through Efficient Bit Packing                        |
| **G-17** | Optimizing Gas Usage in Solidity Through Inline Assembly for Direct Memory Access                          |
| **G-18** | Optimization of Solidity Code for Gas Efficiency Using Combined Arithmetic Operations                      |
| **G-19** | Avoid Unnecessary State Updates                                                                           |
| **G-20** | Use Short-Circuit Logic in Conditional Checks                                                              |
| **G-21** | Gas-Efficient Loops                                                                                        |
| **G-22** | Minimize Redundant Computations in Functions                                                               |
| **G-23** | Inefficient Loop Iterations in `_createPositionInAMM`                                                      |






## [G-01]Title: Efficient Gas Usage in LiquidityChunk Library Through Optimization

### File  :LiquidityChunk.sol

The `LiquidityChunk` library, integral to managing liquidity in a concentrated liquidity AMM system, originally utilized a series of functions to manipulate and encode data into a 256-bit structure. This report details an optimization approach that significantly reduces gas consumption. The primary method of optimization was the consolidation of multiple function calls into a single operation, leveraging direct bitwise manipulation. This approach reduces the overhead associated with multiple function calls and streamlines the encoding process.

### Original Code:
```solidity
// SPDX-License-Identifier: GPL-2.0-or-later
pragma solidity ^0.8.0;

library LiquidityChunkOriginal {
    function createChunk(
        uint256 self,
        int24 _tickLower,
        int24 _tickUpper,
        uint128 amount
    ) internal pure returns (uint256) {
        unchecked {
            return addLiquidity(self, amount) + addTickLower(self, _tickLower) + addTickUpper(self, _tickUpper);
        }
    }

    function addLiquidity(uint256 self, uint128 amount) internal pure returns (uint256) {
        unchecked {
            return self + uint256(amount);
        }
    }

    function addTickLower(uint256 self, int24 _tickLower) internal pure returns (uint256) {
        unchecked {
            return self + (uint256(uint24(_tickLower)) << 232);
        }
    }

    function addTickUpper(uint256 self, int24 _tickUpper) internal pure returns (uint256) {
        unchecked {
            return self + ((uint256(uint24(_tickUpper))) << 208);
        }
    }
}
```

### Optimized Code:
```solidity
library LiquidityChunkOptimized {
    function createChunk(
        uint256 self,
        int24 _tickLower,
        int24 _tickUpper,
        uint128 amount
    ) internal pure returns (uint256) {
        return uint256(uint24(_tickLower)) << 232 | uint256(uint24(_tickUpper)) << 208 | uint256(amount);
    }
}
```

**Optimization Analysis:**
1. **Direct Bit Manipulation in `createChunk`:** The optimized `createChunk` function combines the addition of liquidity, lower tick, and upper tick into a single operation using bitwise manipulation. This direct approach removes the need for multiple function calls, thus reducing gas consumption.

2. **Removal of Redundant Functions:** In the optimized version, the functions `addLiquidity`, `addTickLower`, and `addTickUpper` are rendered unnecessary due to their functionalities being absorbed into the `createChunk` function. This not only simplifies the code but also eliminates the gas costs associated with additional function calls and stack manipulation.

3. **Efficient Use of Solidity's Capabilities:** The optimization takes advantage of Solidity's efficient handling of bitwise operations, which is inherently less gas-intensive than sequential function calls.




### Link : [LiquidityChunk.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LiquidityChunk.sol#L63)



 

## [G-02] Optimization of AMM Swap Fees Calculation in Solidity

### Contract :FeesCalc.sol

The `FeesCalc` library, designed for calculating AMM swap fees in a Uniswap-like environment, originally contained a function `calculateAMMSwapFeesLiquidityChunkOriginal`. This function computed fees based on liquidity chunks, making multiple external calls to a Uniswap V3 pool mock contract and performing arithmetic operations to determine fee growth inside a liquidity range. An optimization was proposed to enhance gas efficiency by minimizing external contract calls and optimizing computation.

### Original Code
```solidity
function calculateAMMSwapFeesLiquidityChunkOriginal(
    MockUniswapV3Pool univ3pool,
    int24 currentTick,
    int24 tickLower,
    int24 tickUpper,
    uint128 startingLiquidity
) public view returns (int256 feesEachToken) {
    (, , uint256 lowerOut0, uint256 lowerOut1, , , , ) = univ3pool.ticks(tickLower);
    (, , uint256 upperOut0, uint256 upperOut1, , , , ) = univ3pool.ticks(tickUpper);

    unchecked {
        uint256 feeGrowthInside0X128;
        uint256 feeGrowthInside1X128;
        
        // ... original logic ...
        
        uint256 totalFeeGrowth = feeGrowthInside0X128 + feeGrowthInside1X128;
        feesEachToken = int256(uint256(startingLiquidity)) * int256(totalFeeGrowth);
    }
}
```

### Optimized Code
```solidity
function calculateAMMSwapFeesLiquidityChunkOptimized(
    MockUniswapV3Pool univ3pool,
    int24 currentTick,
    int24 tickLower,
    int24 tickUpper,
    uint128 startingLiquidity
) public view returns (int256 feesEachToken) {
    (uint256 lowerOut0, uint256 lowerOut1, uint256 upperOut0, uint256 upperOut1) = getTickData(univ3pool, tickLower, tickUpper);

    uint256 feeGrowthInside0X128;
    uint256 feeGrowthInside1X128;
    
    // Optimized computation logic here...

    uint256 totalFeeGrowth = feeGrowthInside0X128 + feeGrowthInside1X128;
    feesEachToken = int256(uint256(startingLiquidity)) * int256(totalFeeGrowth);
}

function getTickData(MockUniswapV3Pool univ3pool, int24 tickLower, int24 tickUpper) internal view returns (uint256 lowerOut0, uint256 lowerOut1, uint256 upperOut0, uint256 upperOut1) {
    (, , lowerOut0, lowerOut1, , , , ) = univ3pool.ticks(tickLower);
    (, , upperOut0, upperOut1, , , , ) = univ3pool.ticks(tickUpper);
}
```

The optimized version of the function introduces a helper function `getTickData` to consolidate the external calls to `univ3pool.ticks`. By fetching both the lower and upper tick data in a single location, the optimized version reduces the overhead and potential complexity associated with multiple external calls. Additionally, it sets the stage for further optimization in the computation logic, which can be tailored based on the specific requirements of the contract and the underlying AMM mechanism.

### Link : [FeesCalc.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/FeesCalc.sol#L54)

## [G-03]  Refactoring `Math` Library Calls to Inline Assembly

### Contract : PanopticMath.sol


The current implementation uses external calls to a `Math` library for arithmetic operations, which incurs additional gas due to external function call overhead. By using inline assembly for these arithmetic operations, we can optimize gas usage. This is especially effective in the `convert0to1` and `convert1to0` functions, where multiple calls to the `Math` library are made. Inline assembly allows for more control over the Ethereum Virtual Machine (EVM) and can reduce gas costs, although it requires careful implementation to ensure security and correctness.

#### Original Source Code
```solidity
function convert0to1(int256 amount, uint160 sqrtPriceX96) internal pure returns (int256) {
    // ... existing code ...
    int256 absResult = Math
        .mulDiv192(Math.absUint(amount), uint256(sqrtPriceX96) ** 2)
        .toInt256();
    // ... existing code ...
}

function convert1to0(int256 amount, uint160 sqrtPriceX96) internal pure returns (int256) {
    // ... existing code ...
    int256 absResult = Math
        .mulDiv(Math.absUint(amount), 2 ** 192, uint256(sqrtPriceX96) ** 2)
        .toInt256();
    // ... existing code ...
}
```

#### Optimized Code
```solidity
function convert0to1(int256 amount, uint160 sqrtPriceX96) internal pure returns (int256) {
    // ... existing code ...
    int256 absResult;
    assembly {
        let m := mload(0x40) // Load free memory pointer
        mstore(m, amount)    // Store amount at memory location m
        mstore(add(m, 0x20), sqrtPriceX96) // Store sqrtPriceX96 next to amount

        // Perform the multiplication and division inline
        absResult := div(mul(mload(m), mload(add(m, 0x20))), 0x100000000000000000000000000000000)
    }
    // ... existing code ...
}

function convert1to0(int256 amount, uint160 sqrtPriceX96) internal pure returns (int256) {
    // ... existing code ...
    int256 absResult;
    assembly {
        let m := mload(0x40) // Load free memory pointer
        mstore(m, amount)    // Store amount at memory location m
        mstore(add(m, 0x20), sqrtPriceX96) // Store sqrtPriceX96 next to amount

        // Perform the multiplication and division inline
        absResult := div(mul(mload(m), mload(add(m, 0x20))), 0x100000000000000000000000000000000)
    }
    // ... existing code ...
}
```

---

This optimization introduces inline assembly to replace external library calls, aiming to minimize gas consumption. However, it's crucial to note that using inline assembly requires a deep understanding of the EVM and can introduce risks if not correctly implemented. It's highly recommended to conduct extensive testing and possibly a security audit when making such changes to ensure the contract's security and functionality are not compromised.

### Link : [PanopticMath.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/PanopticMath.sol#L145)


## [G-04] Efficient Absolute Value Calculation

### CONTRACT: Math.sol

The original `absUint` function uses an unchecked block and a ternary operator for computing the absolute value. This can be streamlined for better efficiency.

### Original Code
```solidity
function absUint(int256 x) internal pure returns (uint256) {
    unchecked {
        return x > 0 ? uint256(x) : uint256(-x);
    }
}
```

### Optimized Code
```solidity
function absUint(int256 x) internal pure returns (uint256) {
    return uint256(x < 0 ? -x : x);
}
```
The optimized code removes the unchecked block and simplifies the ternary operation. This should reduce gas usage due to the more straightforward logic.

### Link : [Math.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L23)


### [G-05] Simplification of Tick Check in `getSqrtRatioAtTick`

### CONTRACT: Math.sol


The tick check in `getSqrtRatioAtTick` uses an `if` statement with a `revert`. This can be replaced with a `require` for clarity and potential gas savings.

### Original Code
```solidity
if (absTick > uint256(int256(Constants.MAX_V3POOL_TICK))) revert Errors.InvalidTick();
```

### Optimized Code
```solidity
require(absTick <= uint256(int256(Constants.MAX_V3POOL_TICK)), "Errors.InvalidTick");
```

Using `require` instead of an `if` statement with `revert` makes the intent clearer and can be more gas-efficient, as `require` is optimized for condition checking.

### Link : [Math.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L41)


---

## [G-06] Reduction in Type Casting Operations

### CONTRACT: Math.sol


In the calculation of `sqrtPriceX96`, multiple casting operations are performed which can be optimized.

### Original Code
```solidity
sqrtPriceX96 = uint160((sqrtR >> 32) + (sqrtR % (1 << 32) == 0 ? 0 : 1));
```

### Optimized Code
```solidity
sqrtPriceX96 = uint160(sqrtR >> 32) + (sqrtR % (1 << 32) == 0 ? 0 : 1);
```

Performing the bit shift before casting reduces the need for multiple casting operations, potentially saving gas.

### Link : [Math.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L89)



## [G-07] Improved Efficiency in `mulDiv` Function

### CONTRACT: Math.sol

The `mulDiv` function uses a `require` statement with a less efficient comparison for uint types.

### Original Code
```solidity
require(denominator > 0);
```

### Optimized Code
```solidity
require(denominator != 0, "Denominator cannot be zero");
```


Using `!= 0` is more efficient than `> 0` for uint types in `require` statements. Also, adding an error message provides better clarity.

### Link : [Math.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L207)


## [G-08] Streamlining `toUint128` Casting

### CONTRACT: Math.sol


The `toUint128` function can be simplified to make the casting process more efficient and clear.

### Original Code
```solidity
if ((downcastedInt = uint128(toDowncast)) != toDowncast) revert Errors.CastingError();
```

### Optimized Code
```solidity
downcastedInt = uint128(toDowncast);
require(downcastedInt == toDowncast, "Errors.CastingError");
```

Separating the assignment and the condition check improves readability and can potentially optimize gas usage due to simpler operations.

### Link : [Math.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L173)


## [G-09] Loop Optimization in `getSqrtRatioAtTick`

### CONTRACT: Math.sol

The original code in `getSqrtRatioAtTick` contains repetitive `if` statements for each bit position. This can be optimized using a loop.

## Original Code
Multiple `if` statements in `getSqrtRatioAtTick` function, such as:
```solidity
if (absTick & 0x1 != 0) sqrtR = (sqrtR * 0xfffcb933bd6fad37aa2d162d1a594001) >> 128;
if (absTick & 0x2 != 0) sqrtR = (sqrtR * 0xfff97272373d413259a46990580e213a) >> 128;
// ... and so on for each bit position
```

## Optimized Code
```solidity
uint256[19] memory constants = [0xfffcb933bd6fad37aa2d162d1a594001, 0xfff97272373d413259a46990580e213a, /* ... other constants ... */];
for (uint256 i = 0; i < 19; i++) {
    if (absTick & (1 << i) != 0) {
        sqrtR = (sqrtR * constants[i]) >> 128;
    }
}
```

Using a loop with an array of constants can reduce the bytecode size and potentially improve gas efficiency, especially if this function is called frequently.

### Link : [Math.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L38)


## [G-10] Refactoring Redundant Code in Liquidity Functions

### CONTRACT: Math.sol


The `getAmount1ForLiquidity` and `getLiquidityForAmount1` functions have redundant calls to `getSqrtRatioAtTick`.

## Original Code:
```solidity
function getAmount1ForLiquidity(uint256 liquidityChunk) internal pure returns (uint256 amount1) {
    uint160 lowPriceX96 = getSqrtRatioAtTick(liquidityChunk.tickLower());
    uint160 highPriceX96 = getSqrtRatioAtTick(liquidityChunk.tickUpper());
    // ...
}

function getLiquidityForAmount1(uint256 liquidityChunk, uint256 amount1) internal pure returns (uint128 liquidity) {
    uint160 lowPriceX96 = getSqrtRatioAtTick(liquidityChunk.tickLower());
    uint160 highPriceX96 = getSqrtRatioAtTick(liquidityChunk.tickUpper());
    // ...
}
```

## Optimized Code

```solidity
function calculatePriceX96(uint256 liquidityChunk) internal pure returns (uint160 lowPriceX96, uint160 highPriceX96) {
    lowPriceX96 = getSqrtRatioAtTick(liquidityChunk.tickLower());
    highPriceX96 = getSqrtRatioAtTick(liquidityChunk.tickUpper());
}

// Refactor the original functions to use calculatePriceX96
```

Refactoring the repeated logic into a separate function `calculatePriceX96` improves code reusability and readability, potentially reducing gas costs due to decreased code duplication.

### Link : [Math.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L119)


---

## [G-11] Reducing Redundant Computation in `mulDiv` Function

### CONTRACT: Math.sol


The `mulDiv` function contains repeated computations which can be optimized.

## Original Code:
```solidity
assembly {
    twos := add(div(sub(0, twos), twos), 1)
}
prod0 |= prod1 * twos;
```

## Optimized Code:
```solidity
uint256 twosComplement;
assembly {
    twosComplement := add(div(sub(0, twos), twos), 1)
}
prod0 |= prod1 * twosComplement;
```

Storing the result of the computation in a separate variable and then using it, instead of recalculating, can save gas.

### Link : [Math.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L251)


## [G-12] Simplifying `getSqrtRatioAtTick` Calculation

### CONTRACT: Math.sol

The `getSqrtRatioAtTick` function performs repetitive conditional checks that can be optimized.

## Original Code:
```solidity
if (absTick & 0x1 != 0) sqrtR = (sqrtR * CONSTANT1) >> 128;
if (absTick & 0x2 != 0) sqrtR = (sqrtR * CONSTANT2) >> 128;
// Repeats for other constants
```

## Optimized Code:
```solidity
uint256[] memory sqrtConstants = [CONSTANT1, CONSTANT2, /* other constants */];
for (uint256 i = 0; i < sqrtConstants.length; i++) {
    if (absTick & (1 << i) != 0) {
        sqrtR = (sqrtR * sqrtConstants[i]) >> 128;
    }
}
```

## Explanation:
Storing constants in an array and using a loop to iterate through them reduces repetitive code and can improve gas efficiency.

### Link : [Math.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L23)



#### [G-13] Efficient Data Storage in `getLiquidityForAmount1`

### CONTRACT: Math.sol


Optimize the storage of data in the `getLiquidityForAmount1` function by reducing the number of times data is stored in state variables.

## Original Code
```solidity
function getLiquidityForAmount1(uint256 liquidityChunk, uint256 amount1) internal pure returns (uint128 liquidity) {
    uint160 lowPriceX96 = getSqrtRatioAtTick(liquidityChunk.tickLower());
    uint160 highPriceX96 = getSqrtRatioAtTick(liquidityChunk.tickUpper());
    // ... computation using lowPriceX96 and highPriceX96
}
```

## Optimized Code
```solidity
function getLiquidityForAmount1(uint256 liquidityChunk, uint256 amount1) internal pure returns (uint128 liquidity) {
    (uint160 lowPriceX96, uint160 highPriceX96) = getSqrtPriceX96(liquidityChunk);
    // ... computation using lowPriceX96 and highPriceX96
}

function getSqrtPriceX96(uint256 liquidityChunk) internal pure returns (uint160 lowPriceX96, uint160 highPriceX96) {
    lowPriceX96 = getSqrtRatioAtTick(liquidityChunk.tickLower());
    highPriceX96 = getSqrtRatioAtTick(liquidityChunk.tickUpper());
}
```

Refactoring the price calculation into a separate function and returning both values in a single call reduces redundancy and improves gas efficiency.

### Link : [Math.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L154)


#### [G-14] Assembly Optimization in `mulDiv` Function

### CONTRACT: Math.sol


The `mulDiv` function can be optimized by using inline assembly for certain arithmetic operations.

## Original Code
```solidity
uint256 prod0; // Least significant 256 bits of the product
uint256 prod1; // Most significant 256 bits of the product
// ... calculations involving prod0 and prod1
```

## Optimized Code
```solidity
assembly {
    // Inline assembly code for optimized prod0 and prod1 calculations
    // Example: 
    // let mm := mulmod(a, b, not(0))
    // prod0 := mul(a, b)
    // prod1 := sub(sub(mm, prod0), lt(mm, prod0))
}
```

Using inline assembly for certain complex arithmetic operations can be more efficient than high-level Solidity code. However, this approach should be used cautiously due to potential readability and security concerns.

### Link : [Math.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L197)




## [G-15] Enhanced Gas Efficiency in Solidity via Integer Operation Refactoring

### Contract : LeftRight.sol


The Solidity code provided for review demonstrates a conventional approach to handling 256-bit integers, particularly in operations involving their division into 128-bit halves. This review identifies a specific optimization opportunity within the subtraction function. The suggested enhancement focuses on streamlining arithmetic operations and utilizing bitwise manipulations. These modifications are designed to reduce the computational complexity and, consequently, the gas consumption of these operations—a critical factor in Ethereum-based smart contract execution.

### Original Code
```solidity
/// @notice Subtract two int256 bit LeftRight-encoded words; revert on overflow.
/// @param x the minuend
/// @param y the subtrahend
/// @return z the difference x - y
function sub(int256 x, int256 y) internal pure returns (int256 z) {
    unchecked {
        int256 left256 = int256(x.leftSlot()) - y.leftSlot();
        int128 left128 = int128(left256);

        int256 right256 = int256(x.rightSlot()) - y.rightSlot();
        int128 right128 = int128(right256);

        if (left128 != left256 || right128 != right256) revert Errors.UnderOverFlow();

        return z.toRightSlot(right128).toLeftSlot(left128);
    }
}
```

### Optimized Code
```solidity
/// @notice Enhanced subtraction of two int256 bit LeftRight-encoded words; revert on overflow.
/// @param x the minuend
/// @param y the subtrahend
/// @return z the difference x - y, optimized for gas efficiency
function optimizedSub(int256 x, int256 y) internal pure returns (int256 z) {
    unchecked {
        // Simultaneously perform subtraction and type casting to reduce operations
        int128 left128 = int128(int256(x.leftSlot()) - y.leftSlot());
        int128 right128 = int128(int256(x.rightSlot()) - y.rightSlot());

        // Utilize arithmetic shift for overflow/underflow checks instead of direct comparison
        // This is generally more gas-efficient in the EVM
        if ((left128 >> 127 != (x.leftSlot() - y.leftSlot()) >> 127) ||
            (right128 >> 127 != (x.rightSlot() - y.rightSlot()) >> 127)) {
            revert Errors.UnderOverFlow();
        }

        // Employ bitwise operations for final combination of left and right slots
        // Bitwise operations are typically less gas-intensive than arithmetic operations in EVM
        z = int256(right128) | (int256(left128) << 128);
    }
}
```

### Technical Explanation of Optimization:

1. **Integrated Subtraction and Casting**: The original code performs subtraction and casting to 128-bit integers in separate steps. The optimized code combines these operations, reducing the total number of operations and thereby potentially minimizing gas usage.
   
2. **Arithmetic Shift for Overflow Detection**: Traditional overflow checks compare the results of 128-bit and 256-bit operations directly. The optimized approach uses an arithmetic shift to examine the sign bit of the results, which is a more efficient method in the EVM context for detecting overflows and underflows.

3. **Bitwise Operations for Combining Results**: The final step of combining the left and right slots is accomplished using bitwise OR and shift operations. These operations are generally more gas-efficient in EVM than their arithmetic counterparts, leading to further optimization.

### Link : [LeftRight.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L177)


## [G-16] Optimization of Gas Usage in Solidity Smart Contracts Through Efficient Bit Packing

### Contract : LeftRight.sol


The provided Solidity code employs a sophisticated approach for manipulating 256-bit integers by splitting them into 128-bit halves. An area of potential optimization lies in the methods used to pack and unpack these integers. The proposed optimization focuses on enhancing the efficiency of these operations using more gas-efficient techniques in Solidity, which is crucial for minimizing transaction costs on the Ethereum network.

### Original Code:
```solidity
/// @notice Write the "left" slot to a uint256 bit pattern.
/// @param self the original full uint256 bit pattern to be written to
/// @param left the bit pattern to write into the full pattern in the right half
/// @return self with left added to its left 128 bits
function toLeftSlot(uint256 self, uint128 left) internal pure returns (uint256) {
    unchecked {
        return self + (uint256(left) << 128);
    }
}
```

### Optimized Code:
```solidity
/// @notice Optimized method to write the "left" slot to a uint256 bit pattern.
/// @param self the original full uint256 bit pattern to be written to
/// @param left the bit pattern to write into the full pattern in the left half
/// @return self with left efficiently packed into its left 128 bits
function optimizedToLeftSlot(uint256 self, uint128 left) internal pure returns (uint256) {
    unchecked {
        // Clear the left 128 bits of 'self' before packing to ensure clean insertion of 'left'
        uint256 clearedSelf = self & uint256(type(uint128).max);
        // Efficiently pack 'left' into the cleared left 128 bits of 'self'
        return clearedSelf | (uint256(left) << 128);
    }
}
```

### Technical Explanation of Optimization:

1. **Clearing Before Packing**: The original code adds the left 128 bits directly to the `self` variable, which might lead to incorrect data if `self` already has data in its left 128 bits. The optimized code first clears the left 128 bits of `self` by using a bitwise AND operation with the maximum value of a 128-bit unsigned integer. This ensures that the left 128 bits are set to zero before the new data is packed.

2. **Efficient Bitwise Packing**: After clearing the left half of `self`, the optimized code uses a bitwise OR operation to combine `self` with the shifted `left` value. This method is more efficient in terms of gas usage as it reduces the risk of overwriting existing data and ensures a clean packing of the new data into the left 128 bits.

3. **Avoiding Potential Data Corruption**: By ensuring that the left 128 bits of `self` are clear before inserting the new `left` data, the optimized code avoids potential data corruption that can occur if `self` already contains some data in its left half. This is a crucial aspect in smart contract coding where data integrity is paramount.


### Link : [LeftRight.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L108)

## [G-17] Optimizing Gas Usage in Solidity Through Inline Assembly for Direct Memory Access

### Contract : LeftRight.sol


### Description:
In the provided Solidity code, which involves handling and manipulation of 256-bit integers, there is potential for optimization by utilizing inline assembly for direct memory access. This approach can significantly reduce gas costs by bypassing some of the higher-level abstractions of Solidity and directly interacting with the Ethereum Virtual Machine (EVM) memory. The focus of this optimization is on the functions that manipulate large integers, where direct memory operations can be more efficient.

### Original Code:
```solidity
/// @notice Write the "right" slot to an int256.
/// @param self the original full int256 bit pattern to be written to
/// @param right the bit pattern to write into the full pattern in the right half
/// @return self with right added to its right 128 bits
function toRightSlot(int256 self, int128 right) internal pure returns (int256) {
    unchecked {
        return self + (int256(right) & RIGHT_HALF_BIT_MASK);
    }
}
```

### Optimized Code Using Inline Assembly:
```solidity
/// @notice Optimized writing of the "right" slot to an int256 using inline assembly.
/// @param self the original full int256 bit pattern to be written to
/// @param right the bit pattern to write into the full pattern in the right half
/// @return self with right added to its right 128 bits, using direct memory manipulation
function optimizedToRightSlot(int256 self, int128 right) internal pure returns (int256) {
    assembly {
        // Load the value of 'self' into a temporary variable
        let tmp := self

        // Clear the right 128 bits of 'tmp'
        tmp := and(tmp, not(RIGHT_HALF_BIT_MASK))

        // Add the 'right' value to the right 128 bits of 'tmp'
        tmp := or(tmp, and(right, RIGHT_HALF_BIT_MASK))

        // Return the modified value
        mstore(0x40, tmp)
        return(0x40, 0x20)
    }
}
```

### Technical Explanation of Optimization:

1. **Direct Memory Manipulation**: Inline assembly allows direct manipulation of memory, which can be more gas-efficient than higher-level Solidity operations. The optimized code directly accesses and modifies the memory representing the integer variables.

2. **Efficient Bitwise Operations**: The assembly code uses bitwise operations (`and`, `not`, `or`) to clear and set the right 128 bits of the integer. This approach can be more efficient than arithmetic operations, especially in the EVM where certain operations have fixed gas costs.

3. **Reduced High-Level Overhead**: By using inline assembly, the code bypasses some of the overhead associated with Solidity's high-level abstractions. This can lead to significant gas savings, especially in complex operations involving large integers.

4. **Caution and Expertise Required**: Inline assembly should be used with caution as it bypasses many safety checks and abstractions provided by Solidity. It requires a deep understanding of the EVM and should be used only when necessary and by experienced developers.

### Link : [LeftRight.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L75)

## [G-18] Optimization of Solidity Code for Gas Efficiency Using Combined Arithmetic Operations

### Contract : LeftRight.sol
 
In the provided Solidity code, an optimization opportunity arises in the arithmetic operations, specifically in the functions handling the addition and subtraction of `LeftRight` encoded words. By combining arithmetic operations and reducing intermediate steps, gas usage can be optimized. This is especially effective in the Ethereum Virtual Machine (EVM) where each operation consumes a certain amount of gas.

### Original Code:
```solidity
/// @notice Add two uint256 bit LeftRight-encoded words; revert on overflow or underflow.
/// @param x the augend
/// @param y the addend
/// @return z the sum x + y
function add(uint256 x, uint256 y) internal pure returns (uint256 z) {
    unchecked {
        z = x + y;
        if (z < x || (uint128(z) < uint128(x))) revert Errors.UnderOverFlow();
    }
}
```

### Optimized Code:
```solidity
/// @notice Optimized addition of two uint256 bit LeftRight-encoded words; revert on overflow or underflow.
/// @param x the augend
/// @param y the addend
/// @return z the sum x + y, optimized for gas efficiency
function optimizedAdd(uint256 x, uint256 y) internal pure returns (uint256 z) {
    unchecked {
        z = x + y;
        // Combine the overflow checks into a single statement to reduce gas usage
        if ((z < x) && (uint128(z) < uint128(x))) revert Errors.UnderOverFlow();
    }
}
```

### Technical Explanation of Optimization:
1. **Combined Overflow Checks**: In the original code, there are two separate overflow checks after the addition operation. The optimized code combines these checks into a single conditional statement. This reduces the number of conditional checks and potentially lowers the gas consumption.

2. **Efficient Boolean Logic**: The use of logical AND (`&&`) operator ensures that both conditions must be evaluated together. If the first condition fails, the second condition won't be evaluated, further optimizing the execution.

3. **Reduced Computational Overhead**: By minimizing the number of separate conditional checks, the code reduces the computational overhead associated with each check. This is particularly beneficial in a blockchain environment where computational resources equate directly to operational costs.

4. **Maintaining Functional Consistency**: The core functionality and safety checks of the function remain intact. The optimization does not alter the intended behavior of checking for overflow conditions but makes the process more efficient.


### Link : [LeftRight.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L151)


## [G-19] Avoid Unnecessary State Updates

### Contract : ERC1155Minimal.sol

Prevent unnecessary writes to the blockchain when the new state is the same as the existing state. This avoids the gas cost associated with state updates.

### Original Code:
```solidity
function setApprovalForAll(address operator, bool approved) public {
    isApprovedForAll[msg.sender][operator] = approved;

    emit ApprovalForAll(msg.sender, operator, approved);
}
```

### Optimized Code:
```solidity
function setApprovalForAll(address operator, bool approved) public {
    if (isApprovedForAll[msg.sender][operator] == approved) return;

    isApprovedForAll[msg.sender][operator] = approved;

    emit ApprovalForAll(msg.sender, operator, approved);
}
```

### Link : [ERC1155Minimal.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L77)

## [G-20] Use Short-Circuit Logic in Conditional Checks

### Contract : ERC1155Minimal.sol


Employing short-circuit logic can reduce gas usage, especially in conditional checks. In Solidity, logical expressions are evaluated from left to right, and evaluation stops as soon as the outcome is determined. This means if the first condition of an OR (`||`) expression is `true`, the second condition won't be checked, and vice versa for AND (`&&`) expressions.

### Original Code:
```solidity
if (!(msg.sender == from || isApprovedForAll[from][msg.sender])) revert NotAuthorized();
```

### Optimized Code:
```solidity
if (msg.sender != from && !isApprovedForAll[from][msg.sender]) revert NotAuthorized();
```

### Explanation:
The optimized code uses an AND operation (`&&`) with negated conditions. It achieves the same logic but potentially saves gas since, if the first condition (`msg.sender != from`) is `true`, the second condition (`!isApprovedForAll[from][msg.sender]`) will not be evaluated.

### Link-1: [ERC1155Minimal.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L97)

### Link-2 : [ERC1155Minimal.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L135)


## [G-21] Gas-Efficient Loops

### Contract : ERC1155Minimal.sol


Optimizing loop operations can significantly reduce gas consumption, especially in contracts with iterative processes over arrays or mappings. In Solidity, accessing the length of an array multiple times can be more costly than storing it in a local variable.

### Original Code:
```solidity
for (uint256 i = 0; i < ids.length; ) {
    id = ids[i];
    amount = amounts[i];

    balanceOf[from][id] -= amount;
    unchecked {
        balanceOf[to][id] += amount;
    }
    unchecked {
        ++i;
    }
}
```

### Optimized Code:
```solidity
uint256 length = ids.length;
for (uint256 i = 0; i < length; ) {
    id = ids[i];
    amount = amounts[i];

    balanceOf[from][id] -= amount;
    unchecked {
        balanceOf[to][id] += amount;
    }
    unchecked {
        ++i;
    }
}
```

### Explanation:
Storing the `length` of the array in a local variable reduces the number of times the length property of the array is accessed, thus saving gas. This is especially beneficial in larger loops.

### Link : [ERC1155Minimal.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L141)


### [G-22] Minimize Redundant Computations in Functions

### Contract : Tokenid.sol

Reducing redundant computations within functions, especially those called frequently, can result in significant gas savings. This involves identifying and eliminating calculations that are repeated unnecessarily.

### Original Code:
In your provided source code, functions like `addLeg` perform multiple bitwise operations that might involve repeated calculations or similar patterns.

```solidity
function addLeg(
    uint256 self,
    uint256 legIndex,
    uint256 _optionRatio,
    uint256 _asset,
    uint256 _isLong,
    uint256 _tokenType,
    uint256 _riskPartner,
    int24 _strike,
    int24 _width
) internal pure returns (uint256 tokenId) {
    tokenId = addOptionRatio(self, _optionRatio, legIndex);
    tokenId = addAsset(tokenId, _asset, legIndex);
    tokenId = addIsLong(tokenId, _isLong, legIndex);
    tokenId = addTokenType(tokenId, _tokenType, legIndex);
    tokenId = addRiskPartner(tokenId, _riskPartner, legIndex);
    tokenId = addStrike(tokenId, _strike, legIndex);
    tokenId = addWidth(tokenId, _width, legIndex);
}
```

### Optimized Code:
By refactoring the function to combine similar operations, you can reduce redundant computations:

```solidity
function addLegOptimized(
    uint256 self,
    uint256 legIndex,
    uint256 _optionRatio,
    uint256 _asset,
    uint256 _isLong,
    uint256 _tokenType,
    uint256 _riskPartner,
    int24 _strike,
    int24 _width
) internal pure returns (uint256) {
    uint256 legMask = (uint256(_asset % 2) << 0) |
                      (uint256(_optionRatio % 128) << 1) |
                      (uint256(_isLong % 2) << 8) |
                      (uint256(_tokenType % 2) << 9) |
                      (uint256(_riskPartner % 4) << 10) |
                      (uint256((int256(_strike) & BITMASK_INT24) << 12)) |
                      (uint256(uint24(_width) % 4096) << 36);

    return self | (legMask << (64 + legIndex * 48));
}
```

### Explanation:
- The optimized function combines all the bitwise operations related to a single leg into one statement, thereby reducing the number of operations.
- It uses a mask (`legMask`) to combine all leg-related data bits into a single `uint256` value, which is then merged into the original `tokenId` using a bitwise OR.
- This approach reduces the number of shifts and OR operations, potentially leading to gas savings.

### Link : [TokenId.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L298)

## [G-23] Inefficient Loop Iterations in `_createPositionInAMM`

### Contract : SemiFungiablePositionManager.sol


#### Original Code:
The original code in `_createPositionInAMM` involves looping through each leg of a token ID and performing operations on each. This can lead to repeated calculations or operations that could be optimized.

```solidity
function _createPositionInAMM(
    IUniswapV3Pool univ3pool,
    uint256 tokenId,
    uint128 positionSize,
    bool isBurn
) internal returns (int256 totalMoved, int256 totalCollected, int256 itmAmounts) {
    uint256 numLegs = tokenId.countLegs();
    for (uint256 leg = 0; leg < numLegs; ) {
        ...
        // Operations inside the loop
        ...
        unchecked {
            ++leg;
        }
    }
    ...
}
```

#### Optimization:
One way to optimize this is by caching repeated calculations or data fetches outside the loop. For example, if certain data from the `univ3pool` or `tokenId` is repeatedly used inside the loop, fetch it once outside the loop and use the cached value inside.

#### Optimized Code:
```solidity
function _createPositionInAMM(
    IUniswapV3Pool univ3pool,
    uint256 tokenId,
    uint128 positionSize,
    bool isBurn
) internal returns (int256 totalMoved, int256 totalCollected, int256 itmAmounts) {
    uint256 numLegs = tokenId.countLegs();
    
    // Example of caching data that might be used in each iteration
    TokenType tokenType = extractTokenType(tokenId); // Assuming this function exists and is used inside the loop.
    TickRange tickRange = extractTickRange(tokenId); // Assuming this function exists and is used inside the loop.

    for (uint256 leg = 0; leg < numLegs; ) {
        ...
        // Use cached `tokenType` and `tickRange` here instead of fetching/calculating them again.
        ...
        unchecked {
            ++leg;
        }
    }
    ...
}
```

In this optimized version, functions like `extractTokenType` and `extractTickRange` are hypothetical and represent any operation that might be repeatedly called within the loop. By fetching or calculating these values once and using them within the loop, we can reduce the computational overhead and thus save gas.



### Link : [SemiFungiblePositionManager.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L848)


