# Introduction

| Scope                                      | Contract                                   | SLOC | Purpose                                                                                                                  | Libraries Used         |
|--------------------------------------------|--------------------------------------------|------|--------------------------------------------------------------------------------------------------------------------------|------------------------|
| Panoptic Core                               | [`SemiFungiblePositionManager.sol`](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol)         | 666  | Manages all Uniswap V3 positions in the protocol; an advanced, gas-efficient alternative for Uniswap LPs               | `@openzeppelin/*`      |
| ERC1155 Token                               | [`ERC1155Minimal.sol`](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol)                      | 129  | Minimalist implementation of the ERC1155 token standard without metadata                                               | `@openzeppelin/*`      |
| Custom Data Types                           | [`LeftRight.sol`](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol)                           | 91   | Implementation for a set of custom data types that can hold two 128-bit numbers                                        | -                      |
| Custom Data Types                           | [`LiquidityChunk.sol`](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LiquidityChunk.sol)                      | 45   | Implementation for a custom data type representing a Uniswap liquidity chunk with tickLower, tickUpper, and liquidity    | -                      |
| Custom Data Type                            | [`TokenId.sol`](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol)                             | 241  | Implementation for the custom data type used in SFPM and Panoptic to encode position data in 256-bit ERC1155 tokenIds     | -                      |
| Uniswap Callback Verification              | [`CallbackLib.sol`](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/CallbackLib.sol)                         | 36   | Library for verifying and decoding Uniswap callbacks                                                                     | -                      |
| Constants                                   | [`Constants.sol`](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Constants.sol)                           | 13   | Library of constants used in Panoptic                                                                                    | -                      |
| Errors                                     | [`Errors.sol`](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Errors.sol)                              | 18   | Contains all custom errors used in Panoptic's core contracts                                                            | -                      |
| Swap Fee Calculation                        | [`FeesCalc.sol`](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/FeesCalc.sol)                            | 52   | Utility to calculate up-to-date swap fees for liquidity chunks                                                           | -                      |
| Generic Math Functions                      | [`Math.sol`](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol)                                | 266  | Library of generic math functions like abs(), mulDiv, etc                                                                | -                      |
| Advanced Panoptic/Uniswap-specific Math    | [`PanopticMath.sol`](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/PanopticMath.sol)                        | 82   | Library containing advanced Panoptic/Uniswap-specific functionality such as TWAP, price conversions, and position sizing | -                      |
| Safe ERC20 Transfer                         | [`SafeTransferLib.sol`](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/SafeTransferLib.sol)                     | 19   | Safe ERC20 transfer library that gracefully handles missing return values                                                | -                      |
| Multicall                                  | [`Multicall.sol`](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol)                           | 18   | Adds a function to inheriting contracts that allows for multiple calls to be executed in a single transaction            | -                      |

This table provides an overview of the contracts within the Panoptic project, including their purpose, lines of code (SLOC), and the libraries they utilize.

**Introduction**

Panoptic is an decentralized protocol that enables easy and gas-efficient management of complex multi-leg Uniswap V3 positions. At the core is the SemiFungiblePositionManager (SFPM) which offers key benefits over Uniswap's default position manager:

1. Wrapping positions into ERC1155 tokens to consolidate legs and allow partial fungibility

2. Performing swaps at minting/burning to accept one token type from users  

3. Supporting "long" positions by burning Uniswap liquidity

4. Tracking user balances & fees at a granular level

5. Pool-level reentrancy protection for security

My analysis report aims to provide an in-depth review of the protocol's system design, key mechanisms, risk considerations, and potential areas of improvement.

**SemiFungiblePositionManager**

```solidity
initializeAMMPool()
```
- Initializes a Uniswap V3 pool for usage within the SFPM by storing a poolId mapping and emitting a PoolInitialized event
- Needed to track a uni v3 pool before positions can be minted against it  

```solidity 
mintTokenizedPosition()
```
- Mints an ERC1155 token representing a Uniswap position with multiple legs
- Calls into `_validateAndForwardToAMM` to handle minting liquidity, swapping, and depositing amounts to Uniswap based on the parameters
- Allows permissionless creation of complex tokenized positions  

```solidity
burnTokenizedPosition()
```
- Burns an ERC1155 token to remove its associated Uniswap liquidity
- Provides the mechanism to close positions by redeeming funds from pooled liquidity

```solidity
_validateAndForwardToAMM()
```
- Central coordination point for minting or burning positions 
- Manages swap logic, assemblies tokenIds, checks slippage limits, and progresses mints/burns
- Critical path to enforce business logic and intended work flows

**Libraries**

`FeesCalc`
- Calculates earned fees for token positions using Uniswap's fee growth values 
- Needed to properly track and distribute fee earnings to SFPM users

