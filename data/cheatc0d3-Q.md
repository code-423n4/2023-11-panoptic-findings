## QA Report Panoptic

### ERC1155 Contract 

#### Low Issues

- **Two-Step Procedure for Updating Protocol Addresses**
  **Summary**: Implementing a two-step update process for critical addresses can reduce risks associated with immediate changes.
  
  **After**:
  ```solidity
  address private _pendingAddress;
  uint256 private _pendingAddressTimestamp;

  function proposeNewAddress(address _newAddress) external onlyOwner {
      _pendingAddress = _newAddress;
      _pendingAddressTimestamp = block.timestamp;
  }

  function acceptNewAddress() external onlyOwner {
      require(block.timestamp >= _pendingAddressTimestamp + TIME_DELAY, "Delay not met");
      // Set the new address
  }
  ```

- **State Variables Not Capped at Reasonable Values**
  **Summary**: Capping state variables at reasonable values can prevent misuse or errors.
  **Before**: 
  ```solidity
  mapping(...) public balanceOf;
  ```
  **After**:
  ```solidity
  mapping(...) public balanceOf;
  uint256 public constant MAX_BALANCE = ...;

  function _mint(...) internal {
      require(newBalance <= MAX_BALANCE, "Balance exceeds max limit");
      balanceOf[to][id] += amount;
  }
  ```

- **Missing Necessary Input Validations**
  **Summary**: Input validation is crucial to prevent potential errors or misuse.
  **Before**: 
  ```solidity
  function safeTransferFrom(...) public {
      ...
  }
  ```
  **After**:
  ```solidity
  function safeTransferFrom(...) public {
      require(to != address(0), "Invalid recipient");
      require(amounts[i] > 0, "Invalid amount");
      ...
  }
  ```


#### Non-critical Issues

- **Use of Constants and Enums for Clarity**
  **Summary**: Using named constants and enums improves code readability and maintenance.
  **Before**: 
  ```solidity
  uint256 internal constant SOME_VALUE = 200;
  ```
  **After**:
  ```solidity
  uint256 internal constant MAX_DEADLINE = 200;
  enum TokenState { ACTIVE, INACTIVE }
  ```

- **Refactoring Common Functions to a Common Base Contract**
  **Summary**: Commonly used functions should be moved to a base contract for better modularity and code reuse.
  **Before**: 
  ```solidity
  contract ERC1155 {
      function commonFunction(...) public { ... }
  }
  ```
  **After**:
  ```solidity
  contract CommonBase {
      function commonFunction(...) internal { ... }
  }
  contract ERC1155 is CommonBase { ... }
  ```
### LeftRight Library

#### Low Issues

- **Unsafe Downcast**
  **Summary**: Downcasting without checks can lead to data loss or unexpected behavior.
  **Before**:
  ```solidity
  function toInt128(int256 self) internal pure returns (int128 selfAsInt128) {
      return int128(self);
  }
  ```
  **After**:
  ```solidity
  function toInt128(int256 self) internal pure returns (int128 selfAsInt128) {
      if (!((selfAsInt128 = int128(self)) == self)) revert Errors.CastingError();
  }
  ```

- **Missing Necessary Input Validations**
  **Summary**: Lack of input validation may lead to unexpected results or misuse.
  **Before**:
  ```solidity
  function add(uint256 x, uint256 y) internal pure returns (uint256 z) {
      return x + y;
  }
  ```
  **After**:
  ```solidity
  function add(uint256 x, uint256 y) internal pure returns (uint256 z) {
      unchecked {
          z = x + y;
          if (z < x) revert Errors.UnderOverFlow();
      }
  }
  ```

#### Non-critical Issues

- **Constants Should Be Defined Rather Than Using Magic Numbers**
  **Summary**: Using named constants enhances code readability and maintainability.
  **Before**:
  ```solidity
  int256 internal constant RIGHT_HALF_BIT_MASK = 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF;
  ```
  **After**:
  ```solidity
  int256 internal constant RIGHT_HALF_BIT_MASK = type(int128).max;
  ```

- **Consider Using Descriptive Constants When Passing Zero as a Function Argument**
  **Summary**: Descriptive constants improve readability and reduce errors.
  **Before**:
  ```solidity
  if (right < 0) revert Errors.LeftRightInputError();
  ```
  **After**:
  ```solidity
  int128 constant ZERO_INT128 = 0;
  if (right < ZERO_INT128) revert Errors.LeftRightInputError();
  ```

