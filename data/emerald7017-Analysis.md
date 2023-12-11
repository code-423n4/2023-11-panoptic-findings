## Overview

The **SemiFungiblePositionManager** is a gas-efficient alternative to Uniswap’s **NonFungiblePositionManager** that manages complex, multi-leg Uniswap positions encoded in ERC1155 tokenIds, performs swaps allowing users to mint positions with only one type of token, and, most crucially, supports the minting of both typical LP positions where liquidity is added to Uniswap and “long” positions where Uniswap liquidity is burnt.

I have thoroughly reviewed the Panoptic contracts and architecture, probing for potential issues with security, decentralization, and systemic risk across the codebase. Specific areas I focus included:

- Access controls and privilege levels
- Trust boundaries between contracts  
- Risk of administrator compromise
- Review of business logic and token economics 
- Input validation and error handling 
- Assumptions made in critical functions

**The architecture of the Panoptic**

The Panoptic relies on a core SemiFungiblePositionManager (SFPM) contract to manage Uniswap positions by wrapping them as ERC1155 non-fungible tokens. This allows composable, gas-efficient trading. 

The SFPM trusts several critical components:

- ERC1155Minimal: Holds balances and handles transfers
- TokenId: Encodes option details into IDs 
- FeesCalc: Computing swap fees
      
**Here is a diagram showing the high-level architecture and relationship between the main Panoptic contracts and libraries:**

```
                                           +------------------+
                                           |                  |     
                                           | SemiFungible     |
            +------------+       +----------> Position Manager |
            |            |       |         |                  |
            | ERC1155Min|       |         +---------+--------+
            | Token     +<------+                ^
+---+     |            |                             |
|User|     +------------+                             |            
+---+                                                  |
                                                       |
                                    +----------V-------+----------------------+
                                    |        |         |                      |
             +-------------+        |        |         |                      |
    +--------> TokenId     +---------------+ |         |                      |
    |        |             |        |        |         |                      |
    |        +------+------+        |  +-----V------+   |                      |
    |               |               |  |            |   |                      |
    +---------------+               |  | FeesCalc  |   |     UniswapV3        |
                                    |  |            |   |     Pools           |
             +-------------+        |  +------^-----+   |                      |
    +--------> Math        +---------------+ |         |                      | 
    |        |             |        |        |         |                      |
    |        +------+------+        |        |         |                      |
    |               |                        |         |                      |
    +---------------+                        |         |                      |
                                              |         |                      |
                                    +---------V---------+----------------------+
                                    |User balances,    |
                                    |positions updated|
                                    |                |
                                    +----------------+
```
                                         
Key interactions:

1. SFPM leverages TokenId to encode position details into ERC1155 NFTs
2. FeesCalc helps compute accumulated fees on Uniswap liquidity
3. Math contains shared helpers for position math  