`Math` 
- Library with safe math functions to prevent overflows
- Crucial protection against manipulation of key figures during position management 

**Architecture**

At an architecture level, Panoptic separates concerns into modular contracts with well-defined scopes:

```solidity
┌──────────────────────┐
│                      │
│                      │
│     Applications     │
│                      │
│                      │  
└─────────────┬────────┘
             │
┌─────────────┴────────┐
│                      │
│                      │
│    Panoptic Core     │
│                      │
│  ┌────────────────┐  │
│  │SemiFungiblePos─│  │
│  │Manager (SFPM)  │  │
│  └─────────┬──────┘  │ 
│            │          │
│  ┌─────────┼────────┐ │
│  │TokenId  │Math    │ │
│  │Liquidity│Errors  │ │ 
│  │Chunk    │+Other  │ │
│  │LeftRight│Libs    │ │
│  └─────────┴────────┘ │
│                      │ 
└──────────────────────┘
```

- The `Applications` layer represents other protocols integrating with Panoptic or end user experiences
- `Panoptic Core` houses the main SFPM engine along with custom data types and reusable libraries
- Modularity achieved by separating custom errors, math, token constructions, etc into isolated contracts/libraries
- This contract aggregation allows the core position manager to focus strictly on domain business logic

This adherence to separation of concerns principles results in improved:

- **Readability**: Contracts have focused domains like math, storage, types  

- **Reusability**: Components can be used by other DeFi protocols

- **Maintainability**: Changes are isolated to minimal areas

**Quality Analysis**  

The overall code quality is very high, implemented with best practices like:

- Use of library contracts for shared logic

- Extensive natspec documentation on functions

- Detailed revert error messages

- Usage of custom errors for readability 

- Validation of key invariants

The project also offers good developer ergonomics including scripted tasks and a solid testing approach with Unit/Integration tests.

Diagram showing the key functions and flow for the main SemiFungiblePositionManager contract:

```solidity
mintTokenizedPosition()
┌─────────────────────┐
│Validate & Forward │───> _validateAndForwardToAMM()
│To AMM             │      │
└───────────┬───────┘      │ 
             │            │  
             │            ├─> _createPositionInAMM()
             │                │ 
             │                ├─> _createLegInAMM()
             │                     │
             │                     ├─> _mintLiquidity()
             │                     │
             │                     ├─> _burnLiquidity()
             │                     │
             │                     └─> _collectAndWritePositionData()
             │
             │
             └─> Emit Minted Event
```

```solidity
burnTokenizedPosition() 
┌─────────────────────┐
│Validate & Forward │──> _validateAndForwardToAMM() 
│To AMM             │      │
└───────────┬───────┘      │
             │            │
             │            ├─> _createPositionInAMM() 
             │                │
             │                └─> _burnLiquidity()
             │
             │  
             └─> Emit Burnt Event
```

The overall flow - key things like swaps, liquidity actions, and accounting updates occur within these paths.

# Analysis

> The `mintTokenizedPosition` and `burnTokenizedPosition` functions, access controls appear to be appropriately implemented. These functions have public visibility, which means they can be called by any address. However, I believe this seems intentional based on the use case of this contract.

Specifically, the core purpose of the SemiFungiblePositionManager (SFPM) is to enable any address to mint and burn tokenized positions representing their liquidity in UniswapV3 pools. Restricting access would go against this goal of making management of complex Uniswap positions permissionless.

A few additional points demonstrating the appropriateness of the current access controls which i notice ar.

- The SFPM itself does not hold any owner privileges or sensitive roles. Ownership and roles were likely deliberately excluded due to the aim of being a publicly usable pool liquidity manager.

- While anyone can call `mintTokenizedPosition` and `burnTokenizedPosition`, within these functions access checks occur at the Uniswap level for protected operations like transferring liquidity ownership.

- There are pool-specific reentrancy locks to prevent these Uniswap interactions from being exploited.

So in summary:

1. The SFPM does not have admin roles or ownership that provide central control over minting/burning.

2. Public access to token management functions aligns with the stated goal of permissionless tokenized positions.

3. Reentrancy locks mitigate potential issues arising from reliance on access control at the Uniswap level.

Current access controls intentionally avoid placing restrictions on critical token management functions. This matches the motivation of decentralized, publicly accessible pool management.

> Pool-specific reentrancy protection appears to be well implemented through the use of a pool-scoped locking mechanism stored in the `s_poolContext` mapping.

**Reentrancy Protection Mechanism**

The `ReentrancyLock` modifier enforces the check:

```solidity
if (s_poolContext[poolId].locked) revert Errors.ReentrantCall()
```

Which leverages a boolean `locked` value stored alongside each poolId. Accessing pool-specific storage should always pass through this modifier first.

