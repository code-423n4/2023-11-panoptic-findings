<div align="center">
  <h1> Panoptic</h1>
  <h5>Effortless options trading on any token, any strike, any size.</h5>
</div>

## Index
- [Index](#index)
- [1. Panoptic: A Comprehensive Overview](#1-panoptic-a-comprehensive-overview)
- [2. All the contracts and the analyses of them](#2-all-the-contracts-and-the-analyses-of-them)
  - [1. SemiFungiblePositionManager Contract :](#1-semifungiblepositionmanager-contract-)
  - [2. ERC1155Minimal Contract:](#2-erc1155minimal-contract)
  - [3. LeftRight Contract:](#3-leftright-contract)
  - [4. LiquidityChunk Contract:](#4-liquiditychunk-contract)
  - [5. TokenId Contract:](#5-tokenid-contract)
  - [6. CallbackLib Contract:](#6-callbacklib-contract)
  - [7. Constants Contract :](#7-constants-contract-)
  - [8. Errors Contract:](#8-errors-contract)
  - [9. FeesCalc Contract:](#9-feescalc-contract)
  - [10. Math Contract:](#10-math-contract)
  - [11. PanopticMath Contract:](#11-panopticmath-contract)
  - [12. SafeTransferLib Contract:](#12-safetransferlib-contract)
  - [13. Multicall Contract:](#13-multicall-contract)
- [3. Panoptic Architecture](#3-panoptic-architecture)
- [4. Systemic Risks](#4-systemic-risks)
- [5. Security Approach of the Project](#5-security-approach-of-the-project)
- [New insights and learnings from the Panoptic project](#new-insights-and-learnings-from-the-panoptic-project)
  - [Handling Positions in Uniswap V3:](#handling-positions-in-uniswap-v3)
  - [Interaction with Uniswap and Extended Functionality:](#interaction-with-uniswap-and-extended-functionality)
  - [Gas Efficiency:](#gas-efficiency)
  - [Options Integration:](#options-integration)

**The Process and Steps We Followed for Codebase Evaluation**

Our approach to analyzing the source code of the Panoptic Protocol was to simplify the information provided by the protocol, using a variety of diagrams to visually clarify the project's key contracts and break down each important part of these contracts. This enhances understanding for developers, security researchers, and users alike. We identified the fundamental concepts and employed simpler language to explain the functionality and goals of the Panoptic Protocol. Furthermore, we organized the information logically into separate sections, each with identifying titles, to provide a clear overall picture of the subject. Our primary goal was to make the information more accessible and easy to understand.

## 1. Panoptic: A Comprehensive Overview

**Panoptic** is the perpetual, oracle-free options protocol, addresses the limitation of **Uniswap v3** by introducing an innovative solution that enables Liquidity Providers to earn additional yields by lending their liquidity tokens. This is achieved through a **Semi-Fungible Position Manager**, allowing LPs to earn beyond traditional fees by participating in **Panoptic's options market**.
![Overview](https://github.com/catellaTech/Panoptic-charts/blob/main/Overview.drawio.png?raw=true)

## 2. All the contracts and the analyses of them
**The scope provided by the protocol involved 3 contracts and 10 libraries. Let's take a closer look at each of them:**
### 1. SemiFungiblePositionManager Contract :
- The core functionality of Panoptic, the SemiFungiblePositionManager (SFPM) is a gas-efficient alternative to Uniswap's NonFungiblePositionManager (NFPM). It efficiently manages complex, multi-leg Uniswap positions encoded in ERC1155 tokenIds. The SFPM facilitates swaps, allowing users to mint positions with only one type of token. Importantly, it supports the minting of both typical LP positions, where liquidity is added to Uniswap, and "long" positions, where Uniswap liquidity is burnt. While it is a core component of the Panoptic V1 protocol, the SFPM can also function as a standalone liquidity manager available for use by any user or protocol.

![SemiFungiblePositionManager](https://github.com/catellaTech/Panoptic-charts/blob/main/SemiFungiblePositionManager.drawio.png?raw=true)

### 2. ERC1155Minimal Contract:
- A simplified realization of the ERC1155 token standard, focusing on essential functionality without including metadata features.
![ERC1155Minimal](https://github.com/catellaTech/Panoptic-charts/blob/main/ERC1155Minimal.drawio.png?raw=true)

### 3. LeftRight Contract:
- Design and implementation of specialized data structures capable of storing two 128-bit numerical values.
![LeftRight](https://github.com/catellaTech/Panoptic-charts/blob/main/LeftRight.drawio.png?raw=true)

### 4. LiquidityChunk Contract:
- Design and implementation of a tailored data type capable of representing a Uniswap liquidity chunk with specific attributes, including tickLower, tickUpper, and liquidity.
![LiquidityChunk](https://github.com/catellaTech/Panoptic-charts/blob/main/LiquidityChunk.drawio.png?raw=true)

### 5. TokenId Contract:
- Implementation of a customized data type utilized in the Semi-Fungible Position Manager (SFPM) and Panoptic to encode position data within 256-bit ERC1155 tokenIds. This data type encompasses a pool identifier and accommodates up to four complete position legs.
![TokenId](https://github.com/catellaTech/Panoptic-charts/blob/main/TokenId.drawio.png?raw=true)

### 6. CallbackLib Contract:
- Library for verifying and decoding callbacks from Uniswap.
![CallbackLib](https://github.com/catellaTech/Panoptic-charts/blob/main/CallbackLib.drawio.png?raw=true)

### 7. Constants Contract :
- Library of Constants used in Panoptic.
> 💡Note: We did not create diagrams for this library as we believe its description is clear enough. However, if someone reading this is not familiar with the concept, constant variables are those whose value remains unchanged. In this case, they have been written in a separate file to maintain code readability and organization.

### 8. Errors Contract:
- Stores all custom errors utilized in the core contracts of Panoptic.
> 💡Note: As for the "constants" library, we refrained from creating diagrams as its purpose is quite clear.

### 9. FeesCalc Contract:
- Utility for calculating current swap fees for liquidity chunks.
![FeesCalc](https://github.com/catellaTech/Panoptic-charts/blob/main/FeesCalc.drawio.png?raw=true)

### 10. Math Contract:
- Library containing generic math functions, including abs(), mulDiv, etc.
![CallbackLib](https://github.com/catellaTech/Panoptic-charts/blob/main/Math.drawio.png?raw=true)

### 11. PanopticMath Contract:
- Library encompassing advanced Panoptic/Uniswap-specific functionality, including features such as TWAP, price conversions, and position sizing math.
![PanopticMath](https://github.com/catellaTech/Panoptic-charts/blob/main/PanopticMath.drawio.png?raw=true)

### 12. SafeTransferLib Contract:
- Safe ERC20 transfer library designed to handle token transfers gracefully, even in scenarios where return values might be missing.
![SafeTransferLib](https://github.com/catellaTech/Panoptic-charts/blob/main/SafeTransferLib.drawio.png?raw=true)

### 13. Multicall Contract:
- Abstract contract the extends inheriting contracts with a function enabling the execution of multiple calls within a single transaction.
![Multicall](https://github.com/catellaTech/Panoptic-charts/blob/main/Multicall.drawio.png?raw=true)

## 3. Panoptic Architecture 
![PanopticArch](https://github.com/catherinee24/img/blob/main/img/Captura%20de%20pantalla%202023-12-01%20145619.png?raw=true)
The protocol's architecture is divided into multiple contracts and specialized libraries to address specific aspects of the protocol, facilitating modularity and updates. The choice to use the ERC-1155 standard stands out as a strategic decision. This standard incorporates optimizations that enhance both the efficiency and security of transactions. The ability to batch transactions results in a significant reduction in the costs associated with token transfers. It's noteworthy that ERC-1155 represents an upgrade from older token standards like ERC-20 and ERC-721. This improvement not only provides additional efficiency but also reflects an evolutionary approach to token standards in the blockchain ecosystem.

## 4. Systemic Risks
> When reviewing the source code and documentation, we encountered questions where we could highlight certain aspects that we find important regarding potential systemic risks
- **Security of ERC-20:**
   - Ensure that ERC-20 token transfer operations are secure and avoid potential vulnerabilities such as reentrancy.
   
   - Carefully validate the results of operations and handle possible errors appropriately.

- **Complexity and Efficiency:**
   - Consider gas efficiency, especially in critical operations such as transfers and mathematical calculations.

- **Testing:** 
  - The audit scope of the contracts to be reviewed is 100% but we recommended to make a test suite of invariants tests in the future to increase the safety of the project and to validate the code's robustness.

## 5. Security Approach of the Project
What the project can add in the understanding of security:
  - After the Code4rena audit is completed and the project is live, I recommend the audit process to continue, projects like [immunefi](https://immunefi.com/).

## New insights and learnings from the Panoptic project 

### Handling Positions in Uniswap V3:

The introduction of the **SemiFungiblePositionManager** addresses the limitation of LP tokens in Uniswap V3, allowing Liquidity Providers to earn additional returns by lending their LP tokens.

### Interaction with Uniswap and Extended Functionality:

Specific functions are provided to interact with Uniswap V3, calculate fees, manage complex positions, and perform advanced operations.

### Gas Efficiency:

Gas efficiency is emphasized in various contracts, such as **SafeTransferLib**, which handles secure transfers of ERC-20 tokens efficiently.

### Options Integration:

The protocol seems to incorporate functionalities related to financial options, allowing LPs to earn additional returns from options buyers.
 

### Time spent:
15 hours