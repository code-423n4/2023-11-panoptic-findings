**Codebase Evaluation**

- High level architecture - components and data flows
- Test coverage across core functionality  
- Documentation quality and comments
- Potential central points of failure
- Complexity management - module boundaries

Central risk points:

- Lack of access control allows anyone enabling new pools
- Admin roles are implicit rather than explicitly declared

Overall relatively decentralized.

**Systemic Risks**

- No system-wide rate limits, but aggregate size caps reduce overflow risks
- Impacts likely confined to one pool if invariant violated
- Short liquidity reduces need for staggered exits

# Overview

Panoptic enables gas-optimized trading of complex multi-leg options positions by wrapping them into ERC-1155 tokens and managing the positions using the SemiFungiblePositionManager (SFPM).

**Key Capabilities:**

- Support for short positions by burning Uni v3 liquidity  
- Bundling multiple option legs across assets into single ERC-1155 tokens
- Swaps when minting in-the-money, paying with just one asset 
- Efficient gas usage via storage packing and custom data types

**Support for Short Positions**

A key innovation in Panoptic is enabling short positions by allowing users to burn Uni v3 liquidity. Typically in AMMs like Uniswap, users can only mint/add liquidity. But Panoptic introduces the concept of "removed liquididity" to track liquidity that is burnt.

This enables short positions analogous to short selling in TradFi markets. When a user mints a short call/put, they burn liquidity from Uniswap and hold the opposing token. As the price changes, they can buy back the liquidity for hopefully less, profiting on the price move.

Tracking removed liquidity is crucial for the fee accounting. As fees accumulate on liquidity sitting in Uniswap, the short position holder owes those fees to whoever minted that liquidity. So the system must track both gross and owed fee accumulations.

Enabling short positions unlocks flexibility in the option positions supported and expands the range of strategies available to traders.

**Bundling Option Legs**

To optimize capital efficiency and gas costs, Panoptic bundles multiple option leg positions across potentially different assets into a single ERC-1155 token. For example, a covered call holding ETH and selling a call could bundle both legs.

This is enabled by the custom `TokenId` data type, which packs up to 4 option legs along with the Uniswap pool address into a single 256-bit uint. Keeping the legs bundled reduces ERC-1155 token creation costs and eases tracking of complex multi-leg strategies.

Analysis of each leg still occurs independently on operations like minting or burning, but the overall accounting and balances are aggregated.

**Paying With One Asset** 

When options are minted in-the-money, some balancing is required because the strike price means tokens are owed to the pool. For example, minting an ITM ETH call requires both ETH and stablecoins to match the price.

Panoptic handles this via swaps during `mint()`, so users only need to pay with one asset type. This improves the UX and avoids needing to approve and staging multiple tokens.

**Gas Optimizations**

Several design decisions optimize the gas efficiency in Panoptic's position management:

- Tightly packed custom data types avoid waste 
- Storage packing reduces slots like dual usage for pool address & reentrancy check
- External pure math libraries compute complex formulas only once
- Checking owner approved for all rather than per token ID saves gas on transfer

The extensive usage of the custom math, packing approaches, and shared libraries provide gas savings that increase capital efficiency and reduce trader costs.

**Main Invariants**

The system preserves key invariants around permissioned liquidity withdrawals and fee accumulations. Users cannot withdraw/remove more than they deposited, and cannot collect more fees than earned.

The [SemiFungiblePositionManager](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol) enforces several key accounting and withdrawal invariants critical for operating a secure and fair liquidity management system:

**Restricted Liquidity Withdrawals**

Users can only withdraw/remove liquidity from Uniswap that they previously deposited themselves. The `s_accountLiquidity` mapping tracks each user's deposited liquidity amount by the tuple of (pool, owner, tick range). 

On any burn or withdrawal, this deposited amount must exceed the withdrawal amount or else it reverts. This prevents users withdrawing assets they don't rightfully own.

By tying withdrawals strictly to past deposits under the same keys, liquidity cannot be stolen or withdrawn beyond the user's ownership.

**Capped Fee Collection** 

Similarly, the fees a user can collect when calling `collect()` are capped based on ownership percentage of the pool liquidity. The `collect()` call references `positions()` data to get the total fees available, and limits collection to a percentage matched to ownership.

So users cannot simply drain all fees from the pool - they are restricted to the amount their liquidity has earned.

**Fair Fee Distribution**

When fees are disbursed to users, it is done so proportionally based on liquidity owned. This includes both standard LP and short-sold liquidity.

The `s_accountPremium` mappings track the accumulating owed and gross fees for each user's deposit. Assigning fees proportional to ownership ensures fair disbursal aligned to staked capital and risk.

By honoring these key principles, the SFPM prevents manipulation attacks around withdrawals and fee accrual that could otherwise drain funds.

## Architecture

**Custom Data Types**

Several custom data types are implemented to optimize storage and gas efficiency:

- [TokenId](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol) - Packs pool addresses and up to 4 option legs into one 256-bit uint
- [LiquidityChunk](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LiquidityChunk.sol) - Tracks tick ranges and liquidity amounts
- [LeftRight](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol) - Splits uints into two 128-bit chunks

**Fee Accumulation**

Complex accounting tracks minted/burned liquidity separately with accumulating owed/gross fee mappings. Ensures correct earnings.

**Reentrancy Protection** 

A per-pool reentrancy lock is packed into pool storage slots to save gas by reusing warmed storage.

**Size Limits**

Caps on max cumulative tokens and liquidity chunk sizes limit systemic risks.

## Recommendations

**Access Controls** 

Add governance controlled access management to critical functions like `initializeAMMPool()`.

**Math Libraries**

Expand usage of safe math libraries to bound risks in all intermediate calculations.

**Simulation Testing**

Implement simulation-based test suites to validate complete system behavior over long sequences of operations.



### Time spent:
16 hours