## Gas Optimization Report Panoptic
### ERC1155 Contract 

- **Using `calldata` Instead of `memory` for Read-Only Arguments**
  **Summary**: Changing from `memory` to `calldata` for external functions' arguments saves gas.
  **Before**: 
  ```solidity
  function safeBatchTransferFrom(..., uint256[] memory ids, ...) public virtual { ... }
  ```
  **After**:
  ```solidity
  function safeBatchTransferFrom(..., uint256[] calldata ids, ...) public virtual { ... }
  ```

- **Caching State Variables with Stack Variables**
  **Summary**: Caching state variables in memory can reduce gas costs.
  **Before**: 
  ```solidity
  for (uint256 i = 0; i < ids.length; ++i) {
      balanceOf[from][id] -= amount;
      balanceOf[to][id] += amount;
  }
  ```
  **After**:
  ```solidity
  for (uint256 i = 0; i < ids.length; ++i) {
      uint256 fromBalance = balanceOf[from][id];
      uint256 toBalance = balanceOf[to][id];
      fromBalance -= amount;
      toBalance += amount;
      balanceOf[from][id] = fromBalance;
      balanceOf[to][id] = toBalance;
  }
  ```

- **Use Assembly for Small Keccak256 Hashes**
  **Summary**: Using assembly for keccak256 can be more gas-efficient for certain operations.
  **Before**: 
  ```solidity
  bytes32 hash = keccak256(abi.encodePacked(...));
  ```
  **After**:
  ```solidity
  bytes32 hash;
  assembly {
      hash := keccak256(add(data, 32), mload(data))
  }
  ```
### LeftRight Library

- **Use Local Variables for Emitting**
  **Summary**: Caching values in local variables before emitting events can save gas.
  **Before**:
  ```solidity
  emit TransferSingle(msg.sender, from, to, id, amount);
  ```
  **After**:
  ```solidity
  address operator = msg.sender;
  emit TransferSingle(operator, from, to, id, amount);
  ```

- **Using Pre Instead of Post Increments/Decrements**
  **Summary**: Pre-increment/decrement operators are generally more gas-efficient.
  **Before**:
  ```solidity
  for (uint256 i = 0; i < ids.length; i++) {
      ...
  }
  ```
  **After**:
  ```solidity
  for (uint256 i = 0; i < ids.length; ++i) {
      ...
  }
  ```
### LiquidityChunk Library

- **Using Pre Instead of Post Increments/Decrements**
  **Summary**: Pre-increment/decrement operators are typically more gas-efficient.
  **Before**:
  ```solidity
  for (uint256 i = 0; i < ids.length; i++) {
      ...
  }
  ```
  **After**:
  ```solidity
  for (uint256 i = 0; i < ids.length; ++i) {
      ...
  }
  ```

- **Use Local Variables for Emitting**
  **Summary**: Using local variables before emitting events can save gas.
  **Before**:
  ```solidity
  emit EventName(value1, value2);
  ```
  **After**:
  ```solidity
  uint256 localValue1 = value1;
  uint256 localValue2 = value2;
  emit EventName(localValue1, localValue2);
  ```
### TokenId Library

- **Using Calldata Instead of Memory for Read-Only Arguments in External Functions**
  - **Summary**: Using `calldata` for read-only external function parameters is more gas-efficient than `memory`.
  - **Before**:
    ```solidity
    function addLeg(uint256 self, uint256 legIndex, uint256 _optionRatio, ...) internal pure returns (uint256) { ... }
    ```
  - **After**:
    ```solidity
    function addLeg(uint256 self, uint256 legIndex, uint256 _optionRatio, ...) external pure returns (uint256) { ... }
    ```

- **Use Local Variables for Emitting**
  - **Summary**: Storing values in local variables before emitting events can reduce gas usage.
  - **Before**:
    ```solidity
    emit Event(self.optionRatio(i), self.asset(i));
    ```
  - **After**:
    ```solidity
    uint256 optionRatio = self.optionRatio(i);
    uint256 asset = self.asset(i);
    emit Event(optionRatio, asset);
    ```
### CallbackLib Library 

1. **Avoid Contract Existence Checks**
   - **Summary**: The library uses high-level calls which include contract existence checks.
   - **Suggested Solution**: Replace high-level calls with low-level calls where appropriate to avoid unnecessary gas costs.

2. **Optimize Function Names**
   - **Summary**: Function names in the library are lengthy, which increases deployment costs.
   - **Suggested Solution**: Shorten function names to reduce gas costs.
   
   - **Before**: `function validateCallbackFromUniswapPool(...) {...}`
   - **After**: `function validateUniCallback(...) {...}`

3. Unsafe Downcast
   - **Before**: `uint256 keccakValue = uint256(keccak256(abi.encode(features)))`
   - **After**: 
     ```solidity
     uint256 keccakValueRaw = uint256(keccak256(abi.encode(features)));
     require(keccakValueRaw <= type(uint160).max, "Value too large for downcast");
     uint160 keccakValue = uint160(keccakValueRaw);
     ```

4. **Constants Should be Defined**
   - **Before**: `if (value < 2 ** 64) {...}`
   - **After**: 
     ```solidity
     uint256 constant THRESHOLD = 2 ** 64;
     if (value < THRESHOLD) {...}
     ```
### FeesCalc Library

- **Optimize Loop Incrementation**
  - **Summary**: Pre-incrementation can be more gas-efficient than post-incrementation.
  - **Before**:
    ```solidity
    for (uint256 i = 0; i < length; i++) { ... }
    ```
  - **After**:
    ```solidity
    for (uint256 i = 0; i < length; ++i) { ... }
    ```

- **Cache External Calls for Gas Efficiency**
  - **Summary**: Caching external call results can save gas when the result is used multiple times.
  - **Before**:
    ```solidity
    return univ3pool.feeGrowthGlobal0X128() - lowerOut0 - upperOut0;
    ```
  - **After**:
    ```solidity
    uint256 globalFeeGrowth0X128 = univ3pool.feeGrowthGlobal0X128();
    return globalFeeGrowth0X128 - lowerOut0 - upperOut0;
    ```
### PanopticMath Library

- **Optimize Loop Incrementation**
  - **Summary**: Pre-incrementation can be more gas-efficient than post-incrementation.
  - **Before**:
    ```solidity
    for (uint256 i = 0; i < length; i++) { ... }
    ```
  - **After**:
    ```solidity
    for (uint256 i = 0; i < length; ++i) { ... }
    ```

- **Cache External Calls for Gas Efficiency**
  - **Summary**: Caching external call results can save gas when the result is used multiple times.
  - **Before**:
    ```solidity
    return Math.mulDiv(Math.absUint(amount), 2 ** 192, uint256(sqrtPriceX96) ** 2).toInt256();
    ```
  - **After**:
    ```solidity
    uint256 priceSquared = uint256(sqrtPriceX96) ** 2;
    return Math.mulDiv(Math.absUint(amount), 2 ** 192, priceSquared).toInt256();
    ```

### Semi-Fungible Position Manager (SFPM)

- **Using Pre Instead of Post Increments/Decrements**
  **Summary**: Pre-increment/decrement operators are typically more gas-efficient.
  **Before**:
  ```solidity
  for (uint256 i = 0; i < ids.length; i++) {
      ...
  }
  ```
  **After**:
  ```solidity
  for (uint256 i = 0; i < ids.length; ++i) {
      ...
  }
  ```
