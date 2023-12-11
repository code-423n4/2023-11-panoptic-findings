## Summary

### Gas Optimization

no |Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] |storage instead of memory for structs/arrays saves gas |  |2|--| 
| [G-02] |Short-circuit rules can be used to optimize some gas usage |  |2|--| 
| [G-03] |Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate|  |5|--| 
| [G-04] |Multiplication and Division by 2 Should use in Bit Shifting |  |3|--| 
| [G-05] |Using calldata instead of memory for read-only arguments in external functions saves gas |  |5|--| 
| [G-06] |Structs can be packed into fewer storage slots |  |5|--| 
| [G-07] |Use assembly for loops to save gas |  |2|--| 
| [G-08] |Remove or replace unused state variables |  |2|--| 
| [G-09] |Don’t initialize default values to variables to reduce gas |  |6|--| 





## Gas Optimizations  

## [G-1] storage instead of memory for structs/arrays saves gas
When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

### Details

```solidity
   413     CallbackLib.CallbackData memory decoded = abi.decode(data, (CallbackLib.CallbackData));
file:
https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L413-L413
```


```solidity
  1134      bytes memory mintdata = abi.encode(
File: 
https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1134
```



## [G-2] Short-circuit rules can be used to optimize some gas usage
Some conditions may be reordered to save an SLOAD (2100 gas), as we avoid reading state variables when the first part of the condition fails (with &&), or succeeds (with ||). For instance, consider a scenario where you have a stateVariable (a variable stored in contract storage) and a localVariable (a variable in memory). 

If you have a condition like stateVariable > 0 && localVariable > 0, if localVariable > 0 is false, the Solidity runtime will still execute stateVariable > 0, which costs an SLOAD operation (2100 gas). However, if you reorder the condition to localVariable > 0 && stateVariable > 0, the stateVariable > 0 check won’t happen if localVariable > 0 is false, saving you the SLOAD gas cost.

Similarly, for the || operator, if you have a condition like stateVariable > 0 || localVariable > 0, and stateVariable > 0 is true, the Solidity runtime will still execute localVariable > 0. But if you reorder the condition to localVariable > 0 || stateVariable > 0, and localVariable > 0 is true, the stateVariable > 0 check won’t happen, again saving you the SLOAD gas cost.

This detector checks for such conditions in the contract and reports if any condition could be optimized by taking advantage of the short-circuiting behavior of && and ||.
```solidity

128     function safeBatchTransferFrom(
        address from,
        address to,
        uint256[] calldata ids,
        uint256[] calldata amounts,
        bytes calldata data
    ) public virtual {
        if (!(msg.sender == from || isApprovedForAll[from][msg.sender])) revert NotAuthorized();


        // Storing these outside the loop saves ~15 gas per iteration.
        uint256 id;
        uint256 amount;
file:/contracts/tokens/ERC1155Minimal.sol
https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/tokens/ERC1155Minimal.sol#L128-L139
```
``` solidty
 90   function safeTransferFrom(
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

file:contracts/types/LiquidityChunk.sol
https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/tokens/ERC1155Minimal.sol#L90-L104
```
``` solidity
163    function createChunk(
        uint256 self,
        int24 _tickLower,
        int24 _tickUpper,
        uint128 amount
file:contracts/types/LiquidityChunk.sol
https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/types/LiquidityChunk.sol#L63-L67
```
``` solidity
 39       return uint64(uint160(univ3pool) >> 96);

file:/contracts/libraries/PanopticMath.sol
https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/libraries/PanopticMath.sol#L39-L39

```
## [G-3]  We can combine multiple mappings below into structs. We can then pack the structs by 
modifying the uint type for the values. This will result in cheaper storage reads since multiple mappings are accessed in functions and those values are now occupying the same storage slot, meaning the slot will become warm after the first SLOAD. In addition, when writing to and reading from the struct values, we will avoid a Gsset (20000 gas) and Gcoldsload (2100 gas), since multiple struct values are now occupying the same slot
```
solidity
 62   mapping(address account => mapping(uint256 tokenId => uint256 balance)) public balanceOf;
file:/contracts/tokens/ERC1155Minimal
https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/tokens/ERC1155Minimal.sol#L62
```
```
solidity
67     mapping(address owner => mapping(address operator => bool approvedForAll))
file:/contracts/tokens/ERC1155Minimal
https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/tokens/ERC1155Minimal.sol#L67
```
```
solidity
147     mapping(address univ3pool => uint256 poolIdData) internal s_AddrToPoolIdData;
file:/contracts/tokens/ERC1155Minimal
https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L147

```
```solidity
 179   mapping(bytes32 positionKey => uint256 removedAndNetLiquidity) internal s_accountLiquidity;
file:/contracts/SemiFungiblePositionManager.sol
 https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L179
 ```
```solidity
 152   mapping(uint64 poolId => PoolAddressAndLock contextData) internal s_poolContext;
file:/contracts/SemiFungiblePositionManager.sol
https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L152
```
   


## [G-4]Multiplication and Division by 2 Should use in Bit Shifting 
The expressions ‘x * 2’ and ‘x / 2’ can be optimized for gas efficiency by utilizing bitwise operations. In Solidity, you can achieve the same results by using bitwise left shift (x << 1) for multiplication and bitwise right shift (x >> 1) for division.

