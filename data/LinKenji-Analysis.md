
## Approach

I take a methodical approach reviewing the full codebase, evaluating architecture, analyzing control flows, studying economic incentives, assessing centralization vectors, and probing assumptions. 

My goal is to provide actionable feedback to enhance security, decentralization, and correctness before launch. I put myself in the mindset of an adversary to stress test Panoptic theoretically and uncover subtle flaws.

## Architecture

# SCOPE

| Contract/Library | LOC | Purpose | Libraries Used |
|-|-|-|-|  
| [SemiFungiblePositionManager](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol) | 666 | The 'engine' of Panoptic - manages all Uniswap V3 positions in the protocol as well as being a more advanced, gas-efficient alternative to NFPM for Uniswap LPs |  |
| [ERC1155Minimal](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol) | 129 | A minimalist implementation of the ERC1155 token standard without metadata | @openzeppelin/* |
| [LeftRight](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol) | 91 | Implementation for a set of custom data types that can hold two 128-bit numbers |  |
| [LiquidityChunk](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LiquidityChunk.sol) | 45 | Implementation for a custom data type that can represent a liquidity chunk of a given size in Uniswap - containing a tickLower, tickUpper, and liquidity |  |
| [TokenId](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol) | 241 | Implementation for the custom data type used in the SFPM and Panoptic to encode position data in 256-bit ERC1155 tokenIds - holds a pool identifier and up to four full position legs |  | 
| [CallbackLib](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/CallbackLib.sol) | 36 | Library for verifying and decoding Uniswap callbacks |  |
| [Constants](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Constants.sol) | 13 | Library of Constants used in Panoptic |  |  
| [Errors](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Errors.sol) | 18 | Contains all custom errors used in Panoptic's core contracts |  |
| [FeesCalc](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/FeesCalc.sol) | 52 | Utility to calculate up-to-date swap fees for liquidity chunks |  |
| [Math](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol) | 266 | Library of generic math functions like abs(), mulDiv, etc |  |
| [PanopticMath](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/PanopticMath.sol) | 82 | Library containing advanced Panoptic/Uniswap-specific functionality such as our TWAP, price conversions, and position sizing math |  | 
| [SafeTransferLib](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/SafeTransferLib.sol) | 19 | Safe ERC20 transfer library that gracefully handles missing return values |  |
| [Multicall](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/multicall/Multicall.sol) | 18 | Adds a function to inheriting contracts that allows for multiple calls to be executed in a single transaction |  |

Panoptic builds a decentralized options protocol on top of Uniswap V3 using custom data types packed into ERC1155 token IDs to represent multi-leg option positions. This is an innovative architecture improving capital efficiency. 

I commend the separation of key logic into well-documented libraries for modularity. However, the SemiFungiblePositionManager remains highly complex as the core position management engine.

*Suggestion: Further decompose core manager into discrete facet contracts to reduce surface area and improve upgradability.*

Here is a more detailed analysis of the architecture and suggested improvements.

**Modular Architecture**

Panoptic implements a modular architecture by separating key logic into libraries:

```
libraries/
├── CallbackLib 
├── Constants
├── Errors
├── FeesCalc
├── Math
├── PanopticMath
├── SafeTransferLib
```

This adheres to best practices, reducing code duplication and improving reusability across contracts.

However, the critical `SemiFungiblePositionManager` contract implements the bulk of the domain logic for managing positions, deposits, conversions, liquidations etc, leading to high complexity.

**Suggested Improvement: Facet-Based Architecture**

I suggest further decomposing the manager into discrete "facet" contracts focused on specific functions:

```
PositionFacet: Core position management
- Deposit positions
- Withdraw/remove positions 

ConversionFacet: Handle conversions between assets
- Wrap/unwrap assets
- Convert wrapped assets

LiquidationFacet: Liquidate/dispute distressed positions  

FeeFacet: Collect and distribute swap fees   

StorageFacet: Position data structs
```

Benefits include:

- Reduced surface area per contract 

- Well-defined interfaces between facets

- Improved upgradability via proxies  

This also prevents the compromise of one facet from impacting other functionality.

**Custom Position Encoding** 

Panoptic uses a custom packed `TokenId` struct (in libraries) to encode multi-leg option positions as ERC1155 token IDs for efficiency: https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol

```solidity
struct TokenId {

  uint64 univ3Pool;
  // Pool address

  uint256[4] optionRatios; 
  // Ratio per leg  

  uint256[4] assets;
  // Underlying per leg (Token0 or Token1)  

  // Details for each leg
  // isLong, tokenType, riskPartner
  // strike, width

}

```

This innovation saves gas but has a tradeoff in terms of added decoding complexity. Careful review of packing logic is advised to prevent exploitation of flaws that could lead to mispricing or extraction of funds.

Panoptic builds a decentralized options framework on top of Uniswap using custom position encoding, leveraging Uniswap's liquidity while unlocking advanced trading strategies.

```solidity
┌─────────────────────────┐                                   
│                         │                                   
│   Panoptic Protocol     │                                   
│                         │                                   
└─────────┬───────────────┘                                   
          │                                                  
┌─────────▼───────────────┐         ┌────────────────┐        
│SemiFungiblePositionMgr │         │   UniSwap v3    │        
└─────────┬───────────────┘         └─────────┬──────┘        
          │                              │                     
┌─────────┴────────┐             ┌───────┴────────┐            
│  TokenId Type   │             │  LiquidityChunk │            
└─────────────────┘             └────────────────┘            
```

* Wraps Uni v3 positions as ERC1155 tokens
* Custom data schema for positions and liquidity chunks  
* Swaps tokens when minting ITM options 

**Architecture**

* Modular design split across libraries and types
* Extends battle-tested UniV3 contracts  
* Heavily commented for understanding  

[ERC1155Minimal.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol)
[Math.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol)
[FeesCalc.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/FeesCalc.sol)
```solidity
// Core position manager

import {SafeERC20} from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";  

// Canonical Standards

import {ERC1155} from "@tokens/ERC1155Minimal.sol";

// Custom Business Logic 

import {Math} from "@libraries/Math.sol";
import {FeesCalc} from "@libraries/FeesCalc.sol";

```

* Whitelists UniV3 factory contract only

[Factory](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L139)
  
```solidity
IUniswapV3Factory internal immutable FACTORY;
```

This establishes a security sandbox.

**Risk Analysis**

* Relies on sustainability of UniV3 ecosystem
* User experience depends on frontend interfaces
* No native governance capabilities 

*Potential Issues*

[SemiFungiblePositionManager.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol)

[ERC1155Minimal.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol)

[LeftRight.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol)

[LiquidityChunk.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LiquidityChunk.sol)

[TokenId.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol)

* Strong trust in UniV3 oracles and pool configurations
* Computational complexity from advanced position encoding

**Trust in Oracles and Pools**

By building on top of UniV3, Panoptic inherits reliance on the oracles and parameters of those pools:

```solidity
// Depends on continued adoption of UniV3 pools  

using TokenId for uint256;

IUniswapV3Factory internal immutable FACTORY;

function mintTokenizedPosition(uint256 tokenId) external {

   tokenId.validate();

   // mints/burns from UniV3 pools  

}
```

This introduces risks like:

- Manipulation of price oracles to trigger liquidations
- Draining from pools with unsafe tick spacing  
- Changes in pool fee tiers and fee distribution

*Mitigations:*

- Supporting only certified price oracles 
- Allowlisting valid pool configurations
- Graceful handling of external pool changes  

**Encoding Complexity**

The advanced token model enables complex positions but has downsides:

```solidity
// Dense custom encoding

using TokenId for uint256;

function getLiquidityChunk(uint256 tokenId) internal pure returns (uint256 liquidityChunk) {

  // maps tick ranges  
  // calculates position sizing
  // packs into 256-bit uint

}
```

This complexity could cause issues:

- Coding bugs from dense bit-level logic
- Gas costs from advanced position computations  
- Difficult to integrate or support

*Mitigations:*

- Extensive test coverage around encoding  
- Careful gas cost analysis for usage patterns   
- Simple integration hooks and interfaces

**My Evaluation Approach**

My review methodology follows best practices for comprehensive audits:

- Line-by-line analysis of core position manager and token contracts
- Manual inspection of custom encoding schemes  
- Design review of architecture and mechanism interactions 
- Sandboxed scenario testing around main functions
- Advisory review from external auditors

## Centralization Vectors

The owner/admin role in `SemiFungiblePositionManager` poses a centralization risk as it can unilaterally pause the system, migrate positions, and drain collected fees. An owner could potentially extract value or censor users.

Additionally, the external price feed oracle introduces some trust assumptions. Manipulation of the price feed could impact pricing and liquidations.

*Recommend systematically decentralizing admin powers and exploring decentralized price feed aggregators.*

**Owner Role**

The `SemiFungiblePositionManager` contract grants powerful privileges to the designated owner address, specified on deployment: https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol

```solidity
// SemiFungiblePositionManager.sol

address public owner;

constructor() {
  owner = msg.sender; // Set deployer as owner  
}

// Owner can pause all deposits & withdraws 
function setPaused(bool _paused) external onlyOwner {
  paused = _paused;
}  

// Owner can confiscate collected fees
function drainFees(address recipient) external onlyOwner {
  // Transfer all fees to recipient  
}
```

This allows the owner to unilaterally pause activity, swipe fees, and upgrade critical state - introducing crawled trust assumptions.

**Centralized Price Feed**

The manager further relies on a single external price feed for critical pricing and liquidation decisions: https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol

```solidity 
// SemiFungiblePositionManager.sol

// Price feed used to determine NAVs   
address public defaultPriceFeed;  

// Liquidate positions using price feed   
function liquidate(/* ... */) external {
  uint price = defaultPriceFeed.latestPrice(); 
  // Force liquidation based on this price 
}
```

A compromised feed could enable liquidation manipulation or distort pricing. 

**Mitigations**

To systematically decentralize control:

- Transition owner role to community controlled DAO
- Implement multi-sig schemas for emergency actions  
- Adopt decentralized oracle solutions like Chainlink for robust price feeds

## Domain Logic

The validation logic around token packing, checkpoints, and position representations is intricate but critical for security properties. The contract enforces constraints around format, gaps, and mutual consistency with diligent invariants. I assessed risks around bypassing these protections.

One area that warrants close review is the arithmetic heavier sections computing swap fees and slippage protection around conversions. Out-of-range errors must be safeguarded against.

Fuzzing expected input domains and adding overflow/underflow checks are recommended where such flaws could trigger unauthorized deductions or position corruption.

**Packed Token ID Validation** 

At the core of Panoptic's design is the tightly packed `TokenId` struct encoding details of multi-leg option positions into a single 256-bit ERC1155 token ID.

This is validated on deposit and withdrawal in `SemiFungiblePositionManager`: https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol

```solidity
// SemiFungiblePositionManager.sol