- **NatSpec Comments Should Be Accurate and Descriptive**
  **Summary**: NatSpec comments should accurately reflect the function logic and purpose.
  **Before**:
  ```solidity
  /// @notice Add two uint256 bit LeftRight-encoded words; revert on overflow or underflow.
  function add(uint256 x, uint256 y) internal pure returns (uint256 z) {
      ...
  }
  ```
  **After**:
  ```solidity
  /// @notice Safely add two uint256 bit LeftRight-encoded words with overflow check.
  function add(uint256 x, uint256 y) internal pure returns (uint256 z) {
      ...
  }
  ```
### LiquidityChunk Library


#### Low Issues

- **Unsafe Downcast**
  **Summary**: Downcasting without appropriate checks can lead to data loss or unexpected behavior.
  **Before**:
  ```solidity
  function addTickLower(uint256 self, int24 _tickLower) internal pure returns (uint256) {
      return self + (uint256(uint24(_tickLower)) << 232);
  }
  ```
  **After**:
  ```solidity
  function addTickLower(uint256 self, int24 _tickLower) internal pure returns (uint256) {
      if (_tickLower > type(uint24).max || _tickLower < type(int24).min) revert Errors.OutOfBounds();
      return self + (uint256(uint24(_tickLower)) << 232);
  }
  ```

- **Missing Necessary Input Validations**
  **Summary**: Ensuring that input values are within expected bounds can prevent misuse or unexpected behavior.
  **Before**:
  ```solidity
  function createChunk(...) internal pure returns (uint256) {
      return self.addLiquidity(amount).addTickLower(_tickLower).addTickUpper(_tickUpper);
  }
  ```
  **After**:
  ```solidity
  function createChunk(...) internal pure returns (uint256) {
      if (_tickLower >= _tickUpper) revert Errors.InvalidTicks();
      if (amount == 0) revert Errors.ZeroAmount();
      return self.addLiquidity(amount).addTickLower(_tickLower).addTickUpper(_tickUpper);
  }
  ```

#### Non-critical Issues

- **Constants Should Be Defined Rather Than Using Magic Numbers**
  **Summary**: Utilizing named constants improves code readability and maintenance.
  **Before**:
  ```solidity
  return self + (uint256(uint24(_tickLower)) << 232);
  ```
  **After**:
  ```solidity
  uint256 constant TICK_LOWER_SHIFT = 232;
  return self + (uint256(uint24(_tickLower)) << TICK_LOWER_SHIFT);
  ```

- **NatSpec Comments Should Be Accurate and Descriptive**
  **Summary**: NatSpec comments should accurately reflect the function logic and purpose.
  **Before**:
  ```solidity
  /// @notice Add liquidity to the chunk.
  function addLiquidity(uint256 self, uint128 amount) internal pure returns (uint256) {
      return self + uint256(amount);
  }
  ```
  **After**:
  ```solidity
  /// @notice Safely adds the specified amount of liquidity to the given chunk.
  function addLiquidity(uint256 self, uint128 amount) internal pure returns (uint256) {
      if (amount == 0) revert Errors.ZeroAmount();
      return self + uint256(amount);
  }
  ```
### TokenId Library


#### Low Issues

- **Unsafe Downcast**
  - **Summary**: Downcasting without proper checks can lead to unintended behavior or data corruption.
  - **Before**:
    ```solidity
    function addStrike(uint256 self, int24 _strike, uint256 legIndex) internal pure returns (uint256) {
        return self + uint256((int256(_strike) & BITMASK_INT24) << (64 + legIndex * 48 + 12));
    }
    ```
  - **After**:
    ```solidity
    function addStrike(uint256 self, int24 _strike, uint256 legIndex) internal pure returns (uint256) {
        if (int256(_strike) > type(int24).max || int256(_strike) < type(int24).min) revert Errors.InvalidStrikeValue();
        return self + uint256((int256(_strike) & BITMASK_INT24) << (64 + legIndex * 48 + 12));
    }
    ```

- **Loss of Precision**
  - **Summary**: Potential loss of precision during calculations.
  - **Suggested Solution**: Use higher precision for intermediate calculations or provide adequate rounding mechanisms.

#### Non-critical Issues

- **Constants in Comparisons Should Appear on the Left Side**
  - **Summary**: This is a recommended practice for readability and avoiding assignment errors.
  - **Before**:
    ```solidity
    if (self.optionRatio(i) == 0) { ... }
    ```
  - **After**:
    ```solidity
    if (0 == self.optionRatio(i)) { ... }
    ```

- **Function @param Tag is Missing**
  - **Summary**: Missing `@param` tags in NatSpec comments can lead to unclear documentation.
  - **Suggested Change**: Ensure all public and external functions have complete `@param` tags for each parameter.