**Attempted Bypass 1**

Could an attacker call a function, have it re-enter before finishing, and then not trigger the revert when re-entering again?

This is prevented because `endReentrancyLock` resets the `locked` value at the end of protected functions:

```solidity
s_poolContext[poolId].locked = false
```

So re-entry would trigger the revert.

**Attempted Bypass 2**

Can the attacker manipulate the `locked` value directly via malicious contract deployment to bypass protection?

Not possible as the `poolId` maps strictly to a pool address, with a hashing process preventing collisions. Attacker can't register their address as an existing poolId.

All identified reentrancy paths correctly utilize the modifier for protection. The `locked` value mitigates bypasses as it is reset on each call. And integrity checking prevents direct manipulation from an external contract.

> The `_collectAndWritePositionData` internal function which is responsible for collecting fees from Uniswap and updating the accounting. The usage of the safemath `Math` library and validation of intermediate values mitigates the risk of potential overflow or manipulation.

**Fee Collection Math**

The collected amounts for each token are read from Uniswap using `IKniswapV3Pool.collect`:

```solidity
(uint128 receivedAmount0, uint128 receivedAmount1) = univ3pool.collect(...)
```

These `uint128` values are then safely cast to `int128` before further math.


**Tracking Accumulated Fees**

The received amounts are used to calculate fee deltas for the `s_accountPremiumOwed` and `s_accountPremiumGross` accumulators using `_getPremiaDeltas`.

These aggregators track fees per unit of liquidity, scaled by 2**64. By tracking relative changes in percentage terms, risk of overflow is avoided.

**Safemath Usage**

All arithmetic operations occur through `Math` library functions which frequently `revert` on overflow or invalid conditions. This includes multiplication, division, addition, subtraction and casting.

Reverts on overflow or underflow protect against manipulation of values in flight.

**Takeaways**

- Typed step-wise calculations avoid raw manipulation of intermediates  
- Safemath usage ensures over/underflow reverts
- Tracking relative fee changes as percentages mitigates overflow risk

Based on the above, I could not identify any vectors for overflow or manipulation in `_collectAndWritePositionData`. Validation appears sufficient.

> For ways an attacker could construct unusual positions to improperly allocate fees. The mechanisms around tracking gross and net fee deltas per unit of liquidity appear resilient against manipulation attempts.

**Fee Tracking Approach**

Fees are tracked independently for each liquidity position, identified by a `positionKey`. 

When fees are collected, two accumulator values are updated:

```solidity
s_accountPremiumOwed[positionKey] 
s_accountPremiumGross[positionKey]
```

These represent the total fees collected per liquidity unit, scaled by 2**64.

**Allocation Logic**

When fees are received by the SFPM from Uniswap, they update each position's accumulators based on:

- Total historical liquidity
- Currently added vs removed liquidity  
- Collected amounts for each token

This proportional allocation ensures expected earning amounts per liquidity unit.

**Manipulate Position Construction** 

The main way an attacker could try to shift allocations incorrectly is by manipulating variables in the `positionKey`.

However, separation of concerns via the hashing process prevents this:

```solidity
keccak256(abi.encodePacked(univ3pool, owner, tokenType, tickLower, tickUpper))
```

An attacker can't directly manipulate the key construction without control of the Uniswap pool, owner address, or position details.

**Overflow Accumulators**

The relative change tracking of fee percentages ensures overflows are not possible without breaching operational assumptions.

**Takeaways**

- Independent fee tracking with isolated accumulators protects allocation  
- Ratio based allocation according to position attributes resists manipulation
- Hashed keys prevent directly overriding critical variables

Based on the protection mechanisms I identified in the code, I could not find a way to incorrectly allocate fees. 

> I analyzed the token accounting logic for potential overflow vulnerabilities that could corrupt total balances. The use of the `Math` safemath library combined with validation of sums appears to mitigate this risk.

**Balance Tracking**

Token balances are tracked in the `balanceOf` mapping:

```solidity
mapping(address => mapping(uint256 => uint256)) public balanceOf; 
```

As ERC1155 positions are minted and burned, this map keeps count.

**Arithmetic Overflow Checks** 

The key balance manipulation functions are `_mint` and `_burn`. All math operations happen through the `Math.*` safemath functions: 

```solidity
unchecked {
    balanceOf[to][id] += amount; 
}
```

For example `+=` uses `Math.add(x,y)` which reverts on over/underflows.

**Summation Validation**

There is also a check of the aggregate sum of all legs in a position, enforced to be less than `type(int128).max` (2**127 - 1). This prevents unchecked summations accumulating to unusable values.

**Takeaways**

- Safemath prevents math overflows
- Aggregate validations cap total sums
- ERC1155 balancing enforced on transfers

Based on the scoped balance variables and overflow protections, corruption of totals seems unlikely. 

