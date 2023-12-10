# 🛠️ Analysis - Panoptic Audit
### Summary
| List |Head |Details|
|:--|:----------------|:------|
|a) |The approach I followed when reviewing the code | Stages in my code review and analysis |
|b) |Analysis of the code base | What is unique? How are the existing patterns used? "Solidity-metrics" was used  |
|c) |Test analysis | Test scope of the project and quality of tests |
|d) |Security Approach of the Project | Audit approach of the Project |
|e) |Other Audit Reports and Automated Findings | What are the previous Audit reports and their analysis |
|f) |Packages and Dependencies Analysis | Details about the project Packages |
|g) |Other recommendations | What is unique? How are the existing patterns used? |
|h) |New insights and learning from this audit | Things learned from the project |



## a) The approach I followed when reviewing the code

First, by examining the scope of the code, I determined my code review and analysis strategy.
https://github.com/code-423n4/2023-11-panoptic

Accordingly, I analyzed and audited the subject in the following steps;

| Number |Stage |Details|Information|
|:--|:----------------|:------|:------|
|1|Compile and Run Test|[Installation](https://github.com/code-423n4/2023-11-panoptic#tests)|Test and installation structure is simple, cleanly designed|
|2|Architecture Review| [Panoptic](https://panoptic.xyz/docs) |Provides a basic architectural teaching for General Architecture|
|3|Graphical Analysis  |Graphical Analysis with [Solidity-metrics](https://github.com/ConsenSys/solidity-metrics)|A visual view has been made to dominate the general structure of the codes of the project.|
|4|Slither Analysis  | [Slither Report](https://github.com/code-423n4/2023-11-panoptic/blob/main/slither.txt)| |
|5|Test Suits|[Tests](https://github.com/code-423n4/2023-11-panoptic#tests)|In this section, the scope and content of the tests of the project are analyzed.|
|6|Manuel Code Review|[Scope](https://github.com/code-423n4/2023-11-panoptic#scope)||
|7|Infographic|[Figma](https://www.figma.com/)|I made Visual drawings to understand the hard-to-understand mechanisms|
|8|Special focus on Areas of  Concern|[Areas of Concern](https://github.com/code-423n4/2023-11-panoptic#main-invariants)||

## b) Analysis of the code base

The most important summary in analyzing the code base is the stacking of codes to be analyzed.
In this way, many predictions can be made, including the difficulty levels of the contracts, which one is more important for the auditor, the features they contain that are important for security (payable functions, uses assembly, etc.), the audit cost of the project, and the time to be allocated to the audit;
Uses Consensys Solidity Metrics
-  **Lines:** total lines of the source unit
-  **nLines:** normalized lines of the source unit (e.g. normalizes functions spanning multiple lines)
-  **nSLOC:** normalized source lines of code (only source-code lines; no comments, no blank lines)
-  **Comment Lines:** lines containing single or block comments
-  **Complexity Score:** a custom complexity score derived from code statements that are known to introduce code complexity (branches, loops, calls, external interfaces, ...)

<img width="961" alt="image" src="https://github.com/code-423n4/2023-11-panoptic/assets/104318932/7fd6fffb-0b06-4765-b7c8-0d51747f9afc">




</br>
</br>

## Entry Point of Project;
<img width="793" alt="image" src="https://github.com/code-423n4/2023-11-panoptic/assets/104318932/f1c88273-f8aa-48e9-a181-6012c591ccc7">

</br>
</br>

## Protocol Roles;
<img width="1047" alt="image" src="https://github.com/code-423n4/2023-11-panoptic/assets/104318932/e451d56e-3615-4865-8b61-4681d14230f8">

</br>
</br>

## Role 1 : Option Sellers
Sell options by borrowing liquidity for a fixed commission fee and relocating it to a Uni v3 pool. Sellers must deposit collateral and can sell options with notional values up to five times larger than their collateral balance
How does selling options work?
<img width="1422" alt="image" src="https://github.com/code-423n4/2023-11-panoptic/assets/104318932/d42960dd-0ed3-461f-9b1c-162c3ce5d6ba">


</br>
</br>

##  Role 2 : Option Buyers
Buy options by moving liquidity out of the Uni v3 pool back to the Panoptic smart contract for a fixed commission fee. Buyers also have to deposit collateral (10% of the notional value of the option) to cover the potential premium to be paid to the sellers.
How does buying options work?
<img width="1528" alt="image" src="https://github.com/code-423n4/2023-11-panoptic/assets/104318932/ff08102a-164e-4a93-b90b-260ce18f3026">

</br>
</br>

##  Role 3 : Panoptic Liquidity Providers 
Provide fungible liquidity to the options market. This liquidity will be lent out to the options traders to allow them to access trading on leverage. Funds can be deposited into Panoptic pools at any ratio;

<img width="830" alt="image" src="https://github.com/code-423n4/2023-11-panoptic/assets/104318932/98dc6172-d3f0-4534-bb6d-09a44c342bad">

</br>
</br>

##  Role 4 : Liquidators
Ensure the health of the protocol by liquidating accounts whose collateral balance falls below the margin requirements. Liquidators will receive a [bonus](https://panoptic.xyz/docs/panoptic-protocol/liquidations#liquidation-bonus) proportional to:

- The amount of funds necessary to cover the distressed positions
- The distance between the strike and the current price
- The in-the-money amount (intrinsic value)
<img width="1025" alt="image" src="https://github.com/code-423n4/2023-11-panoptic/assets/104318932/f2686124-cb52-452e-9935-0dcc17685c5a">


</br>
</br>

##  Contracts


##   

<h1 align="center">  SemiFungiblePositionManager.sol  </h1>

<img width="1570" alt="image" src="https://github.com/code-423n4/2023-11-panoptic/assets/104318932/97649d1e-4ec3-49ab-9bf4-d669634a6540">

</br>
</br>


##  SemiFungiblePositionManager.sol -  [ State Variables - Mappings - Modifiers - Events]
<img width="613" alt="image" src="https://github.com/code-423n4/2023-11-panoptic/assets/104318932/ff1b483f-1965-440e-a86a-c77bc7bf84da">

</br>
</br>

##  SemiFungiblePositionManager.sol -  [ function initializeAMMPool() ]
<img width="1585" alt="image" src="https://github.com/code-423n4/2023-11-panoptic/assets/104318932/6a46eedb-07df-4d0a-b02b-7f75b25d3ba4">

</br>
</br>

<img width="1014" alt="image" src="https://github.com/code-423n4/2023-11-panoptic/assets/104318932/f7ec9d32-84dd-423f-a1d5-763648e6deee">

</br>
</br>
</br>

##  SemiFungiblePositionManager.sol -  [ function uniswapV3MintCallback() ]
<img width="1583" alt="image" src="https://github.com/code-423n4/2023-11-panoptic/assets/104318932/dc9b96cd-e6f3-49fe-963d-54d102b8cd4a">

</br>

<img width="1006" alt="image" src="https://github.com/code-423n4/2023-11-panoptic/assets/104318932/46d504b7-65bb-4f58-9a9a-88078f4976e6">

</br>
</br>
</br>

##  SemiFungiblePositionManager.sol -  [ function uniswapV3SwapCallback() ]
<img width="1580" alt="image" src="https://github.com/code-423n4/2023-11-panoptic/assets/104318932/73850ff4-a17b-4a13-83d7-eb8fa47a50b0">

</br>

<img width="1005" alt="image" src="https://github.com/code-423n4/2023-11-panoptic/assets/104318932/08237417-7990-4526-ba12-1ebda9c2faa0">

</br>
</br>
</br>

##  SemiFungiblePositionManager.sol -  [ function burnTokenizedPosition() ]
<img width="1575" alt="image" src="https://github.com/code-423n4/2023-11-panoptic/assets/104318932/fdbef9f6-fd31-4c1b-bc0b-5fa182e78eb5">

</br>

<img width="1004" alt="image" src="https://github.com/code-423n4/2023-11-panoptic/assets/104318932/7937b1d7-8de3-4242-a84d-59d45458593e">


</br>
</br>
</br>

##  SemiFungiblePositionManager.sol -  [ function mintTokenizedPosition() ]
<img width="1494" alt="image" src="https://github.com/code-423n4/2023-11-panoptic/assets/104318932/20c592f5-1389-48a2-88f1-2fb79fd726fc">

</br>

<img width="1009" alt="image" src="https://github.com/code-423n4/2023-11-panoptic/assets/104318932/b2ebc5c0-07c4-487a-9cbc-2b9cefd4f820">

</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>



The most fundamental point of the project and the most critical area to look at for security is the architecture below, which starts with `burnTokenizedPosition` and `mintTokenizedPosition` and transferred to uniswap V3 with `swapInAMM`. Therefore, we should take a closer look at this architecture and examine it in the first place;

<img width="903" alt="image" src="https://github.com/code-423n4/2023-11-panoptic/assets/104318932/1a44d3f7-60b1-4e92-b91d-9b911ffb396b">

<img width="1313" alt="image" src="https://github.com/code-423n4/2023-11-panoptic/assets/104318932/adb67605-f010-48d0-a5b0-036c29cecccf">


</br>
</br>
</br>
</br>
</br>
</br>




##   <h1 align="center">  Math.sol </h1>

<img width="1088" alt="image" src="https://github.com/code-423n4/2023-11-panoptic/assets/104318932/4a61df7e-1898-43b0-a647-5fdf9c819fa2">


## c) Test analysis
### What did the project do differently? ;
-   1) It can be said that the developers of the project did a quality job, there is a test structure consisting of tests with quality content.

-   2) Overall line coverage percentage provided by your tests : 100 




### What could they have done better?


-  1) In order to understand the test scenarios and develop more effective test scenarios, the following bob, alice and other roles are can be defined one by one, in this way role definitions increase the quality and readability in tests

```solidity

 // Sample labels
vm.label(bob, 'bob');
vm.label(alice, 'alice');
vm.label(DEPLOYER, 'deployer');
vm.label(USDE_OWNER, 'usde owner');
vm.label(POOL_PROXY, 'lending pool');
```
</br>

-  2) Test suites do not test for re-entrancy Test teams are testing many functions and variables, but recently, due to the vulnerability in the Vyper Compiler, the hacking of the projects using certain Vyper compiler and losing 50 million $ has revealed the security weakness here. 
https://cointelegraph.com/news/curve-finance-pools-exploited-over-24-reentrancy-vulnerability
The accuracy of the functions has been tested, but it has not been tested whether the nonReentrant modifier in the function works correctly or not. For this, a test must be written that tries to reentrancy and was observed to fail.

</br>

-  3) If we look at the test scope and content of the project with a systematic checklist, we can see which parts are good and which areas have room for improvement As a result of my analysis, those marked in green are the ones that the project has fully achieved. The remaining areas are the development areas of the project in terms of testing ;


<img width="781" alt="image" src="https://github.com/code-423n4/2023-11-panoptic/assets/104318932/650b8b29-e29a-4021-98dd-9f95dbc54d55">

Ref:https://xin-xia.github.io/publication/icse194.pdf

</br>
</br>

-  4) Based on the summary of the 'Math.sol' smart contract test file, it seems that the current tests cover a range of success and failure scenarios for various functions. However, there are additional tests that could be added to further ensure the robustness and reliability of the contract. Here are some suggestions:

</br>

<img width="1186" alt="image" src="https://github.com/code-423n4/2023-11-panoptic/assets/104318932/25a75511-dbbf-48f9-ab2f-61f526bea49e">

</br>
</br>
</br>
</br>
</br>


<img width="1191" alt="image" src="https://github.com/code-423n4/2023-11-panoptic/assets/104318932/d7600000-11bc-49f7-9eab-a9e8019c1811">

```solidity
// Test: Edge Case Tests for toUint128
    function test_EdgeCase_toUint128() public {
        // Test with the maximum value that can fit in uint128
        uint256 maxUint128 = type(uint128).max;
        assertEq(harness.toUint128(maxUint128), maxUint128, "toUint128 should correctly handle type(uint128).max");

        // Test with a value just above the maximum for uint128
        uint256 aboveMaxUint128 = maxUint128 + 1;
        vm.expectRevert(Errors.Overflow.selector);
        harness.toUint128(aboveMaxUint128);
    }
```

</br>
</br>


<img width="1184" alt="image" src="https://github.com/code-423n4/2023-11-panoptic/assets/104318932/dd6d1411-4a3b-4dea-9dcc-9a3042f412c3">

```solidity

// Test: Precision and Rounding for mulDiv64
    function test_PrecisionAndRounding_mulDiv64() public {
        uint96 a = 123456789;
        uint96 b = 987654321;
        uint256 expectedResult = FullMath.mulDiv(a, b, 2 ** 64);

        // Test for rounding down
        uint256 returnedResult = harness.mulDiv64(a, b);
        assertEq(returnedResult, expectedResult, "mulDiv64 should correctly round down the result");

        // Test for rounding up
        // Adjust 'a' and 'b' to create a scenario where rounding up is expected
        a = 123456789;
        b = 987654320;
        expectedResult = FullMath.mulDiv(a, b, 2 ** 64) + 1;
        returnedResult = harness.mulDiv64(a, b);
        assertEq(returnedResult, expectedResult, "mulDiv64 should correctly round up the result");
    }

    // Test: Precision and Rounding for mulDiv192
    function test_PrecisionAndRounding_mulDiv192() public {
        uint128 a = 123456789123;
        uint128 b = 987654321987;
        uint256 expectedResult = FullMath.mulDiv(a, b, 2 ** 192);

        // Test for rounding down
        uint256 returnedResult = harness.mulDiv192(a, b);
        assertEq(returnedResult, expectedResult, "mulDiv192 should correctly round down the result");

        // Test for rounding up
        // Adjust 'a' and 'b' to create a scenario where rounding up is expected
        a = 123456789123;
        b = 987654321986;
        expectedResult = FullMath.mulDiv(a, b, 2 ** 192) + 1;
        returnedResult = harness.mulDiv192(a, b);
        assertEq(returnedResult, expectedResult, "mulDiv192 should correctly round up the result");
    }

```

</br>
</br>


## d) Security Approach of the Project

### Successful current security understanding of the project;

1 - First they did the main audit from ABDK and OpenZeppelin and resolved all the security concerns in the report 

2- They manage the 2nd audit process with an innovative audit such as Code4rena, in which many auditors examine the codes.


### What the project should add in the understanding of Security;

1- By distributing the project to testnets, ensuring that the audits are carried out in onchain audit. (This will increase coverage)

2- After the project is published on the mainnet, there should be emergency action plans (not found in the documents)

3- Add On-Chain Monitoring System; If On-Chain Monitoring systems such as Forta are added to the project, its security will increase.

For example ; This bot tracks any DEFI transactions in which wrapping, unwrapping, swapping, depositing, or withdrawals occur over a threshold amount. If transactions occur with unusually high token amounts, the bot sends out an alert. https://app.forta.network/bot/0x7f9afc392329ed5a473bcf304565adf9c2588ba4bc060f7d215519005b8303e3

4- After the Code4rena audit is completed and the project is live, I recommend the audit process to continue, projects like immunefi do this. 
https://immunefi.com/

5- Pause Mechanism
This is a chaotic situation, which can be thought of as a choice between decentralization and security. Having a pause mechanism makes sense in order not to damage user funds in case of a possible problem in the project.

6- Upgradability
There are use cases of the Upgradable pattern in defi projects using mathematical models, but it is a design and security option.

7- Emergency Action Plan
In a high-level security approach, there should be a crisis handbook like the one below and the strategic members of the project should be trained on this subject and drills should be carried out. Naturally, this information and road plan will not be available to the public.
https://docs.google.com/document/u/0/d/1DaAiuGFkMEMMiIuvqhePL5aDFGHJ9Ya6D04rdaldqC0/mobilebasic#h.27dmpkyp2k1z

8- ChainAnalysis oracle
With the ChainAnalysis oracle, OFAC interaction can be blocked so that legal issues do not arise

</br>
</br>

## e) Other Audit Reports and Automated Findings 

**Automated Findings:**
https://github.com/code-423n4/2023-11-panoptic/blob/main/bot-report.md


**Other Audit Reports (ABDK and Openzeppelin):**
https://panoptic.xyz/docs/security/audits

**Economic Audit (Three Sigma):**
Economic Audit with [Three Sigma](https://panoptic.xyz/blog/panoptic-three-sigma-partnership)  (July 2023)

</br>
</br>

##  f) Packages and Dependencies Analysis 📦

| Package                                                                                                                                     | Version                                                                                                               | Usage in the project                               | Audit Recommendation                                                                                                             |
| ------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- | -------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| [`openzeppelin`](https://www.npmjs.com/package/@openzeppelin/contracts)         | [![npm](https://img.shields.io/npm/v/@openzeppelin/contracts.svg)](https://www.npmjs.com/package/@openzeppelin/contracts)     |               ERC1155Holder.sol |-  Version `4.8.3` is used by the project, it is recommended to use the newest version `5.0.0` |


</br>
</br>




## g) Other recommendations

✅ The use of assembly in project codes is very low, I especially recommend using such useful and gas-optimized code patterns; https://github.com/dragonfly-xyz/useful-solidity-patterns/tree/main/patterns/assembly-tricks-1

✅ A good model can be used to systematically assess the risk of the project, for example this modeling is recommended; https://www.notion.so/Smart-Contract-Risk-Assessment-3b067bc099ce4c31a35ef28b011b92ff#7770b3b385444779bf11e677f16e101e

✅ All staff accounts for the project should have control policies that require 2FA and must use 2FA wherever possible.
100% comprehensive security cannot be achieved based on smart contract codes alone.
Implement a more comprehensive policy to enforce non-SMS 2FA.
You can find the latest Simswap attack on Code4rena and details about it in this article: 
https://medium.com/code4rena/code4rena-twitter-x-incident-8b7f308a555d

✅ The Reentrancy modifier used by Opensea is more gas optimized and battle tested, I recommend you replace the reentrancy structure in the project with this code;
https://github.com/ProjectOpenSea/seaport/blob/main/contracts/lib/ReentrancyGuard.sol

✅ I recommend you to set up a `system.sol` basic architecture where all contracts are registered.
The entire system can revolve around a single contract, like SystemRegistry. This is the contract that ties all the other contracts together, and from this contract we should be able to list all the other contracts in the system. It's almost like a registry. 

✅ Math.sol contract has been partially forked from Uniswap V3, it would be a more accurate approach in terms of security to specify where Math.sol was forked. There is no information about this in the NatSpec comments.

✅ Analyze the gas usage of each function under various conditions in tests to ensure efficiency and identify potential optimizations.	

✅ I recommend running the test files in the same solidity version as the contracts, for example, the solidity version of the `SemiFungiblePositionManagerHarness.sol` file, which is the most important file of the control, is `pragma solidity = 0.8.18`, while the test file is `pragma solidity ^0.8.0`, it is recommended to choose the same structure for full coverage. is done

✅ `Multicall.multicall()` function is common pattern used in Ethereum smart contracts to execute multiple calls in a single transaction. This is particularly useful for optimizing transaction costs and efficiency. The original of this pattern is also available in Uniswap v3.

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

✅  I recommend that when designing economic models, boundary conditions should be thoroughly tested, and liquidity and price calculations should be strictly evaluated, rather than relying on inequality checks.
Kyberswap was a platform that used Uniswap v3 pools and was hacked for this very reason and suffered a loss of $54 Million;

https://slowmist.medium.com/a-deep-dive-into-the-kyberswap-hack-3e13f3305d3a

Test functions appear to be well-structured for testing key aspects of a liquidity pool, including boundary conditions, liquidity calculations, and price impacts. However, it's important to note;
Given the dynamic nature of DeFi and the lessons learned from incidents like the Kyberswap hack, it's vital to continuously review and update the test suite to cover new scenarios and potential vulnerabilities.

</br>
</br>

## h) New insights and learning from this audit 

🔎 1. SemiFungiblePositionManager Struct
A gas-efficient alternative to Uniswap’s NonFungiblePositionManager that manages complex, multi-leg Uniswap positions encoded in ERC1155 tokenIds, performs swaps allowing users to mint positions with only one type of token, and, most crucially, supports the minting of both typical LP positions


🔎 2. CollateralTracker Struct
Each time positions are minted, burned, or rolled in Panoptic, the CollateralTracker updates the collateral balances and provides information on the collateral requirement, ensuring that the protocol remains solvent and we retain the ability to liquidate distressed positions when needed.

🔎 3. PanopticPool Struct
It is responsible for orchestrating the required calls to the SFPM to actually create option positions in Uniswap, tracking user balances of and accumulating the premia on those positions, and calling the CollateralTracker with the data it needs to settle position changes.

🔎 3. Great NatSpec Descriptions
The greatest thing you learned from the project was; These are NatSpec comments with great explanations and infographics. This is the best example I have ever seen, I must say that the developers have done an excellent job here, this method makes the codes more understandable and easier to control.

Here are some of them;

```solidity

--- SemiFungiblePositionManager.sol ---

    /** 
        We're tracking the amount of net and removed liquidity for the specific region:

             net amount    
           received minted  
          ▲ for isLong=0     amount           
          │                 moved out      actual amount 
          │  ┌────┐-T      due isLong=1   in the UniswapV3Pool 
          │  │    │          mints      
          │  │    │                        ┌────┐-(T-R)  
          │  │    │         ┌────┐-R       │    │          
          │  │    │         │    │         │    │     
          └──┴────┴─────────┴────┴─────────┴────┴──────►                     
             total=T       removed=R      net=(T-R)


     *       removed liquidity r          net liquidity N=(T-R)
     * |<------- 128 bits ------->|<------- 128 bits ------->|
     * |<---------------------- 256 bits ------------------->|
     */



    ///   ┌───────────────────────┐
    ///   │mintTokenizedPosition()├───┐
    ///   └───────────────────────┘   │
    ///                               │
    ///   ┌───────────────────────┐   │   ┌───────────────────────────────┐
    ///   │burnTokenizedPosition()├───────► _validateAndForwardToAMM(...) ├─ (...) --> (mint/burn in AMM)
    ///   └───────────────────────┘       └───────────────────────────────┘



/// @notice When a position is minted or burnt in-the-money (ITM) we are *not* 100% token0 or 100% token1: we have a mix of both tokens.
    /// @notice The swapping for ITM options is needed because only one of the tokens are "borrowed" by a user to create the position.
    /// @notice This is an ITM situation below (price within the range of the chunk):
    ///
    ///        AMM       strike
    ///     liquidity   price tick
    ///        ▲           │
    ///        │       ┌───▼───┐
    ///        │       │       │liquidity chunk
    ///        │ ┌─────┴─▲─────┴─────┐
    ///        │ │       │           │
    ///        │ │       :           │
    ///        │ │       :           │
    ///        │ │       :           │
    ///        └─┴───────▲───────────┴─► price
    ///                  │
    ///             current price
    ///             in-the-money: mix of tokens 0 and 1 within the chunk
    ///
    ///   If we take token0 as an example, we deploy it to the AMM pool and *then* swap to get the right mix of token0 and token1
    ///   to be correctly in the money at that strike.
    ///   It that position is burnt, then we remove a mix of the two tokens and swap one of them so that the user receives only one.



----- TokenId.sol -----

/// @dev From the LSB to the MSB:
/// ===== 1 time (same for all legs) ==============================================================
///      Property         Size      Offset      Comment
/// (1) univ3pool        64bits     0bits      : first 8 bytes of the Uniswap v3 pool address (first 64 bits; little-endian), plus a pseudorandom number in the event of a collision
/// ===== 4 times (one for each leg) ==============================================================
/// (2) asset             1bit      0bits      : Specifies the asset (0: token0, 1: token1)
/// (3) optionRatio       7bits     1bits      : number of contracts per leg
/// (4) isLong            1bit      8bits      : long==1 means liquidity is removed, long==0 -> liquidity is added
/// (5) tokenType         1bit      9bits      : put/call: which token is moved when deployed (0 -> token0, 1 -> token1)
/// (6) riskPartner       2bits     10bits     : normally its own index. Partner in defined risk position otherwise
/// (7) strike           24bits     12bits     : strike price; defined as (tickUpper + tickLower) / 2
/// (8) width            12bits     36bits     : width; defined as (tickUpper - tickLower) / tickSpacing
/// Total                48bits                : Each leg takes up this many bits
/// ===============================================================================================
///
/// The bit pattern is therefore, in general:
///
///                        (strike price tick of the 3rd leg)
///                            |             (width of the 2nd leg)
///                            |                   |
/// (8)(7)(6)(5)(4)(3)(2)  (8)(7)(6)(5)(4)(3)(2)  (8)(7)(6)(5)(4)(3)(2)   (8)(7)(6)(5)(4)(3)(2)        (1)
///  <---- 48 bits ---->    <---- 48 bits ---->    <---- 48 bits ---->     <---- 48 bits ---->    <- 64 bits ->
///         Leg 4                  Leg 3                  Leg 2                   Leg 1         Univ3 Pool Address
///
```

### Time spent:
26 hours