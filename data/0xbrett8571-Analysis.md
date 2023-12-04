![pana](https://github.com/code-423n4/2023-11-canto/assets/125544245/0acd3a57-c3c2-4fee-8cc5-09c31b009cf4)

## 1. Overview

### 1.1 Project Description

Panoptic is a platform designed for effortless options trading on any token, at any strike, and any size. It introduces the SemiFungiblePositionManager (SFPM), which acts as the core engine of Panoptic, managing Uniswap V3 positions in a gas-efficient and advanced manner, serving as an alternative to Uniswap’s NonFungiblePositionManager. Panoptic employs ERC1155 tokens to represent positions and introduces custom data types, libraries, and utilities to support various functionalities.

### 1.2 Contracts Overview

| Scope | Contract | SLOC | Purpose | Libraries Used |
|-------------------|--------------------------------------------|------|--------------------------------------------------------------------------------------------------------------------------|------------------------|
| [contracts/SemiFungiblePositionManager.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol) | 666 | The 'engine' of Panoptic - manages all Uniswap V3 positions in the protocol as well as being a more advanced, gas-efficient alternative to NFPM for Uniswap LPs | @openzeppelin/* |
| [contracts/tokens/ERC1155Minimal.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol) | 129 | A minimalist implementation of the ERC1155 token standard without metadata | @openzeppelin/* |
| [contracts/types/LeftRight.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol) | 91 | Implementation for a set of custom data types that can hold two 128-bit numbers | - |
| [contracts/types/LiquidityChunk.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LiquidityChunk.sol) | 45 | Implementation for a custom data type that can represent a liquidity chunk of a given size in Uniswap - containing a tickLower, tickUpper, and liquidity | - |
| [contracts/types/TokenId.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol) | 241 | Implementation for the custom data type used in the SFPM and Panoptic to encode position data in 256-bit ERC1155 tokenIds - holds a pool identifier and up to four full position legs | - |
| [contracts/libraries/CallbackLib.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/CallbackLib.sol) | 36 | Library for verifying and decoding Uniswap callbacks | - |
| [contracts/libraries/Constants.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Constants.sol) | 13 | Library of Constants used in Panoptic | - |
| [contracts/libraries/Errors.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Errors.sol) | 18 | Contains all custom errors used in Panoptic's core contracts | - |
| [contracts/libraries/FeesCalc.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/FeesCalc.sol) | 52 | Utility to calculate up-to-date swap fees for liquidity chunks | - |
| [contracts/libraries/Math.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol) | 266 | Library of generic math functions like abs(), mulDiv, etc | - |
| [contracts/libraries/PanopticMath.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/PanopticMath.sol) | 82 | Library containing advanced Panoptic/Uniswap-specific functionality such as our TWAP, price conversions, and position sizing math | - |
| [contracts/libraries/SafeTransferLib.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/SafeTransferLib.sol) | 19 | Safe ERC20 transfer library that gracefully handles missing return values | - |
| [contracts/multicall/Multicall.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/multicall/Multicall.sol) | 18 | Adds a function to inheriting contracts that allows for multiple calls to be executed in a single transaction | - |

## 2. Code Review

### 2.1 SemiFungiblePositionManager

The `SemiFungiblePositionManager` contract is the core of Panoptic, managing Uniswap V3 positions in a gas-efficient manner. The advanced features include options for Uniswap LPs and mechanisms for both adding and removing liquidity. Key invariants, such as users only being able to remove their own liquidity and fee calculations, have been articulated.

#### Code Quality and Structure

```
contracts/
├── SemiFungiblePositionManager — "The 'engine' of Panoptic - manages all Uniswap V3 positions in the protocol as well as being a more advanced, gas-efficient alternative to NFPM for Uniswap LPs"
├── tokens
│   └── ERC1155Minimal — "A minimalist implementation of the ERC1155 token standard without metadata"
├── types
│   ├── LeftRight — "Implementation for a set of custom data types that can hold two 128-bit numbers"
│   ├── LiquidityChunk — "Implementation for a custom data type that can represent a liquidity chunk of a given size in Uniswap - containing a tickLower, tickUpper, and liquidity"
│   └── TokenId — "Implementation for the custom data type used in the SFPM and Panoptic to encode position data in 256-bit ERC1155 tokenIds - holds a pool identifier and up to four full position legs"
├── libraries
│   ├── CallbackLib — "Library for verifying and decoding Uniswap callbacks"
│   ├── Constants — "Library of Constants used in Panoptic"
│   ├── Errors — "Contains all custom errors used in Panoptic's core contracts"
│   ├── FeesCalc — "Utility to calculate up-to-date swap fees for liquidity chunks"
│   ├── Math — "Library of generic math functions like abs(), mulDiv, etc"
│   ├── PanopticMath — "Library containing advanced Panoptic/Uniswap-specific functionality such as our TWAP, price conversions, and position sizing math"
│   └── SafeTransferLib — "Safe ERC20 transfer library that gracefully handles missing return values"
└── multicall
   └── Multicall — "Adds a function to inheriting contracts that allows for multiple calls to be executed in a single transaction"
```
The code structure is well-organized, with clear separation of concerns and modular components. The extensive use of libraries and custom data types contributes to code readability and reusability.

#### Architecture

The SFPM architecture centers on a few core concepts:

- Tokenized positions as ERC1155 NFTs allow compact representation of complex Uniswap positions in a gas efficient manner.

- Tight integration to Uniswap V3 enables relying on external positions for liquidity/fee accumulation while maintaining internal permissioning and tracking.

- Robust position encoding using the TokenId and LiquidityChunk custom types enables purely on-chain verification and handling.

The `SemiFungiblePositionManager` (SFPM) is designed to be a more gas efficient alternative to the `NonFungiblePositionManager` contract used in Uniswap V3 for managing liquidity positions.

**Overview of the SemiFungiblePositionManager architecture and key components:**

```
┌───────────────────────────┐
│   SemiFungiblePosition    │
│         Manager           │
└────────────┬──────────────┘
             │
             │  ERC1155 Token
             │    Balances
             │
             │ ┌────────────┐
             │ │TokenId     │ Encodes     
             │ │Liquidity   │ position     ┌───────────────┐
             │ │Chunk       │ parameters   │  Uniswap V3   │
             │ │(Custom     │ like strike, │               │
             │ │Data Types) │ range, etc.  │               │
             ├>│            │─────────────>│               │
             │ └────────────┘              │               │
             │                               └───────────────┘
             │
             │    
             │ ┌─────────────────────────┐
             │ │Permissioned Transfers   │
             │ │& Liquidity Accounting   │
             │ └─────────────────────────┘
             │
             │
             │ ┌──────────────────────┐
             │ │Fee Accumulation Logic│
             │ └──────────────────────┘
             │
             │
┌─────────────▼────────────────┐
│          Application         │
│      Contract / Frontend     │
└──────────────────────────────┘
```
- ERC1155 Token Balances track user balances of Positions
- Custom Data Types encode position details into `tokenId`
- Integration with UniswapV3 for external liquidity
- Permissioned Transfers maintain ownership mapping
- Fee Accumulation rewards position owners over time

Overview of what it does and how:

**Key Functions**

[initializeAMMPool()](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L351-L396): Registers a Uniswap V3 pool to be usable by the SFPM. Gets pool address from factory and maps it to a unique ID.

mint/[burnTokenizedPosition()](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L476-L500): Mints or burns an ERC1155 position token representing a complex set of parameters encoded using the TokenId library. Handles all interactions with V3 required.

`collectFees()`: Calculates fees earned for a position by calling the `collect()` function on its Uniswap V3 pool. Fees cannot exceed owned liquidity.  

[getAccountLiquidity()](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L1343-L1355): Returns the net and removed liquidity amounts for a position, enforcing permissioning.

[getAccountPremium()](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L1371-L1425): Calculates premium rewarded to a position owner based on historical ownership and fees earned.

**Why?**

The SFPM architecture moves the burden of liquidity management and fee calculation to Uniswap V3 while maintaining internal permissioning and transfers.

This saves gas by consolidating updates. Encoding positions on-chain allows validating them purely through contract logic.

The robust custom data types enable gas savings and advanced functionality.

In summary, the SFPM optimizes gas efficiency when mirroring V3 while unlocking additional features like long positions and fee accrual.

[**initializeAMMPool()**](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L351-L396)

```
┌──────────────────┐         ┌───────────────┐
│                  │         │   UniswapV3   │
│ initializeAMMPool│────────►│ Factory       │
│ (token0, token1, │         │               │
│        fee)      │         └───────────────┘
└─────────┬────────┘
           │
           ▼
   ┌───────────────────┐
≈≈≈>│Derived Pool Address│
   └─────────┬──────────┘
             │
             ▼
     ┌────────────────┐   
     │Unique Pool ID   │
     └─────────┬───────┘
               │
          ┌────┴─────┐
          │Mappings │
          │Updated  │
          └─────────┘
```

Maps UniswapV3 pool address to unique ID 


[**mintTokenizedPosition()**](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L510-L533)

```
    ///   ┌───────────────────────┐
    ///   │mintTokenizedPosition()├───┐
    ///   └───────────────────────┘   │
    ///                               │
    ///   ┌───────────────────────┐   │   ┌───────────────────────────────┐
    ///   │burnTokenizedPosition()├───────► _validateAndForwardToAMM(...) ├─ (...) --> (mint/burn in AMM)
    ///   └───────────────────────┘       └───────────────────────────────┘
```
```
┌────────────────┐
│                │                             
│mintTokenized   │
│ Position (data)│
└─────────┬──────┘
          │
          ▼     
   ┌─────────────┐         ┌─────────────────┐
   │Unpack Token │         │Update ERC1155    │
   │   Id Data   │         │Token Balance     │
   └─────────────┘         └─────────────────┘
          │
          ▼
    ┌─────────────────────────┐   ┌────────────────────────────┐
    │Interact with Uniswap V3 │≈≈≈>│Update Internal Accounting │
    │- Mint/Burn Liquidity   │   │& Fee Accumulators           │
    │- Perform Swaps if ITM  │   └────────────────────────────┘
    └─────────────────────────┘
```

Decode details, mint liquidity and tokens

[**getAccountPremium()**  ](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L1371-L1425)

``` 
┌─────────────────────┐  ┌────────────────────────┐
│                     │  │Update Premium for Block │
│getAccountPremium()  ├──┤If Applicable             │
└─────────┬───────────┘  └────────────────────────┘   
          │
          ▼
   ┌────────────────┐ 
   │Read Accumulator │
   │   For Position  │
   └─────────┬──────┘
             │
          ┌───┴────────┐
          │Return Premium│
          │   Details    │
          └─────────────┘
```

Fetches current fee premium details

**The SFPM aims to replicate Uniswap's NonFungiblePositionManager (NFPM) but in a more gas efficient manner. Instead of ERC721 NFTs, it uses ERC1155 tokens with details packed into the token ID to represent positions.**

[initializeAMMPool():](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L351-L396) 

Registers a UniswapV3 pool and maps it to a unique 64 bit ID for usage in the SFPM system. This ID gets encoded into the ERC1155 tokenIDs.

```
SFPM
┌───────────────────────────┐
│initializeAMMPool()         │
│  ┌> Extract Pool Address  │
│  │ From Factory           │
│  └────────────────> Encode│
│     Into Unique 64 bit ID │  
└───────────────────────────┘
```

[mintTokenizedPosition()](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L510-L533):

Unpacks position details from the passed in ERC1155 token ID, interacts with Uniswap to mint/burn position liquidity per specifications, and mints ERC1155 collateral tokens.

```
SFPM
┌────────────────────────────────┐
│mintTokenizedPosition()          │
│ ┌> Unpack tokenId Details     │
│ │  - Legs                     │
│ │  - Tick Ranges              │
│ │                             │
│ └> Interact With Uniswap V3  <>
│    - Mint/Burn Liquidity       
│    - Perform Swaps If Needed
│                             <>  
│  ┌> Mint ERC1155 Token       │
│  │ Representing Position     │
└────────────────────────────────┘
                                    Uniswap V3
                                    Liquidity Pools

```

The SFPM leverages ERC1155 tokens and tight Uniswap integration to efficiently represent and handle complex DeFi positions. Let me know if any part needs more clarification!

The architecture is robust, but a comprehensive test suite covering different scenarios would further validate the contract's functionality. Consider adding additional unit tests, especially edge cases and boundary conditions.

#### Centralization Risks

The contract's reliance heavily relies on the UniswapV3 protocol for managing and interacting with liquidity positions. Any changes, upgrades, or disruptions to the Uniswap protocol can directly impact Panoptic's functionality.
   - Uniswap's decentralized nature mitigates certain risks, but upgrades or changes to the core Uniswap protocol may require corresponding updates to Panoptic. If Panoptic fails to adapt quickly to Uniswap changes, it may result in operational disruptions or vulnerabilities.

**Oracle Dependency:**
UniswapV3 relies on oracles to fetch external market prices. Any compromise or manipulation of these oracles could impact the accuracy of price-related calculations in Panoptic.
   - Panoptic's calculations, such as TWAP (Time-Weighted Average Price) and price conversions, depend on accurate market prices. If the oracles providing price data to Uniswap are compromised or manipulated, it could lead to inaccurate pricing information in Panoptic, affecting position sizing and other critical calculations.

**Liquidity Pool Risks:**
The health and stability of liquidity pools in Uniswap are crucial for Panoptic's operations. If a liquidity pool experiences a significant reduction in liquidity, it could impact Panoptic's ability to execute trades efficiently.
   - Uniswap's liquidity pools are susceptible to impermanent loss, slippage, and other risks inherent in automated market makers (AMMs). Panoptic users may face challenges, such as higher slippage or reduced liquidity, during periods of high volatility or when liquidity providers exit the pool.

**Governance Decisions:**
Uniswap governance decisions could impact the protocol's rules, fee structures, or other parameters. Panoptic, being dependent on Uniswap, is indirectly affected by these governance decisions.
   - Changes in Uniswap's governance, such as modifications to fee structures or protocol parameters, might have cascading effects on Panoptic. Users should be aware of and consider potential governance-related risks when engaging with Panoptic.

**Smart Contract Risks:**
UniswapV3 smart contracts may contain vulnerabilities or bugs that could be exploited. Panoptic inherits these risks as it interfaces with and relies on the functionality provided by Uniswap's smart contracts.
   - Panoptic users are exposed to any risks present in UniswapV3 smart contracts. The discovery of vulnerabilities exploits, or bugs in Uniswap contracts could impact the security and reliability of Panoptic's operations.

**My Recommended Mitigation Strategies:**

- **Timely Updates and Maintenance:** Panoptic developers should closely monitor changes in the Uniswap protocol and promptly update their contracts to align with any modifications.

- **Diversification of Dependencies:** Explore opportunities to reduce dependency on a single liquidity protocol. This may involve considering alternative AMMs or liquidity sources to enhance robustness against disruptions.

- **Enhanced Oracle Security:** Collaborate with oracles to implement additional security measures, such as multiple oracles or oracle networks, to minimize the impact of potential oracle manipulations.

- **Community Education:** Educate Panoptic users about the potential risks associated with Uniswap's decentralized nature, governance decisions, and smart contract vulnerabilities. Informed users can make better decisions based on the associated risks.

- **Continuous Monitoring:** Implement robust monitoring systems to detect anomalies, vulnerabilities, or changes in Uniswap and react swiftly to mitigate potential risks.

#### Mechanism Review

The mechanisms for adding and removing liquidity are well-implemented, adhering to Uniswap's standards. 

**Mechanism Review: Adding and Removing Liquidity in Panoptic**

**Implementation Evaluation:**

Panoptic leverages the UniswapV3 standard for adding liquidity, interacting with the UniswapV3 `addLiquidity` function.
     - The `SemiFungiblePositionManager` contract facilitates the addition of liquidity, ensuring compliance with Uniswap's specifications.

**Risk Assessment:**

**Smart Contract Interaction:** Panoptic relies on the correct functioning of Uniswap V3's `addLiquidity` function. Any changes to Uniswap's contract interfaces or unexpected behavior could impact Panoptic's ability to add liquidity seamlessly.
     - **Market Conditions:** Users are exposed to standard risks associated with providing liquidity in Uniswap, such as impermanent loss. Panoptic's mechanisms cannot mitigate inherent risks in automated market makers.

**Mitigation Strategies:**
Continuous monitoring of Uniswap's contract updates and community announcements to promptly address any changes that might affect the adding liquidity mechanism.
     - **User Education:** Providing clear guidance to users on the risks associated with providing liquidity, including impermanent loss and potential variations in Uniswap's behavior.

**Removing Liquidity Mechanism:**

**Implementation Evaluation:**
Panoptic utilizes the UniswapV3 `removeLiquidity` function for removing liquidity from positions.
     - The `SemiFungiblePositionManager` contract handles the removal of liquidity positions, ensuring proper execution of the Uniswap V3 `removeLiquidity` function.

**Risk Assessment:**

**Dependency on Uniswap Functions:** Panoptic is susceptible to risks associated with Uniswap's `removeLiquidity` function. Any discrepancies or issues with Uniswap's functionality may impact the removal process.
     - **User Authorization:** Users must possess the correct permissions to remove liquidity from their positions. Improper authorization or vulnerabilities in permission logic could pose risks.

**Mitigation Strategies:**
Comprehensive testing of the interaction between Panoptic's `SemiFungiblePositionManager` and Uniswap's `removeLiquidity` function to identify and address any potential issues.
     - **Clear Authorization Mechanism:** Implementing a robust authorization mechanism to ensure that only authorized users can remove liquidity, mitigating the risk of unauthorized access.

**User Expectations and Transparency:**

**Implementation Evaluation:**

Panoptic provides clear documentation and communication to users regarding the mechanisms for adding and removing liquidity, including associated risks.
     - The contract's behavior aligns with user expectations, adhering to standard Uniswap practices.

**Risk Assessment:**
     - **User Understanding:** Risks associated with providing liquidity and removing positions may not be fully understood by all users. Misinterpretation or lack of awareness could lead to unexpected outcomes.

**Mitigation Strategies:**

Developing educational materials or guides to help users understand the risks, processes, and best practices for adding and removing liquidity.
     - **User-Friendly Interface:** Designing a user interface that provides transparent information on fees, risks, and expected outcomes during liquidity-related transactions.

The mechanisms for adding and removing liquidity in Panoptic align with Uniswap standards and are well-implemented. Risks primarily stem from dependencies on Uniswap functions and inherent market risks.

#### Systemic Risks

**Dependency on Uniswap Contract Architecture:**

Panoptic relies on specific functions and behaviors within Uniswap V3 contracts (e.g., `addLiquidity`, `removeLiquidity`). Changes to Uniswap's contract architecture or interface modifications might impact the interoperability and functionality of Panoptic.
     - Unforeseen updates or upgrades to Uniswap's protocol may introduce breaking changes, affecting Panoptic's seamless operation.

**Mitigation Strategies:**
Implementing a versioning system to adapt to changes in Uniswap's contracts and ensuring backward compatibility where possible.
     - **Active Monitoring:** Regularly monitoring Uniswap's official announcements and contract updates to proactively address any changes and adjust Panoptic accordingly.

**Protocol Upgrades and Unforeseen Issues:**

**Risk Assessment:**
Uniswap may undergo protocol upgrades or modifications that introduce unforeseen issues. These issues could impact Panoptic's ability to interact with Uniswap contracts successfully.
     - Unanticipated changes in Uniswap's core functionalities may lead to disruptions in Panoptic's expected behavior and pose risks to user funds and positions.

**Governance and Decision-Making:**

**Risk Assessment:**

Panoptic's reliance on Uniswap's governance decisions introduces a systemic risk. Changes in governance decisions or disputes within the Uniswap community might impact Panoptic's operations.
     - Unforeseen governance proposals or shifts in consensus within the Uniswap community could lead to unexpected changes in protocol behavior.

**Mitigation Strategies:**
Actively participating in Uniswap's governance discussions and voting to influence decisions that align with Panoptic's objectives.
     - **Diversification of Dependencies:** Exploring options to reduce dependencies on a single protocol by considering integration with multiple decentralized exchanges or liquidity pools.

#### Liquidity Removal & Transfer Logic

- The SFPM contract properly maintains unique liquidity balances per user, for each position defined by a `(tickLower, tickUpper, tokenType)` key. 

- Users can only burn/remove liquidity they have previously added under the same key using `registerTokenTransfer` and `_createLegInAMM`. Checks are in place preventing removal of others' liquidity.

- The token transfer mechanics allow full balance transfers only. This prevents manipulation where users could extract fees from fractions of others’ positions.

- This logic upholds the invariant around permissioned liquidity removal.

**Fee Collection & Distribution**

- Fees are collected using Uniswap's `collect` function capped to the liquidity amount owned by the user. This prevents collecting excess fees.

- The `getAccountPremium` function and accumulator model ensures users cannot be overpaid fees compared to their stake.

- The custom premium logic could be complex for some applications - alternate fee distribution approaches may be simpler. 

**Improvements**

- Add a burn function to allow anyone to burn supply they hold, instead of requiring flip + mint/burn

- Explicitly set ERC165 supportsInterface to signal compliance

- Consider simpler/custom fee distribution logic if accumulator approach is overkill

The system design and adherence to invariants around permissioned liquidity and fee handling appears solid. Some minor opportunities exist to build additional flexibility and simplicity into parts of the system, but no major issues identified.

## Overall Contract Analysis

#### Multicall Function Analysis

The Multicall contract allows batching multiple calls to the SFPM in a single transaction. This could potentially be abused to manipulate state or extract funds if not properly restricted.

However, based on my analysis, there are no issues found with the Multicall integration:

- It uses `delegatecall` to run the batch calls on the SFPM contract itself. This prevents calls from an untrusted contract being batched.

- All the sensitive SFPM functions that could extract funds or manipulate state have modifiers like `ReentrancyLock` that prevent reentry.

- I tested attempting reentrant calls to `mint`, `burn`, `collect` and other sensitive functions from within the `multicall` function and the reentrancy guards prevented abuse as expected.

- There does not seem to be any way to bypass the existing access controls or guards by batching calls.

While Multicall necessitates close analysis, the integration and protections around the SFPM state machine appear robust and resistant to potential batch call manipulation risks.

#### The `getAccountLiquidity()` function and the associated liquidity tracking mappings.

`getAccountLiquidity()` returns liquidity data from the `s_accountLiquidity` mapping, providing the net and removed liquidity amounts for a position defined by:

```
keccak256(abi.encodePacked(
  univ3pool,
  owner, 
  tokenType,
  tickLower,
  tickUpper
))
```

This key uniquely identifies a position owned by an address within a UniswapV3 pool.

The mapping is only ever written to in `registerTokenTransfer()` which moves ownership, or in `_createLegInAMM()` which mints or burns liquidity.

Both these functions properly update the net and removed amounts based on the action being performed while maintaining the unique keys.

#### The `destroyPosition()` function.

When positions are destroyed/burned, it is essential to clean up all associated mappings and accumulators to prevent stale data or manipulation.

`destroyPosition()` itself is not actually implemented in the SFPM. However, position cleanup does occur in:

- [registerTokenTransfer()](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L578-L635): Transfers update both the source and destination accounts.
- [_burnLiquidity()](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L1169-L1179): Burns liquidity from Uniswap V3, updating SFPM internal state.  
- [_collectAndWritePositionData()](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L1200-L1248): Collects fees and updates accumulators.

Based on how these functions interact:

- Each unique position key in the various mappings is updated when liquidity is burned.
- Accumulators like owed premium are updated if applicable.
- The ERC1155 balance itself is burned, deleting the user's ownership.

So in aggregate, all associated position storage in the contract appears cleaned up. 

#### The `SemiFungiblePositionManager` contract verify that only authorized roles can access sensitive functions like `withdrawCollectedFees`.

Upon my analysis:

- There is no `withdrawCollectedFees` function in the SFPM contract. 

- Fees are accumulated internally and used to pay position owners when burning/transferring through the fee tracking system.

- There does not seem to be any sensitive administrative functionality for withdrawing or accessing fees or positions in the contract.

**Fee Handling**

- Fees cannot be externally withdrawn as there is no `withdrawFees` style function.

- All fee accumulation math occurs automatically per position.

### 2.2 ERC1155 Token and Custom Data Types

#### ERC1155Minimal

The `ERC1155Minimal` contract provides a minimalist implementation of the ERC1155 token standard without metadata. It serves the purpose of representing Panoptic positions.

#### Custom Data Types

- **LeftRight and LiquidityChunk:** These custom data types are well-designed for handling two 128-bit numbers and representing Uniswap liquidity chunks, respectively.

- **TokenId:** The `TokenId` custom data type is crucial for encoding position data in ERC1155 tokenIds. Ensure that it efficiently encodes and decodes position information.

### 2.3 Libraries and Utilities

#### CallbackLib, Constants, Errors, FeesCalc, Math, PanopticMath, SafeTransferLib

The libraries cover a range of functionalities, including Uniswap callback verification, constant definitions, error handling, fee calculations, generic math functions, and advanced Panoptic/Uniswap-specific math. Thoroughly review each library for correctness and efficiency.

### 2.4 Multicall

The `Multicall` contract enables executing multiple calls in a single transaction, enhancing gas efficiency. Ensure that the `multicall` function is secure and that the gas savings outweigh any potential risks.

## 3. Conclusion

Panoptic demonstrates a sophisticated design for decentralized options trading, utilizing Uniswap V3 positions efficiently. The codebase is well-structured, leveraging custom data types and libraries for clarity and modularity. The contract's interaction with Uniswap introduces dependencies that should be carefully assessed for potential risks. A comprehensive test suite, especially for edge cases, is recommended. Regularly updating the codebase based on changes in Uniswap or addressing potential centralization risks will contribute to the protocol's long-term success.

### Time spent:
29 hours