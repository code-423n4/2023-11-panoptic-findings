# Analysis - Panoptic Contest

![Panoptic-Protocol](https://code4rena.com/_next/image?url=https%3A%2F%2Fstorage.googleapis.com%2Fcdn-c4-uploads-v0%2Fuploads%2Fve7mSg8Pcp2.0&w=256&q=75)

## Description overview of The Panoptic Contest

The Panoptic protocol consists of smart contracts on Ethereum that allow for permissionless, decentralized trading of perpetual put and call options through direct user interaction without intermediaries, using Uniswap v3 liquidity provider positions as the underlying mechanism for option pricing and trading in a novel way compared to traditional options markets while enabling both retail and professional traders to access global options liquidity and provide liquidity themselves to earn fees but bearing the risks of leverage, unfavorable premiums, forced liquidations and margin calls.

**The key contracts of Panoptic protocol for this Audit are**:

Auditing the key contracts of the Panoptic Protocol is essential to ensure the security of the protocol. Focusing on them first will provide a solid foundation for understanding the protocol's operation and how it manages user assets and stability. Here's why it's important to audit these specific contracts:

- **SemiFungiblePositionManager.sol**: The contract enables the creation, management and settlement of synthetic options positions represented as ERC tokens by employing a sophisticated on-chain accounting model to precisely track individual liquidity exposures and variable option premium values over time for positions stored as mappings, while seamlessly integrating with Uniswap through functions that interface directly with AMM pools for atomic minting and burning of position liquidity chunks as well as fee collection to facilitate the use of decentralized exchange liquidity for options.

- **tokenId.sol**: This contract encoding complex multi-legged options positions into a single standardized uint256 tokenId.Validation of the encoded option positions and Tracking of core position details within the encoded tokenId.

- **FeesCalc.sol**: FeesCalc contract is crucial for calculating and managing fees in the context of options positions within an Automated Market Maker (AMM) or Uniswap-like environment.

-- **PanopticMath.sol**: This library contains advanced Panoptic/Uniswap-specific functionality, including TWAP (Time-Weighted Average Price) calculations, price conversions, and position sizing math. It suggests a central role in handling critical mathematical computations related to the protocol's core functionalities.

As the audit of these contracts, I checked for potential security vulnerabilities, such as reentrancy, access control issues, and logic flaws. Additionally, thoroughly test the functions and roles defined in these contracts to make sure they behave as expected.

## System Overview

### Scope

1. **Contracts**

- **SemiFungiblePositionManager.sol**: This contract manages tokenized representations of multi-leg liquidity positions on Uniswap V3 pools according to the ERC1155 standard, by initializing pools, minting/burning tokens representing positions, interacting with underlying Uniswap pools to add/remove appropriate liquidity chunks and collect fees when positions are created/removed, tracking detailed per-position liquidity, fee accumulation and pool context state in mappings, and implementing callbacks to Uniswap for depositing, withdrawing and collecting proceeds of represented liquidity positions on behalf of token holders.

  ![SemiFungiblePositionManager.sol](https://private-user-images.githubusercontent.com/70327041/287521653-98185276-f7fd-4ddb-a1d6-c73727d99153.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTEiLCJleHAiOjE3MDE2MTEwMjgsIm5iZiI6MTcwMTYxMDcyOCwicGF0aCI6Ii83MDMyNzA0MS8yODc1MjE2NTMtOTgxODUyNzYtZjdmZC00ZGRiLWExZDYtYzczNzI3ZDk5MTUzLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFJV05KWUFYNENTVkVINTNBJTJGMjAyMzEyMDMlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjMxMjAzVDEzMzg0OFomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTllYmE1OGNmMGIwNDMzNTIyYmY0OWJkZDhiMjMyNWJhZWZmZTYzMWUyYTc3MmMwYjc1OWFjZTUyMzk5NGIzN2EmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JmFjdG9yX2lkPTAma2V5X2lkPTAmcmVwb19pZD0wIn0.FuIjOoNtw9hO8e6haNorzilf4apf-vEWftsEuv69nKw)

- **ERC1155Minimal.sol**: This contract is a basic ERC1155 implementation allowing the creation, transfer, and querying of multiple token types. It includes events for single and batch transfers, approval management, and internal functions for minting and burning tokens. The contract supports callback functions in recipients and follows the ERC1155 standard. It acts as a foundation for creating more complex contracts that extend or customize its functionality.

  ![ERC1155Minimal.sol](https://private-user-images.githubusercontent.com/70327041/287523765-ded74e58-d8b3-4d26-bd36-1352bb0bc15c.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTEiLCJleHAiOjE3MDE2MTI5NzMsIm5iZiI6MTcwMTYxMjY3MywicGF0aCI6Ii83MDMyNzA0MS8yODc1MjM3NjUtZGVkNzRlNTgtZDhiMy00ZDI2LWJkMzYtMTM1MmJiMGJjMTVjLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFJV05KWUFYNENTVkVINTNBJTJGMjAyMzEyMDMlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjMxMjAzVDE0MTExM1omWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPThlN2RmZjAyMmJmZjQ5M2M2ZDI1Zjc4ZDEzNDJjNDFjMGI0NGYyODgwODEzYzVhMzdmYmEwYmU0MTRmYThkZWMmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JmFjdG9yX2lkPTAma2V5X2lkPTAmcmVwb19pZD0wIn0.h5ciLoi4OCGOgrYPeku1EQQ6Piya1q8uXSdf5PweGOo)

- **Multicall.sol**: Enables the execution of multiple function calls in a single transaction. It is designed for batch operations, such as executing complex sequences of functions or emergency actions in a single atomic transaction. The contract includes a multicall function that takes an array of calldata for each call, executes them sequentially using delegatecall, and returns the results. If any of the calls fail, it reverts the entire transaction, propagating the revert reason. The contract is useful for optimizing gas usage when performing multiple operations together.

2. **Libraries**

- **LeftRight.sol**: This library facilitates the packing and manipulation of two separate 128-bit data chunks into a single 256-bit slot. It includes methods for extracting, writing, and performing arithmetic operations on the left and right halves of a 256-bit word.

  - Data Packing:leftSlot and rightSlot functions extract the left and right 128-bit chunks from a given 256-bit word.
  - Writing to Slots:toLeftSlot and toRightSlot functions write data into the left and right halves of a 256-bit word, respectively.
  - Math Helpers:
    - add function performs addition of two 256-bit LeftRight-encoded words, reverting on overflow or underflow.
    - sub function performs subtraction of two 256-bit LeftRight-encoded words, reverting on overflow.
    - toInt128 converts an int256 to int128, reverting on overflow or underflow.
    - toUint128 converts a uint256 to uint128, reverting on overflow.
    - toInt256 converts a uint256 to int256, reverting on overflow.

- **LiquidityChunk.sol**: This library is designed to manage and represent information about a liquidity chunk in a concentrated liquidity automated market maker (AMM). The library enables the creation, encoding, and decoding of a LiquidityChunk, which consists of the lower and upper ticks defining the tick range and the amount of liquidity within that range.

  - LiquidityChunk Structure: A LiquidityChunk is represented as a packed uint256, combining information about liquidity, lower tick, and upper tick.

  - Packing Rules:

    - Liquidity (128 bits): Represents the amount of liquidity within the chunk.
    - Tick Upper (24 bits): Represents the upper tick of the chunk.
    - Tick Lower (24 bits): Represents the lower tick of the chunk.

  - Encoding Functions:createChunk: Creates a new LiquidityChunk by adding liquidity, lower tick, and upper tick to an existing uint256.

    - addLiquidity: Adds liquidity to the existing LiquidityChunk.
    - addTickLower: Adds the lower tick to the existing LiquidityChunk.
    - addTickUpper: Adds the upper tick to the existing LiquidityChunk.

  - Decoding Functions:
    - tickLower: Retrieves the lower tick from a LiquidityChunk.
    - tickUpper: Retrieves the upper tick from a LiquidityChunk.
    - liquidity: Retrieves the amount of liquidity from a LiquidityChunk.

- **TokenId.sol**: This library is for working with option position IDs. The contract provides functions for encoding and decoding information related to option positions into/from a uint256, which is used as a token ID.

  - Decoding Functions:

    - univ3pool: Extracts the Uniswap v3 Pool ID from the option position ID.
    - asset: Gets the asset basis for a specific leg in the position (token0 or token1).
    - optionRatio: Retrieves the number of contracts per leg.
    - isLong: Checks whether a specific leg is a long position.
    - tokenType: Gets the type of token moved for a given leg (token0 or token1).
    - riskPartner: Retrieves the associated risk partner of a leg.
    - strike: Gets the strike price tick of a specific leg.
    - width: Gets the width of a specific leg.

  - Encoding Functions:

    - addUniv3pool: Adds the Uniswap v3 Pool ID to the option position ID.
    - addAsset: Adds the asset basis for a specific leg in the position.
    - addOptionRatio: Adds the number of contracts per leg.
    - addIsLong: Adds the "isLong" parameter indicating whether a leg is long.
    - addTokenType: Adds the type of token moved for a given leg.
    - addRiskPartner: Adds the associated risk partner of a leg.
    - addStrike: Adds the strike price tick of a specific leg.
    - addWidth: Adds the width of a specific leg.
    - addLeg: Adds a complete leg to the option position.

  - Helpers:

    - flipToBurnToken: Flips the "isLong" positions in the legs of the option position, useful during burning an option position.
    - countLongs: Counts the number of long positions in the option position.
    - asTicks: Gets the lower and upper ticks of a leg in the option position, considering tick spacing.

  - Validation:
    - validate: Validates an option position and its active legs, returning the underlying Uniswap V3 Pool address.

- **CallbackLib.sol**: Is a library for verifying and decoding Uniswap callbacks with the help of validateCallback function that checks if a callback comes from the claimed Uniswap pool by computing the deployed pool address and comparing it with the provided sender address.

- **FeesCalc.sol**: The FeesCalc library is designed to handle fee calculations related to liquidity chunks in Uniswap V3 option positions. It contains functions to compute fees collected from AMM, swap/trading activities within the given liquidity chunk. The calculations take into account the current AMM tick, the starting liquidity of the option position leg, and the specific liquidity chunk's tick range.

- **Math.sol**: Provides various math helper functions for handling calculations related to Uniswap V3 tick values, liquidity amounts, and other general mathematical operations.

  - Absolute Value Calculation: The absUint function calculates the absolute value of a signed integer (int256) and returns it as an unsigned integer (uint256).

  - Tick Math: The getSqrtRatioAtTick function computes the square root of a constant (1.0001) raised to the power of the tick divided by 2. The result is represented as a Q64.96 fixed-point number.

  - Liquidity Amounts (Strike + Width): The library provides functions (getAmount0ForLiquidity, getAmount1ForLiquidity, getLiquidityForAmount0, and getLiquidityForAmount1) to calculate token amounts and liquidity based on Uniswap V3 liquidity chunks. These functions consider the tick range and liquidity within the chunk.

  - Casting: The toUint128 function is used to safely downcast a uint256 to a uint128. It throws an error if overflow or underflow occurs during the downcast.

  - MulDiv Algorithms: The library includes several mulDiv functions that perform fixed-point multiplication and division with full precision. These functions use various bit-shifting techniques to optimize gas costs. They are designed to handle different precisions (64, 96, 128, and 192 bits).

- **PanopticMath.sol**: The PanopticMath library facilitates options management in Uniswap v3 pools by providing functions for calculating pool IDs, liquidity chunks, and token conversions, utilizing external libraries for mathematical operations and custom types for structured data representation.

### Protocol roles

The system defines different roles, including Traders, Liquidity Providers, and Liquidators.

1.  Traders: The primary role is to manage and control access to specific functions within the Core and Minter contracts. Admins can be classified into global admins, function admins, and collection admins.

- Selling an Option
  1. Seller posts collateral
  2. Seller uses Panoptic to deploy borrowed funds as a concentrated liquidity provider position into Uniswap
  3. Seller collects Uniswap LP fee rewards + an additional 'spread' from buyers as 'streamia' (streaming premia).
- Buying an Option
  1. Buyer posts collateral
  2. Buyer uses Panoptic to borrow a seller’s LP token from Uniswap and convert it to one of the underlying tokens
  3. Buyer continuously pays streamia to options sellers.

2. Liquidity Providers: A Panoptic Liquidity Provider (PLP or Panoptic LP) passively provides fungible liquidity of any token in any amount to a Panoptic pool to be borrowed by options traders. PLPs earn yield proportional to the volume of options traded in Panoptic.

3. Liquidators: A liquidator ensures the health of the protocol by closing accounts whose collateral balance falls below the margin requirements. Liquidators are rewarded with a bonus for minimizing the protocol's risk of insolvency.

## Approach Taken-in Evaluating The Panoptic Protocol

Accordingly, I analyzed and audited the subject in the following steps;

1.  **Core Protocol Contract Overview**:

    The SemiFungiblePositionManager (SFPM) contract, serving as the primary contract managing Uniswap V3 positions tokenized as ERC1155 tokens, enables the management of complex multi-leg positions and serves as an enhanced alternative to Uniswap's NFPM. It supports both typical LP positions, involving the addition of liquidity to Uniswap pools, and "long" positions, where liquidity is burned from Uniswap. The SFPM efficiently encodes all position data, including pool and legs, in ERC1155 tokenIds. Interacting with Uniswap pools, it facilitates the addition/removal of liquidity and the collection of fees necessary for minting/burning positions, while maintaining ownership, balances, and accumulated fees for positions. Supporting contracts define position data types, custom errors, math utilities, etc., to aid the SFPM, which aims to comply with ERC1155 for tokenized positions. Key invariants include that users can only remove liquidity they previously added, and fees collected do not exceed actual earned fees for a user's liquidity. The SFPM also accommodates any ERC20 tokens, including ERC-777, in Uniswap V3 pools.

    I focused on thoroughly understanding the codebase and providing recommendations to improve its functionality.
    The main goal was to take a close look at the important contracts and how they work together in the Panoptic Protocol.
    I started with the SemiFungiblePositionManager contract that is at the core of the system and from which other contracts interact or depend on. In this case.

    I start with the following contracts, which play crucial roles in the Panoptic protocol system:

    **Main Contracts I Looked At**

                SemiFungiblePositionManager.sol
                ERC1155Minimal.sol
                TokenId.sol
                FeesCalc.sol

    I started my analysis by examining the SemiFungiblePositionManager.sol contract, which serves as the main contract managing Uniswap V3 positions tokenized as ERC1155 tokens. The contract includes various functionalities, such as handling the initialization of Uniswap pools, minting and burning tokenized positions, and interacting with Uniswap pools to add/remove liquidity and collect fees. It incorporates features like reentrancy protection, fee calculations, and compliance with ERC1155 for tokenized positions. The contract also defines supporting structures and libraries to aid in its operations.

    Then, I turned my attention to the ERC1155Minimal.sol because it appears to be a minimalist implementation of the ERC1155 standard without metadata. This contract serves as an abstract foundation, providing essential functions and events for the management of fungible and non-fungible tokens. The contract includes necessary features such as single and batch transfers, approval management, and internal functions for minting and burning tokens. The code is structured with OpenZeppelin libraries and inherits from the abstract ERC1155 contract, offering a flexible and customizable base for specific token implementations.

    Then dive into the TokenId.sol contract because it serves as a critical component in Panoptic's ERC1155 representation of option positions within the Single-Sided Farming and Position Management (SFPM) system. This contract defines the structure of token IDs, detailing the packing rules for information related to each leg of an options position. The TokenId library provides methods for encoding, decoding, and validating these token IDs, making it essential for the proper functioning and manipulation of option positions in the Panoptic protocol. Understanding this contract is crucial for grasping how option position details are efficiently stored and retrieved within the Panoptic ecosystem.

    Then audit FeesCalc contracts, for calculating fees related to option positions and their associated liquidity chunks on the AMM/Uniswap. The library includes functions to compute AMM swap fees for a given liquidity chunk within an option position, considering factors such as the current price tick, starting liquidity, and the Uniswap pool. It also provides a method to determine the fee growth in the AMM for a specific option position's liquidity chunk within its tick range. The implementation utilizes custom types and libraries like LeftRight, LiquidityChunk, and TokenId for efficient data representation and manipulation. Accurate fee calculations within defined liquidity chunks are essential for the proper functioning of the Panoptic protocol, particularly in the context of Single-Sided Farming and Position Management (SFPM) for options.

2.  **Documentation Review**:

    Then went to Review [This Docs](https://panoptic.xyz/docs/intro) for a more detailed and technical explanation of Panoptic protocol.

3.  **Compiling code and running provided tests**:

4.  **Manuel Code Review** In this phase, I initially conducted a line-by-line analysis, following that, I engaged in a comparison mode.

    - **Line by Line Analysis**: Pay close attention to the contract's intended functionality and compare it with its actual behavior on a line-by-line basis.

    - **Comparison Mode**: Compare the implementation of each function with established standards or existing implementations, focusing on the function names to identify any deviations.

## Architecture Description and Diagram

Architecture of the contracts that are part of the Panoptic protocol:

![Diagram](https://private-user-images.githubusercontent.com/70327041/287639954-03a65dda-5fe5-48c2-86c0-463e0240acaa.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTEiLCJleHAiOjE3MDE2ODAyMTUsIm5iZiI6MTcwMTY3OTkxNSwicGF0aCI6Ii83MDMyNzA0MS8yODc2Mzk5NTQtMDNhNjVkZGEtNWZlNS00OGMyLTg2YzAtNDYzZTAyNDBhY2FhLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFJV05KWUFYNENTVkVINTNBJTJGMjAyMzEyMDQlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjMxMjA0VDA4NTE1NVomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWZmZGEwZTkxM2NmNWZkZDVlYTMwOWU4NTBlODlkMDcyZWRlNDRlYWYyZjgwYzUwZjBlOGFmNDNiMjU0Njk4ODkmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JmFjdG9yX2lkPTAma2V5X2lkPTAmcmVwb19pZD0wIn0.cJUDw2OYaGJn9DE0F29vGfm4vBNdH84P37aA-f8GA9Y)

The UniswapV3Pool.sol interface facilitates swapping and automated market making between ERC20 assets, forming the foundation for Panoptic's deployed contracts, such as SemiFungiblePositionManager.sol, which, as a replacement for Uniswap's Nonfungible Position Manager, manages up to 4-legged Uniswap V3 positions in ERC1155 non-fungible tokens. Additionally, PanopticFactory.sol deploys options markets on existing Uniswap V3 pools, and PanopticPool.sol creates and manages undercollateralized options, handling positions, collateral, liquidations, and forced exercises. The CollateralTracker.sol monitors and manages collateral using a shares model, while the NFPMigrator.sol in the periphery assists in migrating positions from the Nonfungible Position Manager in Uniswap v3-periphery.

Overall, the Panoptic architecture is well designed and thought out, seamlessly integrating with Uniswap's infrastructure to enable efficient and comprehensive management of options markets, collateral, and liquidity positions.

**SemiFungiblePositionManager.sol**

![Diagram](https://private-user-images.githubusercontent.com/70327041/287641157-8677f517-538a-469a-b184-ec094cee31fa.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTEiLCJleHAiOjE3MDE2ODAyODAsIm5iZiI6MTcwMTY3OTk4MCwicGF0aCI6Ii83MDMyNzA0MS8yODc2NDExNTctODY3N2Y1MTctNTM4YS00NjlhLWIxODQtZWMwOTRjZWUzMWZhLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFJV05KWUFYNENTVkVINTNBJTJGMjAyMzEyMDQlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjMxMjA0VDA4NTMwMFomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTUwNjU4NzYzMmY5MzgxYjJmMzgyYzdmZDFhZWZkMzY4MDk2N2UzZTFkNzE1Y2UzMGU1YWUwMzEyMTc4MWY5YjQmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JmFjdG9yX2lkPTAma2V5X2lkPTAmcmVwb19pZD0wIn0.dGQxzBpt3wZPuQ6Mow-zqi9rSxI--nETz0Y8McyPXfg)

The SemiFungiblePositionManager for Panoptic efficiently manages liquidity provider (LP) positions using the ERC1155 interface, allowing for the creation, burning, and rolling of up to 4-legged Uniswap V3 positions wrapped in an ERC1155 non-fungible token, with detailed methods for initializing pools, minting and burning tokenized positions, and viewing account fees and liquidity, all seamlessly integrated into Panoptic's option market infrastructure.

### Some potential areas for improvement and Architecture feedback

1. In SemiFungiblePositionManager.sol contract the code interacts with Uniswap V3 in several places. Consider creating a separate contract or library to abstract the interaction with Uniswap, encapsulating the logic related to minting, burning, and swapping.

2. Formal Verification: Explore formal verification tools and techniques to prove the correctness of the contracts using a tool like Certora.

3. The Protocol doesn't include explicit mechanisms for upgradability. If upgrades are necessary, it might require a new deployment, which could be seen as a lack of flexibility.

## Codebase Quality

Overall, I consider the quality of the Panoptic protocol codebase to be Good. The code appears to be mature and well-developed. We have noticed the implementation of various standards adhere to appropriately. Details are explained below:

| Codebase Quality Categories              | Comments                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| ---------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Code Maintainability and Reliability** | The Panoptic Protocol contracts demonstrates good maintainability through modular structure, consistent naming, and efficient use of libraries. It also prioritizes reliability by handling errors, conducting security checks. However, for enhanced reliability, consider professional security audits and documentation improvements. Regular updates are crucial in the evolving space.                                                                                                                                                                                                                                                                                            |
| **Code Comments**                        | During the audit of the Panoptic contracts codebase, I found that some areas lacked sufficient internal documentation to enable independent comprehension. While comments provided high-level context for certain constructs, lower-level logic flows and variables were not fully explained within the code itself. Following best practices like NatSpec could strengthen self-documentation. As an auditor without additional context, it was challenging to analyze code sections without external reference docs. To further understand implementation intent for those complex parts, referencing supplemental documentation was necessary.                                      |
| **Documentation**                        | The documentation of the Panoptic project is quite comprehensive and detailed, providing a solid overview of how Panoptic protocol is structured and how its various aspects function. However, we have noticed that there is room for additional details, such as diagrams, to gain a deeper understanding of how different contracts interact and the functions they implement. With considerable enthusiasm. We are confident that these diagrams will bring significant value to the protocol as they can be seamlessly integrated into the existing documentation, enriching it and providing a more comprehensive and detailed understanding for users, developers and auditors. |
| **Testing**                              | The audit scope of the contracts to be audited is 100% this is Excellent.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| **Code Structure and Formatting**        | The codebase contracts are well-structured and formatted. It inherits from multiple components, adhering for-example The SemiFungiblePositionManager contract follows a modular structure with clear separation of concerns into Core contract logic, inherited IUniswapV3Pool standards, and imported supporting contracts.Functions are grouped thematically and formatted uniformly with visibility, modifiers, and error handling.efficiency.                                                                                                                                                                                                                                      |
| **Strengths**                            | Comprehensive unit tests,Utilization of Natspec                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |

## Systemic & Centralization Risks

The analysis provided highlights several significant systemic and centralization risks present in the Panoptic protocol. These risks encompass concentration risk in SemiFungiblePositionManager, ERC1155Minimal risk and more, third-party dependency risk, and centralization risks arising. Additionally, the absence of fuzzing and invariant tests could also pose risks to the protocol’s security.

Here's an analysis of potential systemic and centralization risks in the contracts:

### Systemic Risks:

1. **No having, fuzzing and invariant tests could open the door to future vulnerabilities**.

2. The SemiFungiblePositionManager.sol contract performs operations based on specified tick limits (tickLimitLow and tickLimitHigh). If these limits are not set appropriately or if the price oracle is manipulated, it can lead to systemic risks such as excessive slippage or inaccurate pricing.

3. The swapping mechanism for ITM Swapping options introduces systemic risks in SemiFungiblePositionManager contract. If the swapping logic is not precise or if there are vulnerabilities in the netting swap calculations, it may lead to unexpected behavior and financial risks.

4. The SemiFungiblePositionManager updates liquidity and fee values between accounts based on token transfers. If there are errors or vulnerabilities in the update process, it can result in systemic risks related to incorrect accounting and financial losses.

5. Emergency Controls: There are no explicit emergency controls or pausing mechanisms. The absence of such features might be a risk in case of unforeseen issues or attacks.

6. The ERC1155Minimal doesn't include explicit protections against front-running, where malicious actors could manipulate transaction ordering.

7. The LiquidityChunk.sol contract assumes that the upper bits (80 bits) of the uint256 are zero. This assumption is critical for the correct functioning of packing and unpacking operations. Any deviation from this assumption could lead to unexpected behavior.

8. Fee Calculation Accuracy: The accuracy of fee calculations depends on the correctness of the Uniswap pool's fee growth values and the provided ticks. Any discrepancies in fee growth or tick values could lead to inaccurate fee calculations.

### Centralization Risks:

1. Single Uniswap Pool Dependency: The contract is tightly coupled to a specific Uniswap V3 pool (IUniswapV3Pool). Any changes or issues with this pool can directly impact the contract's functionality.

**Properly managing these risks and implementing best practices in security and decentralization will contribute to the sustainability and long-term success of the Panoptic protocol.**

## Conclusion

In general, The Panoptic protocol presents a well-designed architecture for managing generative NFT collections with its set of core contracts, we believe the team has done a good job regarding the code. However, the analysis revealed some security and decentralization risks that warrant attention. Chief among these is over-reliance on centralized admin accounts for critical functions. Additionally, it is recommended to improve the documentation and comments in the code to enhance
understanding and collaboration among developers and auditors. It is also highly recommended that the team continues to invest in security measures such as mitigation reviews, audits, and bug bounty programs to maintain the security and reliability of the project.


### Time spent:
24 hours