| S.No | Title                                                                                                      |
|------|------------------------------------------------------------------------------------------------------------|
| **L-01** | Potential Precision Loss in `getAmount0ForLiquidity` Function                                             |
| **L-02** | Potential Division by Zero in `mulDiv` Function                                                           |
| **L-03** | Potential Handling Issue of Extreme Values in `absUint`                                                   |
| **L-04** | Arithmetic Precision Loss in `asTicks` Function                                                           |
| **L-05** | Lack of Zero Address Check in `safeTransferFrom` and `safeBatchTransferFrom`                              |
| **L-06** | Redundant Code Length Check for `to` Address in Transfer Functions                                        |
| **L-07** | Lack of Event Emission in `_mint` for Failed Receiver Call                                                |
| **L-08** | Address Collision Risk in Callback Validation Logic                                                       |


##  [L-01] Potential Precision Loss in `getAmount0ForLiquidity` Function

### Contract : Math.sol

The functions `getAmount0ForLiquidity`  perform calculations to determine the amount of tokens based on liquidity values. These calculations involve division and multiplication operations with fixed-point arithmetic. Due to the nature of these operations, there's a risk of precision loss, which can lead to slightly inaccurate calculations. This might not be a critical issue but could impact the financial accuracy of the contract in certain scenarios.

### Code Snippet:
```solidity
function getAmount0ForLiquidity(uint256 liquidityChunk) internal pure returns (uint256 amount0) {
    ...
    unchecked {
        return
            mulDiv(
                uint256(liquidityChunk.liquidity()) << 96,
                highPriceX96 - lowPriceX96,
                highPriceX96
            ) / lowPriceX96;
    }
}

```

### Expected Behavior:
The ideal behavior in financial calculations is to minimize precision loss as much as possible. This ensures that every token calculation is as accurate as possible, maintaining the integrity of financial transactions.

### Actual Behavior:
In the current implementation, while the calculations are technically correct, the division and multiplication with fixed-point numbers could lead to rounding errors. Over time, or in high-frequency trading environments, these minor inaccuracies could compound, potentially leading to discrepancies in token balances or liquidity calculations.



To provide an example of the potential precision loss in the `getAmount0ForLiquidity` and `getAmount1ForLiquidity` functions, let's consider a hypothetical scenario with specific input values. We'll calculate the expected and actual outcomes to illustrate the potential for precision loss.

### Scenario for `getAmount0ForLiquidity` Function

### Given Inputs:
- `liquidityChunk.liquidity()`: \( 1000 \) (example liquidity value)
- `lowPriceX96`: \( 500000000000000000 \) (example lower price in Q64.96 format)
- `highPriceX96`: \( 1000000000000000000 \) (example higher price in Q64.96 format)

### Expected Calculation:
The ideal calculation aims to minimize precision loss. In an environment with infinite precision, the calculation would be:

\[ \text{Expected Amount0} = \frac{1000 \times (1000000000000000000 - 500000000000000000)}{1000000000000000000} \div 500000000000000000 \]

### Actual Calculation in Solidity:
Solidity does not handle floating-point operations and has limited precision. The calculation in the contract is:

\[ \text{Actual Amount0} = \left\lfloor \frac{1000 \ll 96 \times (1000000000000000000 - 500000000000000000)}{1000000000000000000} \right\rfloor \div 500000000000000000 \]

Here, `1000 << 96` means the liquidity is left-shifted by 96 bits to convert it into a Q64.96 format before multiplication and division.

### Scenario for `getAmount1ForLiquidity` Function

### Given Inputs:
- `liquidityChunk.liquidity()`: \( 1000 \) (example liquidity value)
- `lowPriceX96`: \( 500000000000000000 \) (example lower price in Q64.96 format)
- `highPriceX96`: \( 1000000000000000000 \) (example higher price in Q64.96 format)

### Expected Calculation:
Again, with infinite precision, the calculation would be:

\[ \text{Expected Amount1} = 1000 \times (1000000000000000000 - 500000000000000000) \]

### Actual Calculation in Solidity:
In Solidity, with limited precision:

\[ \text{Actual Amount1} = \left\lfloor \frac{1000 \times (1000000000000000000 - 500000000000000000)}{2^{96}} \right\rfloor \]