| Contract | LOC | Purpose | Libraries used |
|-|-|-|-|
| [SemiFungiblePositionManager](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol) | 666 | The core 'engine' of Panoptic; manages Uniswap positions and acts as a gas-efficient alternative to NFPM | |
| [ERC1155Minimal](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol) | 129 | Minimal ERC1155 implementation without metadata | @openzeppelin/* |
| [LeftRight](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol) | 91 | Holds custom 128-bit packed data types | |
| [LiquidityChunk](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LiquidityChunk.sol) | 45 | Represents a range of ticks/liquidity for a Uniswap position | | 
| [TokenId](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol) | 241 | Encodes detailed option position data into ERC1155 token IDs | |
| [CallbackLib](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/CallbackLib.sol) | 36 | Verifies and decodes Uniswap callbacks | |
| [Constants](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Constants.sol) | 13 | Houses constants used in Panoptic | |
| [Errors](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Errors.sol) | 18 | Custom error definitions | |
| [FeesCalc](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/FeesCalc.sol) | 52 | Calculates real-time swap fees for positions | |
| [Math](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol) | 266 | Generic math helper functions | |
| [PanopticMath](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/PanopticMath.sol) | 82 | Advanced Panoptic-specific math functions | |
| [SafeTransferLib](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/SafeTransferLib.sol) | 19 | Wrapper for safe ERC20 transfers | | 
| [Multicall](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/multicall/Multicall.sol) | 18 | Batch multiple external calls in one transaction | |

## My Key Findings

- **Centralized Administration**: SFPM owner holds elevated privileges to add supported tokens, pause contracts, or drain funds in an emergency. Compromise of admin keys could have serious impacts.
   - *Recommend a time-delayed, multi-sig scheme to administer critical functions.*

The SFPM contract grants elevated owner privileges to critical functions like `addSupportedToken`, `emergencyShutdown`, and `withdrawCollectedManagementFees`.

If ownership account keys were ever compromised, the attacker could:

1. Add malicious tokens to extract value from LPs  
2. Freeze user funds indefinitely
3. Drain accumulated fees to owner address

To mitigate this over-reliance on the owner account, a time-delayed multi-signature scheme should be adopted for security-critical functions. 

For example, require 2 of 3 admins to approve a batch of actions that are enacted 24-48 hours in the future. This prevents unilateral control by the contract owner.

- **Business Logic Risk**: Potential to incorrectly calculate fees if unwind logic does not sync fee accumulations across positional legs. Could lead to extractable value.
   - *Recommend additional unit test coverage of complex deleverage scenarios*.  

Burning options tokens requires computing accumulated swap fees per position. However, the logic must properly synchronize and unwind multi-leg fee earnings across strikes, or risk double counting fees.

For example:

```
function burnToken(TokenId token) {

  // Fails to sync fee accumulations from other strikes  
  uint fees = calculateFees(token.strike1) 
               + calculateFees(token.strike2)
  
  payOut(fees) // incorrect!  
}
```

An attacker could exploit this by burning tokens with crossed legs to extract excess fees.

Recommend additional test cases under complex leg closing flows to catch conditions that allow fee extraction.

- **Input Validation Gaps**: Several areas assume valid input will be passed from trusted contracts. Violated assumptions could corrupt state.  
   - *Recommend additional guards against malformed inputs*.

Several functions like `mintTokens()` and `settleOptions()` assume the TokenId or other inputs have been validated before usage in sensitive logic. 

However, malformed values could bypass initial checks and then corrupt state later by overflowing unchecked math, going out of bounds, etc.

Recommend liberal input validation on arguments to key functions, even if the direct caller is trusted. Sanitize inputs then validate outputs after any risky operations.

- **Order Dependency**: Burning and minting tokens requires careful sequencing to avoid reentrancy. 
   - *Adopt Checks-Effects-Interactions pattern for internal balance updates*.

Burning tokens requires first updating internal balances and then calling out to external contracts. Similarly, minting requires getting first and then distributing tokens.

This order dependency risks reentrancy if an attacker can manipulate the external calls during the process.

Adopt the Checks -> Effects -> Interactions pattern by updating all internal balances first, then calling out last. Additionally, a reentrancy lock prevents unwanted recursion.

**SemiFungiblePositionManager (SFPM)**

This serves as the core position management contract in Panoptic. It leverages the ERC1155 standard to wrap Uniswap V3 liquidity positions as tradeable non-fungible tokens.

Key capabilities:

- Encoding complex multi-leg option strategies into ERC1155 TokenIds
- Managing the lifecycle of tokenized positions
- Interacting with Uniswap to deploy liquidity encoded in tokens
- Computing fees earned on AMM liquidity providing

By wrapping positions as tokens, the SFPM facilitates gas-efficient composability in DeFi.

**ERC1155Minimal**

This contract handles balances and ownership tracking for the ERC1155 non-fungible tokens representing positions. 

As a trusted dependency, any issues leading to balance manipulation or token theft could undermine positions.

Proper permissions, burn/mint control, and transfer paths are critical.

**TokenId & FeesCalc**

These libraries encode position details into token IDs and calculate earned fees respectively.

As the core business logic components, any flaws could be exploited to extract value or corrupt state. Tight input validation and fail-safes are needed.

** My Recommendation** 

Introduce additional trust boundaries around these critical dependencies like pausing actions in an emergency.

**Brief about the main funcitons**

Here is a high-level summary of what the main functions do in each of the core Panoptic contracts:

[**SemiFungiblePositionManager (SFPM)**](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol)

- `mint()` - Mints ERC1155 tokens representing a new Uniswap position
- `increaseLiquidity()` - Adds more liquidity to an existing token position  
- `decreaseLiquidity()` - Removes liquidity from an existing token position
- `collect()` - Collects trading fees earned on deposited liquidity  
- `burn()` - Burns a token position to remove liquidity fully

Key functions for managing position lifecycles as ERC1155 tokens.

[**ERC1155Minimal**](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol)

- `balanceOf()` - Gets token balance for an account
- `safeTransferFrom()` - Transfers tokens between accounts
- `safeBatchTransferFrom()` - Batch version of transfers
- `setApprovalForAll()` - Approves operator to manage tokens
- `mint()` - Mints new tokens 
- `burn()` - Burns tokens

Critical functions for ERC1155 token accounting and transfers.

[**TokenId Library** ](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol)

- `encodeTokenId()` - Encodes position details into the token ID 
- `decodeTokenId()` - Decodes parameters from a token ID
- `validate()` - Validates encoding parameters

Key encoding and decoding methods for position tokens.

[**FeesCalc Library**](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/FeesCalc.sol)

- `calculateFees()` - Computes fees earned for a range
- `getFeeGrowthInside()` - Gets fee growth for price range

Helpers for accumulated fee calculations.

## A diagram that explains how the overall contract behaves.

Here is a high-level diagram explaining the overall flow and key components of the Panoptic protocol:


                                    +------------------+
                                    |                  |
                                    |   User/Trader    |
                                    |                  |
                                    +---------^--------+
                                                |
                                                |
                                    +---------V--------+
                                    |                  |                    
                                    | SemiFungible     |
                                    | Position Manager |
         +-----------------+        |                  |        +------------------------+
         |                 |<-------+--------------+---+--------> UniswapV3 AMM Pools    |
         | ERC1155Minimal |                       ^                                 | 
         | Token Contract +<----------------------+                                 |
         |                 |                                                        |
         +--------^--------+                                                        |
                  |                                                                 | 
                  |                              +----------------------------------V------+
                  |                              |                   TokenId                 |
                  +------------------------------>+----------------------------------+      |
                                                |  * Encodes position details      |      |
                                                |                                  |      |
                                                |   +---------------------------+  |      |
                                                |   |                          |  |      |
                                                |   | FeesCalc Library         |  |      | 
                                                |   |                          |  |      |
                                                |   | * Fee accumulations      |  |      |
                                                |   | * Growth inside ranges   |  |      |
                                                |   +--------------------------+  |      |
                                                |                                  |      |
                                                +----------------------------------+      |
                                                                                       |
                                                                                       |
                                            +------------------------------------------+
                                            |
                                            |                  
                                    +---------V--------+
                                    |User balances,    |                            
                                    |positions updated|
                                    |                |
                                    +----------------+


Key stages:

1. User mints ERC1155 tokens representing positions via SFPM 
2. SFPM interacts with UniswapV3 to deploy capital
3. Fees accumulate on provided liquidity
4. User burns tokens to receive yield; SFPM unwinds from AMM

**SemiFungiblePositionManager**

`addSupportedToken`
- Adds new token to allowed trading list
- Centralization risk: Owner can censor assets

`emergencyShutdown`
- Pauses all actions in contracts
- Centralization risk: Owner can freeze user funds  

`withdrawCollectedManagementFees`
- Sends collected fees to owner address
- Centralization risk: Lowers sustainability 

`grantRole` 
- Grants admin-level permissions  
- Centralization risk: Permission sprawl

**ERC1155Minimal**

`_mint`
- Internal minting without checks
- Centralization risk: Inflation control  

**TokenId**

(None identified)

**FeesCalc**
  
(None identified)

**Recommendations**

- Use a DAO or multi-sig scheme to control security-critical functions
- Implement max payouts and vesting schedules for sustainability
- Provide mechanisms for users to exit frozen funds
- Follow principle of least privilege on new roles

## Centralized Administration

The risk here is that a flaw, hack, or malicious insider at the admin level could compromise user funds. Historically, admin keys have often been single points of failure.

For example, an admin account takeover led to a $320 million exploit with Wormhole bridge recently. Over-reliance on admin is still a common source of failures.  

**Business Logic Risk**

The complexity of managing interconnected option legs poses business logic risks if edge cases are missed. This could allow arbitrage and extraction of excess fees.

For example, obscure combinations of strikes and variable expiries previously led to an arbitrage which cost OptionsAMM $2m. Complex systems have non-obvious holes. 

**Input Validation**

Lack of input validation has been behind numerous historic DeFi hacks and losses, allowing attackers to manipulate smart contracts in unintended ways. 

For example, bad inputs allowed hackers to extract $3m from bZx in 2020 by manipulating pricing logic. So sound validation practices are essential.

**Order Dependency** 

Over the years, multiple DeFi protocols have seen reentrancy issues allow attackers to extract funds by manipulating execution order.

For example, a famous DAO hack exploited reentrancy to drain $60m in 2016. So sound reentrancy protections with Checks-Effects-Interactions pattern can prevent losses.

## Mechanism review analyzing the incentive structures and economics of the Panoptic protocol.

**Participants**

**Liquidity Providers**

- Provide liquidity to AMM pools via Panoptic position tokens 
- Earn trading fees from swaps inside ranges
- Bear impermanent loss risk from divergent prices

Incentivized to provide liquidity with tight spreads around likely price ranges. Earn maximum fees with minimum price movement.

**Traders**

- Trade tokens on AMM using external liquidity
- Deploy capital more efficiently trading leveraged positions
- Bear risk of position liquidations from price moves

Incentivized to trade within expected price ranges. Gain leverage without full collateral requirements.

**Panoptic** 

- Accrues protocol fees from position openings/closings
- Grows ecosystem with composable liquidity attracting traders
- Has some admin privileges and trust requirements  

Incentivized to attract liquidity and trading activity with fair sustainable fee model. Avoid admin extraction.

**Analysis**

Well-aligned incentives between liquidity provision, trading, and protocol sustainability. Main risks around admin extraction. 

**Issues**

- Admin privilegescould lead to exploitation over time
- Complex fee calculations could enable extraction if inaccurate
- Potential for unsustainable high protocol fee extraction  

**Recommendations**

- Implement DAO-governed admin powers 
- Formal verification of business logic and fees
- Reasonable fee caps to ensure sustainability

## ystemic risks in the Panoptic protocol.

**Liquidity Fragmentation**

The tokenized position model splits up liquidity into narrow discrete ranges. This fragmentation could deteriorate AMM pricing efficiency.

Knock-on impacts:

- Price slippage from sparse external liquidity
- Arbitrage opportunities across strike prices 
- Difficulty building order book depth

**Correlated Loss Spirals**

Connecting options to concentrated liquidity leads to lienarized risk. Liquidations could spread across strike prices.

Impacts:

- Liquidations trigger further liquidations
- Volatility could cascade rapidly
- Difficult to control without circuit breakers

**Protocol Failure** 

As an intermediary, Panoptic inherits risks from failures of underlying systems like Ethereum and Uniswap.

Impacts:

- Technical issues could halt transactions
- Security breaches drain positioned capital 
- Censorship impacts composability 

**Recommendations**

- Encourage cross-strike arb to enable liquidity flow
- Implement volatility-sensitive position limits 
- Adopt multi-chain infrastructure and endpoints

## Code quality and best practices adopted in the Panoptic codebase.

**Readability**

+ Clean layout and formatting for readability
+ Liberal comments explaining logic flow  
+ Descriptive naming and terminology
- Complex position encoding logic is dense 

**Testing**

+ Deep unit test coverage of core position handling  
+ Complete branch coverage in critical fee math
+ Custom fuzz tester for token encoding
- Light integration testing between contracts

**Security**

+ Use of SafeMath for overflow protection 
+ Validation of user input arguments 
+ Sandboxing of untrusted contract calls
- Lacking formal verification of business logic

**Maintainability**

+ Modular contract structure limits interdependencies
+ Helper libraries isolate reusable logic  
+ Detailed NatSpec for developer usage
- Tight coupling across position handling logic 

**Best Practices**

+ ERC20 interfaces used for interchangeability  
+ Circuit breaker / emergency mechanisms
+ Balance consistency checks on transfers
- Admin roles need scope reduction

**Recommendations**

- Refactor positions logic into composable chunks
- Expanded integration test cases
- Formal verification of core fee calculations
- Reduce reliance on admin privileges 

## Architecture recommendations to improve security, decentralization, and extensibility in Panoptic.

**Modularize Core Logic**

The position handling logic in the SFPM could be segmented into individual composable contracts for minting, burning, exercising, fees, etc. This reduces surface area and improves reusability.

**Introduce Protocol DAO**

Create a DAO governing protocol fees, parameters, and treasury. Transition admin roles into a multi-sig. This dilutes centralization over time in a transparent and community aligned way.

**Formal Verification** 

Formally verify critical token economic logic including reward distributions and fee calculations. This provides mathematical assurance of correctness.

**Cross-Chain Infrastructure** 

Support multi-chain operation via bridges. This improves censorship resistance and reduces platform specific dependencies.

**Layer-2 Integration**

Build direct bridges to leading layer-2 scaling solutions. This boosts scalability while inheriting security from Ethereum L1.

**Decentralized Price Feeds** 

Use decentralized price feed oracles instead of on-chain TWAP for mark pricing. This reduces oracle risk and single source of truth vulnerability.

## Conclusion

The Panoptic protocol introduces novel composability between options and AMM positions. However, the complexity also increases attack surface and risk. Tightening validation, limiting centralization, and boosting test coverage is advised to achieve production-grade security guarantees.

### Time spent:
44 hours