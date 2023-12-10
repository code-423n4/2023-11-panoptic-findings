##### QA Report Issues List

- [x] **Low 01** → Use a more recent version of OpenZeppelin dependencies
- [x] **Low 02** → `Multicall.multicall()` function raw revert pattern should be change
- [x] **Non-Critical 01** → Test suites do not test for attack vectors, especially re-entrancy
- [x] **Non-Critical 02** → Project Upgrade and Stop Scenario should be
- [x] **Non-Critical 03** → Missing Event for  initialize
- [x] **Non-Critical 04** → Use descriptive names for Contracts and Libraries


### [Low-1] Use a more recent version of OpenZeppelin dependencies

**Context:**
All contracts

**Description:**
For security, it is best practice to use the latest OZ version.


```js
lib/openzeppelin-contracts/package-lock.json:
  2    "name": "openzeppelin-solidity",
  3:   "version": "4.8.3",
```



**Recommendation:**
Old version of OZ is used `(4.8.3)`, newer version can be used `(5.0.0)` 

https://github.com/OpenZeppelin/openzeppelin-contracts/releases/tag/v5.0.0


### [Low-2] `Multicall.multicall()` function raw revert pattern should be change

**Description:**

`Multicall.multicall()` function is common pattern used in Ethereum smart contracts to execute multiple calls in a single transaction. This is particularly useful for optimizing transaction costs and efficiency. The original of this pattern is also available in Uniswap v3.

Raw Revert Reason: It reverts with the raw error message directly from the call. This approach is straightforward but might not always provide clear, human-readable error messages.

Therefore I recommend updating it


```diff
contracts/multicall/Multicall.sol:
  12:     function multicall(bytes[] calldata data) public payable returns (bytes[] memory results) {
  13:         results = new bytes[](data.length);
  14:         for (uint256 i = 0; i < data.length; ) {
  15:             (bool success, bytes memory result) = address(this).delegatecall(data[i]);
  16: 
  17:             if (!success) {
  18:                 // Bubble up the revert reason
  19:                 // The bytes type is ABI encoded as a length-prefixed byte array
  20:                 // So we simply need to add 32 to the pointer to get the start of the data
  21:                 // And then revert with the size loaded from the first 32 bytes
  22:                 // Other solutions will do work to differentiate the revert reasons and provide paranthetical information
  23:                 // However, we have chosen to simply replicate the the normal behavior of the call
  24:                 // NOTE: memory-safe because it reads from memory already allocated by solidity (the bytes memory result)
- 25:                 assembly ("memory-safe") {
- 26:                     revert(add(result, 32), mload(result))
- 27:                 }

+		if (result.length < 68) revert();
+   		          assembly {
+	                          result := add(result, 0x04)
                }
                revert(abi.decode(result, (string)));
  28:             }
  29: 
  30:             results[i] = result;
  31: 
  32:             unchecked {
  33:                 ++i;
  34:             }
  35:         }
  36:     }
  36  }
```



### [Non Critical-1] Test suites do not test for attack vectors, especially re-entrancy

Test teams are testing many functions and variables, but recently, due to the vulnerability in the Vyper Compiler, the hacking of the projects using certain Vyper compiler and losing 50 million $ has revealed the security weakness here.

Attack vectors, especially re-entrancy, seem untested and trusted

```solidity
contracts/SemiFungiblePositionManager.sol:
  474      /// @return newTick the current tick in the pool after all the mints and swaps
  475      /// @return newTick the current tick in the pool after all the mints and swaps
  476:     function burnTokenizedPosition(
  477:         uint256 tokenId,
  478:         uint128 positionSize,
  479:         int24 slippageTickLimitLow,
  480:         int24 slippageTickLimitHigh
  481:     )
  482:         external
  483:         ReentrancyLock(tokenId.univ3pool())
  484:         returns (int256 totalCollected, int256 totalSwapped, int24 newTick)
  485:     {


contracts/SemiFungiblePositionManager.sol:
  508      /// @return newTick the current tick in the pool after all the mints and swaps
  509      /// @return newTick the current tick in the pool after all the mints and swaps
  510:     function mintTokenizedPosition(
  511:         uint256 tokenId,
  512:         uint128 positionSize,
  513:         int24 slippageTickLimitLow,
  514:         int24 slippageTickLimitHigh
  515:     )
  516:         external
  517:         ReentrancyLock(tokenId.univ3pool())
  518:         returns (int256 totalCollected, int256 totalSwapped, int24 newTick)
  519:     {


```





### [Non Critical-2] Project Upgrade and Stop Scenario should be

At the start of the project, the system may need to be stopped or upgraded, I suggest you have a script beforehand and add it to the documentation.
This can also be called an " EMERGENCY STOP (CIRCUIT BREAKER) PATTERN ".

This can be done by adding the 'pause' architecture, which is included in many projects, to critical functions, and by authorizing the existing onlyOwner.

https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol


### [Non Critical-3] Missing Event for  initialize

Most of them do not use event-emit when setting the startup items in the project's constructor.



```solidity

contracts/SemiFungiblePositionManager.sol:
  340      /// @param _factory the Uniswap v3 Factory used to retrieve registered Uniswap pools
  341      /// @param _factory the Uniswap v3 Factory used to retrieve registered Uniswap pools
  342:     constructor(IUniswapV3Factory _factory) {
  343:         FACTORY = _factory;
  344:     }

```

### [Non Critical-4] Use descriptive names for Contracts and Libraries


This codebase will be difficult to navigate, as there are no descriptive naming conventions that specify which files should contain meaningful logic.

Prefixes should be added like this by filing:

Interface I_
absctract contracts Abs_
Libraries Lib_

We recommend that you implement this or a similar agreement.