### Summary:
In both scenarios, the actual calculations in Solidity involve bit shifting and flooring operations, which can lead to precision loss compared to the expected calculations with higher precision. This precision loss might not be significant in a single transaction but could become noticeable over a large number of transactions, especially in a DeFi context where accuracy is critical.

Let's perform these calculations to see the difference in output.

The calculation results demonstrate the precision loss in the `getAmount0ForLiquidity` and `getAmount1ForLiquidity` functions:

### `getAmount0ForLiquidity` Function Results
- **Expected Amount0**: Approximately \( 1 \times 10^{-15} \)
- **Actual Amount0 (Solidity)**: \( 79,228,162,514,264 \)

### `getAmount1ForLiquidity` Function Results
- **Expected Amount1**: \( 500,000,000,000,000,000,000 \) (considering high precision)
- **Actual Amount1 (Solidity)**: \( 0 \)

### Analysis:
- For `getAmount0ForLiquidity`, the expected result is a very small number close to zero, which is not practical in a blockchain context where values are usually integers. However, the Solidity calculation results in a significantly higher value due to the conversion and flooring operations. This discrepancy can lead to incorrect financial calculations in the contract.
- For `getAmount1ForLiquidity`, the expected high-precision result is a large number, but the Solidity calculation, limited by integer arithmetic, results in zero. This is a clear example of precision loss, where the actual implementation does not capture the fractional part of the result.

These discrepancies highlight the potential issues with fixed-point arithmetic in Solidity, especially when dealing with financial calculations that require high precision. While Solidity's integer arithmetic is suitable for many use cases, special care must be taken in scenarios where precision is critical, such as in DeFi applications.


### Scenario for `getAmount0ForLiquidity` Function

### Given Inputs:
- `liquidityChunk.liquidity()`: \( 1000 \) (example liquidity value)
- `lowPriceX96`: \( 500000000000000000 \) (example lower price in Q64.96 format)
- `highPriceX96`: \( 1000000000000000000 \) (example higher price in Q64.96 format)

### Expected Calculation:
The ideal calculation aims to minimize precision loss. In an environment with infinite precision, the calculation would be:

\[ \text{Expected Amount0} = \frac{1000 \times (1000000000000000000 - 500000000000000000)}{1000000000000000000} \div 500000000000000000 \]

### Actual Calculation in Solidity:
Solidity does not handle floating-point operations and has limited precision. The calculation in the contract is:

\[ \text{Actual Amount0} = \left\lfloor \frac{1000 \ll 96 \times (1000000000000000000 - 500000000000000000)}{1000000000000000000} \right\rfloor \div 500000000000000000 \]

Here, `1000 << 96` means the liquidity is left-shifted by 96 bits to convert it into a Q64.96 format before multiplication and division.


Let's perform these calculations to see the difference in output.

The calculation results demonstrate the precision loss in the `getAmount0ForLiquidity`  functions:

### `getAmount0ForLiquidity` Function Results
- **Expected Amount0**: Approximately \( 1 \times 10^{-15} \)
- **Actual Amount0 (Solidity)**: \( 79,228,162,514,264 \)



### Analysis:
- For `getAmount0ForLiquidity`, the expected result is a very small number close to zero, which is not practical in a blockchain context where values are usually integers. However, the Solidity calculation results in a significantly higher value due to the conversion and flooring operations. This discrepancy can lead to incorrect financial calculations in the contract.

---

## [L-02] Potential Division by Zero in `mulDiv` Function

### Contract : Math.sol

The `mulDiv` function is designed to perform a multiplication and division operation in one step, computing `floor(a×b÷denominator)` with full precision. However, there seems to be a lack of explicit handling or checking for a zero denominator before performing the division operation. While the function does have a requirement that `denominator > prod1` to prevent overflow, it does not explicitly check if the `denominator` is zero before the division takes place. This oversight could lead to undefined behavior or a revert if a zero `denominator` is passed to the function.

### Code Snippet
```solidity
function mulDiv(
    uint256 a,
    uint256 b,
    uint256 denominator
) internal pure returns (uint256 result) {
    unchecked {
        ...
        // Make sure the result is less than 2**256.
        // Also prevents denominator == 0
        require(denominator > prod1);
        ...
    }
}
```