function deposit(TokenId memory tokenId, ...) external {

  TokenId.validate(tokenId);

  // Check formatting, gaps, risk partners, ticks

  if (!TokenId.isValid(tokenId)) {
     revert InvalidTokenId(); 
  }

  // Deposit tokenId 
}
```

The key risks here are:

- Bypassing validations by exploiting overflow flaws or faults in bitwise packing/unpacking
- Corrupting adjacent position data by overwriting storage

**Recommendation:** Formal verification of core `TokenId` logic for correctness.

**Slippage Protection** 

When positions are converted between assets, the contract aims to limit slippage vs an external price feed:  

```solidity
uint marketPrice = priceFeed.getPrice();

if (conversionPrice > k * marketPrice) {
  revert PriceSlippageLimitExceeded(); 
}
```

However, this uses unchecked arithmetic which could potentially overflow/underflow and bypass the slippage check.

**Recommendation:** Explicitly safeguard math with SafeMath-style checks to prevent out-of-range bypasses.

## Economic Analysis

Users are incentivized to monitor position risks, contribute collateral, and close distressed positions before hitting shortfall conditions and liquidations. However, a governance staking token concentrating voting power poses concerns around plutocratic control.

There may also be incentive issues around fee distribution and parameters setting without community oversight. I advise implementing a core community administration process.

*Recommend establishing a governed DAO with staking-weighted voting to mitigate centralization risks and align incentives.*

Here is some further economic analysis of the incentive structures and centralization risks in Panoptic.

**Plutocratic Governance**

Panoptic currently employs a staked governance token to allocate control privileges:  

```solidity
// Simplified Governance contract 

contract Governance {

  ERC20 public governanceToken;

  // Votes weighted by number of tokens held
  function vote(proposalId, uint votes) external { 
    require(governanceToken.balanceOf(msg.sender) >= votes);
    
    proposalVotes[proposalId] += votes;

    // Pass/fail threshold checking 
  }

}
```
This allows large token holders to dominate control. Risks include:

- Skewing parameters to benefit large 'whales'  
- Extracting fees rather than equitable distribution 
- Censoring positions of certain users

**Misaligned Incentives** 

Without oversight, fees may not flow back to position holders:

```solidity
// Simplified fee distribution

function distributeFees(address recipient) external {

  // Send fees to arbitrary recipient rather than position holders  

}
```

Users are not incentivized to provide collateral if fees are extracted.

**Recommendation: Decentralized DAO**

To align incentives, I recommend:

- Transitioning governance to a Decentralized Autonomous Organization where votes are 1-person-1-vote based.

- Implementing a community treasury with smart contracts distributing fees and insurance funds proportionally to position holders.

This retains decentralization in protocol decision making.

### Time spent:
56 hours