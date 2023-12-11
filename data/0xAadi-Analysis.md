## Index

1. [Overview](#overview)
2. [Contracts](#contracts)
3. [Auditing Approach](#auditing-approach)
4. [Codebase Quality Analysis](#codebase-quality-analysis)
   - [Modularity](#modularity)
   - [Comments](#comments)
   - [Libraries](#libraries)
   - [Access Controls](#access-controls)
   - [Events](#events)
   - [Error Handling](#error-handling)
5. [Centralisation Risk Analysis](#centralisation-risk-analysis)
6. [Systemic Risk Analysis](#systemic-risk-analysis)
7. [Recommendations](#recommendations)

## Overview

The Panoptic protocol facilitates the creation of option contracts for **UniswapV3** positions. It enables option sellers to create option contracts with up to four legs, such as covered PUT/CALL options. This is achieved by substituting the **UniswapV3** `FungiblePositionManager`(ERC721) contract with Panoptic's `SemiFungiblePositionManager`(ERC1155). The protocol empowers users to trade their liquidity positions seamlessly as option positions.

## Contracts
The pivotal contracts in this Audit for the protocol are:
- **contracts/multicall/Multicall.sol** — An abstract contract, inherited by the core contract `SemiFungiblePositionManager.sol`, provides a function to execute multiple calls in a single transaction. The purpose of this contract is to provide flexibility to `SemiFungiblePositionManager` to create multiple positions and cover positions in a single transaction.
- **contracts/tokens/ERC1155Minima.sol** — This abstract contract is a minimalist implementation of the `ERC1155` token standard without metadata, adapted from Solmate's minimalist implementation of the `ERC1155` token. This contract is inherited by `SemiFungiblePositionManager.sol` to facilitate the implementation of option positions. It is specifically employed in SemiFungiblePositionManager to handle positions, effectively replacing the `ERC721` implementation in the **UniswapV3**’s `FungiblePositionManager` contract.
- **contracts/SemiFungiblePositionManager.sol** — The core contract that executes the core business logic, encompassing the minting and burning of options, by providing liquidity to **UniswapV3** pools.
- **contracts/types/LeftRight.sol** — This contract efficiently stores option details by encoding the data in a custom data type capable of holding two 128-bit numbers, thereby optimizing gas usage.
- **contracts/types/LiquidityChunk.sol** — Implementation for a specialized data type capable of representing a Uniswap liquidity chunk of a specified size. This data type encapsulates a `tickLower`, `tickUpper`, and `liquidity`.
- **contracts/types/TokenId.sol** — This custom data type is designed to encode position data within 256-bit `ERC1155` tokenIds, accommodating a pool identifier and supporting up to four complete position legs.
- **contracts/libraries/CallbackLib.sol** — This contract serves **Panoptic** by ensuring the verification and decoding of Uniswap callbacks, confirming that the calls originate from **UniswapV3**Pool contracts.
- **contracts/libraries/Constants.sol** — A unified space for constants used in **Panoptic**.
- **contracts/libraries/Errors.sol** — A unified space for custom errors used in **Panoptic**'s core contracts.
- **contracts/libraries/FeesCalc.col** — Contract helps to calculate swap fees for liquidity chunks.
- **contracts/libraries/Math.sol** — Library of generic math functions.
- **contracts/libraries/PanopticMath.sol** — Library containing advanced Panoptic/Uniswap-specific math.
- **contracts/libraries/SafeTransferLib.sol** — Safe ERC20 transfer library.

## Auditing Approach

In the process of auditing the smart contracts, I delved into comprehending the details of both the `FungiblePositionManager` contract in **UniswapV3** and the `SemiFungiblePositionManager` contract in **Panoptic**, which are pivotal in creating option positions. My approach involved a meticulous understanding of the code base, conducting a thorough code review to ensure solidity and security. To validate the functionality and robustness of the contracts, I actively engaged in compiling the code and experimenting with various test cases. This comprehensive examination allowed me to assess the contracts' resilience and functionality under different scenarios, contributing to a holistic evaluation of their performance and security measures.

## Codebase Quality Analysis

The codebase under scrutiny demonstrates exceptional quality, boasting both high readability and thorough documentation through well-placed comments. Despite grappling with intricate concepts, the code remains explanatory, ensuring auditors and developers can easily comprehend its functionality.

### Modularity

The codebase exhibits a high degree of modularity, effectively segregating complex additional logics into libraries, particularly for mathematics and fee calculations. Additionally, distinct contract types are utilized for data encoding, streamlining the storage of option details. Notably, the `SemiFungiblePositionManager` contract enhances readability by segregating business logics for minting and burning options into separate functions.

### Comments

All contracts are meticulously commented, offering valuable insights into functionality. This proves invaluable for auditors and developers, aiding in visualizing how options behave within specific tick ranges. Comments also elucidate the fee distribution or collection processes, contributing to a comprehensive understanding of this complex protocol.

### Libraries

The protocol's contracts have a minimal number of external libraries, with well-coded and commented implementations. All the contracts used named imports of libraries, which enhance the overall quality of the contracts.

### Access Controls

A noteworthy feature is the absence of access controls on any protocol functions, promoting a high degree of decentralization, transparency, and simplicity.

### Events

User-interacting functions exposed by the `SemiFungiblePositionManager` contract emit necessary events, all appropriately indexed. Notable events include `PoolInitialized`, `TokenizedPositionBurnt`, `TokenizedPositionMinted`, `TransferSingle`, and `TransferBatch`.

### Error Handling

The contracts excel in error handling, particularly the `SemiFungiblePositionManager` contract, which benefits from the dedicated `contracts/libraries/Errors.sol`.

However, one defined error, `NoLegsExercisable()`, is unused and can be safely removed to reduce contract size. Additionally, it is advisable to implement error handling in `safeTransferFrom()`, `safeBatchTransferFrom()`, and `_burn()` for scenarios where the from address lacks sufficient tokens for the operation.

## Centralisation Risk Analysis
The **Panoptic** protocol is implemented as a purely decentralized application without any elements of centralization. The contracts within its scope do not contain any functions that require exclusive access or governance. Furthermore, the protocol interacts with Uniswap V3, which is also a decentralized application, thereby minimizing the risk of centralization within **Panoptic**.

## Systemic Risk Analysis
Due to the lack of centralization risks, Governance, oracles, major Smart Contract Vulnerabilities or Protocol Interdependencies(except **uniswapV3**) The **panoptic** protocols seem to have a high degree of protection against systemic risks. However any future attacks on **uniswapv3** will affect the **Panoptic** protocol. 

## Recommendations

1. Reduce excessive use of `unchecked` block.
2. Properly handle low balance errors in `safeTransferFrom()`, `safeBatchTransferFrom()`, and `_burn()` methods in `contracts/tokens/ERC1155Minima.sol`.
3. Properly handle transfer the tokens to zero addresses in `safeTransferFrom()` and `safeBatchTransferFrom()` methods in `contracts/tokens/ERC1155Minima.sol`.


### Time spent:
36 hours