### Expected Behavior
The function should explicitly check for a zero `denominator` and handle it appropriately, either by returning a specific value or reverting with a clear error message. This would prevent undefined behavior and ensure that the function fails gracefully in case of an invalid input.

### Actual Behavior
Currently, the function does not explicitly check if the `denominator` is zero before performing the division. While the `require(denominator > prod1)` statement indirectly prevents division by zero in most cases, it does not cover the scenario where both `prod1` and `denominator` are zero. In such a case, the function might exhibit undefined behavior or cause a revert without a clear explanation.

### Recommendation
It is recommended to add an explicit check at the beginning of the function to ensure that the `denominator` is not zero. This can be done using a `require` statement with an appropriate error message:

```solidity
require(denominator != 0, "Division by zero error");
```

This additional check will enhance the robustness of the function and prevent potential issues related to division by zero.

---

### [L-03] Potential Handling Issue of Extreme Values in `absUint`

### Contract : Math.sol

The `absUint` function is designed to compute the absolute value of an `int256` and return it as a `uint256`. The function employs a straightforward method of checking if the input is positive or negative and then converting it accordingly. However, this implementation could potentially lead to unexpected behavior when dealing with the extreme negative value `type(int256).min` (i.e., `-2**255`).

In Solidity, the `int256` type can represent values from `-2**255` to `2**255 - 1`. The issue arises because the absolute value of `type(int256).min` (`-2**255`) is `2**255`, which cannot be represented as a positive `int256` but can fit in a `uint256`. The current implementation does not explicitly address this edge case.

### Code Snippet
```solidity
function absUint(int256 x) internal pure returns (uint256) {
    unchecked {
        return x > 0 ? uint256(x) : uint256(-x);
    }
}
```

### Expected Behavior
The function should correctly handle the conversion of `type(int256).min` to its absolute value. Since `type(int256).min` is a special case where its absolute value is outside the range of `int256` but within the range of `uint256`, the function should explicitly account for this edge case to ensure accurate computation.

### Actual Behavior
The current implementation of `absUint` may not correctly handle the case when `x` is `type(int256).min`. Due to the nature of two's complement representation of negative numbers in Solidity, simply negating `type(int256).min` does not change its value, leading to a potential logical error where the function returns a negative value despite being intended to return an absolute (positive) value.

### Recommendation
To address this potential issue, consider adding a specific check for `type(int256).min`:

```solidity
function absUint(int256 x) internal pure returns (uint256) {
    unchecked {
        if (x == type(int256).min) {
            return uint256(type(int256).min * -1);
        }
        return x > 0 ? uint256(x) : uint256(-x);
    }
}
```

This additional check ensures that the function can handle all possible `int256` values, including the extreme case of `type(int256).min`. It also makes the function more robust and reliable, particularly in scenarios where extreme values might be encountered.

---

## [L-04] Arithmetic Precision Loss in `asTicks` Function**

### Contract : Tokenid.sol

The `asTicks` function in the contract is designed to calculate the upper and lower tick boundaries for an options position. It uses integer arithmetic to compute these values, which can lead to precision loss due to integer division. This issue arises when the product of `selfWidth` and `tickSpacing` results in an odd number, causing the subsequent division by 2 to truncate any decimal part. In financial applications like options trading, where exact values are crucial, this loss of precision, although minor, could have implications for trading strategies and market operations.

### Code Snippet
```solidity
function asTicks(
    uint256 self,
    uint256 legIndex,
    int24 tickSpacing
) internal pure returns (int24 legLowerTick, int24 legUpperTick) {
    unchecked {
        int24 selfWidth = width(self, legIndex);
        int24 selfStrike = strike(self, legIndex);
        int24 oneSidedRange = (selfWidth * tickSpacing) / 2;

        (legLowerTick, legUpperTick) = (selfStrike - oneSidedRange, selfStrike + oneSidedRange);
    }
}
```

### Expected Calculation
- **Scenario**: Let's assume `selfWidth = 1` and `tickSpacing = 3`. The product `selfWidth * tickSpacing` equals `3`. When divided by `2`, the ideal result should be `1.5`.
- **Ideal Lower and Upper Ticks**: If `selfStrike = 1000`, the ideally calculated lower and upper ticks would be:
  - `legLowerTick` = `1000 - 1.5 = 998.5`
  - `legUpperTick` = `1000 + 1.5 = 1001.5`