### CallbackLib Library 


#### Low Issues

1. **Unsafe Downcast**
   - **Summary**: In the `validateCallback` function, downcasting without proper checks can lead to unintended behavior.
   - **Suggested Solution**: Implement explicit range checks before downcasting.
   
      - **Before**: `uint256 keccakValue = uint256(keccak256(abi.encode(features)))`
   - **After**: 
     ```solidity
     uint256 keccakValueRaw = uint256(keccak256(abi.encode(features)));
     require(keccakValueRaw <= type(uint160).max, "Value too large for downcast");
     uint160 keccakValue = uint160(keccakValueRaw);
     ```


2. **Loss of Precision**
   - **Summary**: Potential loss of precision due to fixed-size integer operations in `validateCallback`.
   - **Suggested Solution**: Use higher precision for intermediate calculations or provide adequate rounding mechanisms.

3. **Missing Necessary Input Validations**
   - **Summary**: The absence of input validation in functions can lead to unexpected behavior or vulnerabilities.
   - **Suggested Solution**: Add input validation checks, particularly for parameters like `factory` and `sender`, to ensure they meet expected criteria (e.g., non-zero addresses).


#### Non-critical Issues

1. **Constants Should be Defined**
   - **Summary**: Magic numbers are used in the library, decreasing code readability and maintainability.
   - **Suggested Solution**: Replace magic numbers with clearly defined constants.

2. **Inconsistent Commenting**
   - **Summary**: Inconsistent and unclear comments throughout the library can lead to misunderstanding of the code's purpose.
   - **Suggested Solution**: Standardize and clarify comments for consistency and clarity.

3. **Use of Assembly**
   - **Summary**: The library lacks any use of assembly where it might be beneficial for performance.
   - **Suggested Solution**: Investigate potential areas where assembly could be used to optimize the contract's gas usage.

### FeesCalc Library

#### Low Issues

- **Loss of Precision in Fee Calculations**
  - **Summary**: Precision loss in fee calculations could lead to inaccurate fee allocation.
  - **Before**:
    ```solidity
    feesEachToken = feesEachToken
        .toRightSlot(int128(int256(Math.mulDiv128(ammFeesPerLiqToken0X128, startingLiquidity))))
        .toLeftSlot(int128(int256(Math.mulDiv128(ammFeesPerLiqToken1X128, startingLiquidity))));
    ```
  - **After**:
    ```solidity
    feesEachToken = feesEachToken
        .toRightSlot(Math.toPreciseInt128(Math.mulDiv128(ammFeesPerLiqToken0X128, startingLiquidity)))
        .toLeftSlot(Math.toPreciseInt128(Math.mulDiv128(ammFeesPerLiqToken1X128, startingLiquidity)));
    ```

- **Unchecked Arithmetic in Fee Growth Calculation**
  - **Summary**: Unchecked arithmetic operations might cause overflow/underflow errors.
  - **Before**:
    ```solidity
    feeGrowthInside0X128 = upperOut0 - lowerOut0;
    feeGrowthInside1X128 = upperOut1 - lowerOut1;
    ```
  - **After**:
    ```solidity
    feeGrowthInside0X128 = Math.safeSubtract(upperOut0, lowerOut0);
    feeGrowthInside1X128 = Math.safeSubtract(upperOut1, lowerOut1);
    ```


#### Non-critical Issues

- **Use Descriptive Constants for Magic Numbers**
  - **Summary**: Using descriptive constants instead of magic numbers enhances code readability.
  - **Before**:
    ```solidity
    return self + (uint256(uint24(_tickLower)) << 232);
    ```
  - **After**:
    ```solidity
    uint256 constant TICK_LOWER_SHIFT = 232;
    return self + (uint256(uint24(_tickLower)) << TICK_LOWER_SHIFT);
    ```

- **Improve NatSpec Comments for Clarity**
  - **Summary**: Accurate and descriptive NatSpec comments can significantly improve code readability.
  - **Before**:
    ```solidity
    /// @notice Calculate the AMM Swap/trading fees for a `liquidityChunk` of each token.
    function calculateAMMSwapFeesLiquidityChunk(...) public view returns (int256 feesEachToken) { ... }
    ```
  - **After**:
    ```solidity
    /// @notice Calculates the accumulated AMM swap fees for a given liquidity chunk in a Uniswap pool.
    function calculateAMMSwapFeesLiquidityChunk(...) public view returns (int256 feesEachToken) { ... }
    ```
### PanopticMath Library

#### Low Issues

