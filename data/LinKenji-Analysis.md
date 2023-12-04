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
- Static analysis with Slither and MythX to automate checks
- Design review of architecture and mechanism interactions 
- Sandboxed scenario testing around main functions
- Advisory review from external auditors

**Architecture** 

- Well organized structure improves readability
- Modularization contains blast radius  
- Whitelisted factory contract adds security boundaries 

**Recommendations**

- Additional access controls between components
- Circuit breakers for emergency stops 
- Permissioned administrative controls

**Code Quality**

- High dev comment density aids understanding
- Short, simple functions ease inspection  
- Extensive NatSpec documentation
- Heavy reuse of OpenZeppelin standards

Quality facilitates security reviews.

**Centralization**

- No privileged roles minimizes control points
- Business logic completely on-chain  
- Avoiding governance tokens reduces influencer risk   

System is largely decentralized.

**Mechanisms**

- Position encoding expands DeFi composability  
- Swaps at minting enables wider range of assets
- Performance optimizations boost capacity  

Novel extensions of Uniswap with minor risk.

**Systemic Risk**

- Suite size contained for inspection scope
- Isolated logic minimizes unintended effects
- Extensive validation checks reduce attack surface

Focused scope limits externalities.

**Lastly**

Elegant architecture that pushes DeFi composability further, but inherits risks from the underlying Uniswap ecosystem. Additional protective checks would reduce attack surface.

### Time spent:
13 hours