- Since ticks are typically represented as integers, these would ideally be rounded or truncated based on specific rules for handling fractional ticks.

### Actual Calculation (With Precision Loss)
- **Observed Behavior**: Due to integer division in Solidity, the division `3 / 2` results in `1`, not `1.5`. This leads to slightly different tick calculations:
  - `legLowerTick` = `1000 - 1 = 999`
  - `legUpperTick` = `1000 + 1 = 1001`
- **Resulting Imprecision**: The lower tick is calculated 0.5 ticks higher than ideal, and the upper tick is calculated 0.5 ticks lower than ideal, leading to a narrower range than expected.

### Impact
This precision loss, while small in each individual calculation, could aggregate over many transactions or affect sensitive trading strategies. In a financial context where precision is paramount, such deviations, even minor, could impact trading decisions, market efficiency, and user confidence in the system's accuracy.

---


## [L-05] Lack of Zero Address Check in `safeTransferFrom` and `safeBatchTransferFrom`**

### Contract : ERC1155Minimal.sol

Missing Zero Address Validation in Transfer Functions. The `safeTransferFrom` and `safeBatchTransferFrom` functions do not validate if the `to` address is the zero address. Transferring to the zero address is generally considered an error.

### Suggested Fix: 
Add a requirement to check if `to` is not the zero address.

### Code Snippet :
  ```solidity
  function safeTransferFrom(
      address from,
      address to,
      uint256 id,
      uint256 amount,
      bytes calldata data
  ) public {
      ...
  }

  function safeBatchTransferFrom(
      address from,
      address to,
      uint256[] calldata ids,
      uint256[] calldata amounts,
      bytes calldata data
  ) public virtual {
      ...
  }
  ```
  
  ---

### [L-06] Redundant Code Length Check for `to` Address in Transfer Functions**

### Contract : ERC1155Minimal.sol

 In both `safeTransferFrom` and `safeBatchTransferFrom`, there is a check for `to.code.length != 0` which is meant to determine if the `to` address is a contract. However, this is redundant due to the EIP-1155 standard's requirement for the receiver to implement the ERC1155 token receiver interface.

### Suggested Fix: 

Consider removing the redundant check or clarify its purpose if it's serving a specific need.

### Code Snippet:
  ```solidity
  if (to.code.length != 0) {
      ...
  }
  ```
  
  ---

## [L-07] Lack of Event Emission in `_mint` for Failed Receiver Call**

### Contract : ERC1155Minimal.sol

 In the `_mint` function, the `TransferSingle` event is emitted before checking if the `to` address can receive ERC1155 tokens. If the receiver call fails, the event would have been emitted incorrectly.

### Suggested Fix : 
Move the event emission after the receiver call check.

## Code Snippet :
  ```solidity
  function _mint(address to, uint256 id, uint256 amount) internal {
      ...
      emit TransferSingle(msg.sender, address(0), to, id, amount);
      ...
  }
  ```

---




## [L-08] Address Collision Risk in Callback Validation Logic**

### Contract: CallbackLib.sol

The `validateCallback` function employs deterministic address computation to verify the authenticity of a Uniswap pool. This approach is based on the assumption that the address derived from the contract's creation bytecode and the creator's address is unique. However, this method is susceptible to address collision risks, where a different contract, under specific circumstances, might share the same computed address, leading to erroneous validation.

### Suggested Fix: 
Strengthen the validation logic by incorporating additional checks beyond address computation. This could involve maintaining a registry of verified pool addresses or introducing cryptographic verification methods that ensure the uniqueness and legitimacy of the pool beyond its address.

### Code Snippet:
  ```solidity
  function validateCallback(
      address sender,
      address factory,
      PoolFeatures memory features
  ) internal pure {
      // existing logic to compute and compare addresses
      if (
          address(
              uint160(
                  uint256(
                      keccak256(
                          abi.encodePacked(
                              bytes1(0xff),
                              factory,
                              keccak256(abi.encode(features)),
                              Constants.V3POOL_INIT_CODE_HASH
                          )
                      )
                  )
              )
          ) != sender
      ) revert Errors.InvalidUniswapCallback();
  }
  ```
---