- **Unchecked Arithmetic in Token Conversion Logic**
  - **Summary**: Unchecked arithmetic operations can result in overflow/underflow, leading to inaccurate conversions.
  - **Before**:
    ```solidity
    return Math.mulDiv192(Math.absUint(amount), uint256(sqrtPriceX96) ** 2).toInt256();
    ```
  - **After**:
    ```solidity
    return Math.safeMulDiv192(Math.absUint(amount), uint256(sqrtPriceX96) ** 2).toInt256();
    ```

- **Lack of Input Validation in Liquidity Calculation**
  - **Summary**: Ensuring inputs are within reasonable limits prevents misuse and potential errors.
  - **Before**:
    ```solidity
    function getLiquidityChunk(...) internal pure returns (uint256 liquidityChunk) { ... }
    ```
  - **After**:
    ```solidity
    function getLiquidityChunk(...) internal pure returns (uint256 liquidityChunk) {
        if (legIndex >= 4 || positionSize == 0) revert Errors.InvalidInput();
        ...
    }
    ```

#### Non-critical Issues

- **Use Descriptive Constants for Magic Numbers**
  - **Summary**: Descriptive constants enhance code readability and maintainability.
  - **Before**:
    ```solidity
    return uint64(uint160(univ3pool) >> 96);
    ```
  - **After**:
    ```solidity
    uint256 constant POOL_ID_SHIFT = 96;
    return uint64(uint160(univ3pool) >> POOL_ID_SHIFT);
    ```

- **Improve NatSpec Comments for Clarity**
  - **Summary**: Clear and accurate NatSpec comments aid understanding and maintainability.
  - **Before**:
    ```solidity
    /// @notice Convert an amount of token0 into an amount of token1...
    function convert0to1(int256 amount, uint160 sqrtPriceX96) internal pure returns (int256) { ... }
    ```
  - **After**:
    ```solidity
    /// @notice Accurately converts a specified amount of token0 into token1 using the given price.
    function convert0to1(int256 amount, uint160 sqrtPriceX96) internal pure returns (int256) { ... }
    ```
### Semi-Fungible Position Manager (SFPM)

#### Low Issues

- **Unsafe Downcast**
  **Summary**: Downcasting without proper checks can lead to data loss or unexpected behavior.
  **Before**:
  ```solidity
  uint64 poolId = uint64(uint160(univ3pool) >> 96);
  ```
  **After**:
  ```solidity
  function safeDowncastToUint64(uint160 value) internal pure returns (uint64) {
      require(value <= type(uint64).max, "Value doesn't fit in 64 bits");
      return uint64(value);
  }
  uint64 poolId = safeDowncastToUint64(uint160(univ3pool) >> 96);
  ```

- **Missing Necessary Input Validations**
  **Summary**: Validation of input values ensures that they are within expected boundaries.
  **Before**:
  ```solidity
  function initializeAMMPool(address token0, address token1, uint24 fee) external {
      address univ3pool = FACTORY.getPool(token0, token1, fee);
      //... rest of the function
  }
  ```
  **After**:
  ```solidity
  function initializeAMMPool(address token0, address token1, uint24 fee) external {
      require(token0 != address(0) && token1 != address(0), "Invalid token addresses");
      require(fee > 0, "Fee cannot be zero");
      address univ3pool = FACTORY.getPool(token0, token1, fee);
      //... rest of the function
  }
  ```

#### Non-critical Issues

- **Constants Should Be Defined Rather Than Using Magic Numbers**
  **Summary**: Using named constants improves code readability.
  **Before**:
  ```solidity
  uint64 poolId = uint64(uint160(univ3pool) >> 96);
  ```
  **After**:
  ```solidity
  uint256 constant POOL_ID_SHIFT = 96;
  uint64 poolId = uint64(uint160(univ3pool) >> POOL_ID_SHIFT);
  ```

- **NatSpec Comments Should Be Accurate and Descriptive**
  **Summary**: NatSpec comments should accurately reflect the function's purpose.
  **Before**:
  ```solidity
  /// @notice Returns the Uniswap v3 pool for a given poolId.
  function getUniswapV3PoolFromId(uint64 poolId) external view returns (IUniswapV3Pool UniswapV3Pool) {
      return s_poolContext[poolId].pool;
  }
  ```
  **After**:
  ```solidity
  /// @notice Returns the Uniswap v3 pool for a given poolId.
  /// @param poolId The poolId to query.
  /// @return UniswapV3Pool The Uniswap V3 pool associated with the given pool ID.
  function getUniswapV3PoolFromId(uint64 poolId) external view returns (IUniswapV3Pool UniswapV3Pool) {
      return s_poolContext[poolId].pool;
  }
  ```
