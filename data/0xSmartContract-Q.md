##### QA Report Issues List

- [x] **Low 01** → Use a more recent version of OpenZeppelin dependencies
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