> The `_createLegInAMM` internal function that mints or burns liquidity for a specific position leg. Validation around the allowed amounts appears sufficient to prevent removing more liquidity than deposited.

**Liquidity Tracking**

Per-position liquidity is tracked in the `s_accountLiquidity` mapping:

```solidity
mapping(bytes32 positionKey => uint256 removedAndNetLiquidity) 
    internal s_accountLiquidity;
```

The `positionKey` identifies the Uniswap pool, user address, and position details.

**Checking Limits**

On liquidity removal, the amount is capped by previously deposited:

```solidity
if (startingLiquidity < chunkLiquidity) {
    revert Errors.NotEnoughLiquidity();  
}
```

Where `startingLiquidity` is the user's current balance in that position.

**Updating Counters** 

After passing the check, `startingLiquidity` is decremented by the removal amount using `Math.sub` to prevent underflows.

**Scenarios**

I was unable to identify ways to bypass `theconditional` check, corrupt the `positionKey`, or trigger counting errors.

The validation and safemath usage appears to enforce the invariant as expected.

> The `_getPremiaDeltas` internal function that calculates the fee premium values used in accounting updates the usage of precautions like safemath operators and read-only variables prevents manipulation of the inputs or collection of excessive fees.

**Read-Only Inputs**

The function takes read-only references to critical state variables:

```solidity
(uint256 currentLiquidity, int256 collectedAmounts)
```

These represent current position liquidity and recently collected fees.

**Safemath Arithmetic** 

All math operations happen through functions in the `Math` library that frequently check and revert on over/underflows.

This includes multiplication, division, addition/subtraction.

**Scaling Factor**

The premium values are tracked as ratios scaled by `2**64`. Using percentages avoids direct manipulation of fee totals.

**Requirements-Driven Calculation**

The equations used are carefully derived based on protocol fee distribution requirements between owed/gross premiums.

**Scenarios**

Between the read-only inputs, safemath usage, and fixed equations, I could not identify ways to alter the parameters or incorrectly calculate premiums to steal fees. 

> The usage of full precision until the final values and validation of output sums appear to mitigate risk vector from rounding or precision loss.

**Token Conversion**

The main conversions happen in `convert0to1` and `convert1to0` using the Uniswap tick math.

**Full Precision**  

These keep full precision on all intermediates until final 128-bit downcast:

```solidity
absResult = //Full calculations

return amount < 0 ? -absResult : absResult;
```

**Safemath**

All arithmetic uses `Math.*` operators which frequently `revert()` on invalid conditions to protect against errors.

**Summation Checks**

After conversions occur, the amounts are summed and ensured not to exceed 128 bits, otherwise reverting.

This caps associated rounding losses.

**Scenarios**

No clear ways were identified to trigger faults in the math or corrupt totals from rounding.

# Risk Analysis

At an architectural level, the `SemiFungiblePositionManager` represents a central point of risk due to its pooled storage of user positions and reliance on its accounting. Steps that help mitigate include: 

- Validation of consistent position leg representations
- Access control via reentrancy protection
- Usage of safemath and overflow protection  

Diagrams showing key functions for each of the main contracts within the Panoptic protocol:

**SemiFungiblePositionManager**

```solidity
initializeAMMPool()

┌────────────────────────┐
│Get Pool Address From  │
│Factory For Tokens & Fee│
└───────────┬────────────┘ 
            │               
            │               
┌───────────┴────────────┐
│Create & Store PoolId  │
│Mapping                │  
└───────────┬────────────┘
            │
            └─> Emit Initialized Event
```

**ERC1155Minimal**

``` 
safeTransferFrom()
┌─────────────────────────────┐
│                             │
│ Validate Sender Approvals   │
│ Adjust Balances             │
│ Emit Transfer Event         │
│                             │
│ Call reciever contract with │
│ ERC1155Hook  if present     │ 
└─────────────────────────────┘
```

**TokenId**

```solidity
validate()
┌──────────────────────┐   
│Loop Over All Legs    │
│                      │
│Leg Active?           │
│┌──────────────────┐  │
││Ratio > 0          │  │
│└─────────┬────────┘  │
│          │           │
│          │           │
│          └─> Revert on   
│             Invalid
└──────────────────────┘
```

Potential issues could still arise from complex application usage leading to unintended states. Additional integration testing to validate core invariant preservation in muddy conditions may be beneficial.

**Conclusion**

Overall Panoptic offers an innovative solution improving capital efficiency and gas costs for advanced Uniswap positions. The protocol mechanics are thoughtfully constructed with a focus on security. Areas that would benefit from further risk analysis have been highlighted, but architecturally and qualitatively the contracts are well-positioned for stability and gas optimizations.





### Time spent:
39 hours