Using bitwise shift operations (SHL and SHR) instead of multiplication (MUL) and division (DIV) opcodes can lead to significant gas savings. The MUL and DIV opcodes cost 5 gas, while the SHL and SHR opcodes incur a lower cost of only 3 gas.

By leveraging these more efficient bitwise operations, you can reduce the gas consumption of your smart contracts and enhance their overall performance.

```solidity
115       return uint256((self >> (64 + legIndex * 48 + 8)) % 2);
file:contracts/types/TokenId.sol
https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/types/TokenId.sol#L115

```


```solidity
 125  return uint256((self >> (64 + legIndex * 48 + 9)) % 2);
```
file:contracts/types/TokenId.sol
           
https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/types/TokenId.sol#L125

```solidity
   389         int24 oneSidedRange = (selfWidth * tickSpacing) / 2;
file:/contracts/types/TokenId.sol
https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/types/TokenId.sol#L389

```


## [G-5] Using calldata instead of memory for read-only arguments in external functions saves gas
When a function with a memory array is called externally, the abi.decode() step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having memory arguments, it’s still valid for implementation contracts to use calldata arguments instead.

If the array is passed to an internal function, which passes the array to another internal function where the array is modified and therefore memory is used in the external call, it’s still more gas-efficient to use calldata when the external function uses modifiers; since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one.


```solidity
179    address[] calldata owners,
180    uint256[] calldata ids
181    ) public view returns (uint256[] memory balances) {
182    balances = new uint256[](owners.length);
file:/contracts/tokens/ERC1155Minimal.sol
https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/tokens/ERC1155Minimal.sol#L179-L182

```


```solidity
 113       results = new bytes[](data.length);

file:/contracts/multicall/Multicall.sol

https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/multicall/Multicall.sol#L13
```
## [G-6] Structs can be packed into fewer storage slots


### Detials

```solidity
  12  struct PoolFeatures {
  13   address token0;
  14  address token1;
  15  uint24 fee;
       
       
file:
https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/libraries/CallbackLib.sol#L12-L15

```

```solidity
  21      address payer;
file:
https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/libraries/CallbackLib.sol#L21

```
## [G-7]Use assembly for loops to save gas
Saves 2450 GAS for every iteration from instances
Assembly is more gas efficient for loops. Saves minimum 350 GAS per iteration as per remix gas checks.
```solidity
14         for (uint256 i = 0; i < data.length; ) {
            (bool success, bytes memory result) = address(this).delegatecall(data[i]);


            if (!success) {
                // Bubble up the revert reason
                // The bytes type is ABI encoded as a length-prefixed byte array
                // So we simply need to add 32 to the pointer to get the start of the data
                // And then revert with the size loaded from the first 32 bytes
                // Other solutions will do work to differentiate the revert reasons and provide paranthetical information
                // However, we have chosen to simply replicate the the normal behavior of the call
                // NOTE: memory-safe because it reads from memory already allocated by solidity (the bytes memory result)
                assembly ("memory-safe") {
                    revert(add(result, 32), mload(result))
                }
            }


file:/contracts/multicall/Multicall.sol
https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/multicall/Multicall.sol#L14-L28
```
```solidity:
   468         for (uint256 i = 0; i < 4; ++i) {
file: /contracts/types/TokenId.sol
https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/types/TokenId.sol#L468
```
```


```

## [G-8] Remove or replace unused state variables
Saves a storage slot. If the variable is assigned a non-zero value, saves Gsset (20000 gas). If it’s assigned a zero value, saves Gsreset (2900 gas). If the variable remains unassigned, there is no gas savings unless the variable is public, in which case the compiler-generated non-payable getter deployment cost is saved. If the state variable is overriding an interface’s public function, mark the variable as constant or immutable so that it does not use a storage slot
```
solidity

 67      mapping(address owner => mapping(address operator => bool approvedForAll))
   file:/contracts/tokens/ERC1155Minimal.sol
   https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/tokens/ERC1155Minimal.sol#L67
   ```
   ```solidity
 62   mapping(address account => mapping(uint256 tokenId => uint256 balance)) public balanceOf;
   file:/contracts/tokens/ERC1155Minimal.sol
   https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/tokens/ERC1155Minimal.sol#L62-L62
   ```
 ## [G-9]Don’t initialize default values to variables to reduce gas
   Saves 13 GAS for local variable and 2000 GAS for state variabl
   ```solidity
     550      for (uint256 i = 0; i < ids.length; ) {
    file : /contracts/SemiFungiblePositionManager.sol
    https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L550
    ```
    ```solidity
      187          for (uint256 i = 0; i < owners.length; ++i) {
    file:/contracts/tokens/ERC1155Minimal.sol
    https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/tokens/ERC1155Minimal.sol#L187

    ```
    ```soldity 
       141     for (uint256 i = 0; i < ids.length; ) {
    file:/contracts/tokens/ERC1155Minimal.sol
    https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/tokens/ERC1155Minimal.sol#L141
    ```
    ```slidity
     114   using LeftRight for uint256;
     115   using LeftRight for int256;
    file:/contracts/types/LeftRight.sol
    https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/types/LeftRight.sol#L14-L15
    ```
    ```solidity
      468          for (uint256 i = 0; i < 4; ++i) {
    
     file:/contracts/types/TokenId.sol
     https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/types/TokenId.sol#L468
     ```