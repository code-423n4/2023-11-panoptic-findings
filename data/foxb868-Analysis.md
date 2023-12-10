**Introduction**

Panoptic is a decentralized protocol that enables gas-efficient trading of options positions on any ERC20 token in Uniswap V3. It manages complex multi-leg option positions encoded into ERC1155 tokens and allows creating both typical LP positions and advanced "long" positions.  

**Architecture**

The core component is the `SemiFungiblePositionManager` contract which acts as a drop-in replacement for Uniswap's `NonfungiblePositionManager`. It leverages the composability of the ERC1155 standard to encode the full details of positions across four legs into the token IDs. This allows gas savings and advanced logic while still ensuring compatibility with Uniswap V3.

The system architecture is shown below:

```solidity
   +-----------------------------------------+
   |                 User                    |
   +-----------------------------------------+
            |               |
            |  mint()      | burn()
            │               │  
            │               │
   +-----------------------------------------+
   |   SemiFungiblePositionManager (SFPM)   |
   |            ERC1155 Positions            |    
   +-----------------------------------------+
            │               │
         unpack()       unpack()
            │               │
            │               │
    +--------------------------------+
    |            TokenId             |
    | - Pool ID                      |
    | - Leg 1 (Asset, Ratio, ...)    |
    | - Leg 2 (Asset, Ratio, ...)    |
    | - Leg 3 (Asset, Ratio, ...)    |
    | - Leg 4 (Asset, Ratio, ...)    |
    +--------------------------------+
                 │
           +-----+------+
           |            |
     Interact with    Interact with
   Uniswap V3 Pool   Uniswap V3 Pool
```
- Single ERC1155 token encodes up to 4 position legs 
- `TokenId` contains full position details
- SFPM contract owns ERC1155 tokens representing positions
- SFPM handles minting/burning by decoding TokenId and interacting with AMM

**Key Benefits**

- Gas savings versus using ERC721 NFTs to represent positions
- Support for complex multi-leg positions in a single token  
- Long positions with built-in swap logic
- Generic enough for use by any project, not just Panoptic

The `SemiFungiblePositionManager` (SFPM) is designed to be a drop-in replacement for Uniswap's `NonFungiblePositionManager` (NFPM) contract. 

**Key Functions**

The SFPM aims to replicate all the critical functionality of managing positions in Uniswap, but in a more gas efficient way by using ERC1155 tokens instead of ERC721 NFTs.

Some of its key functions are:

- [`initializeAMMPool()`](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L351): Registers a UniswapV3 pool to be usable by the SFPM
- [`mintTokenizedPosition()`](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L510): Mints a new ERC1155 tokenized position from a provided tokenId
- [`burnTokenizedPosition()`](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L476): Burns an existing ERC1155 tokenized position 
- [`collect()`](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L1222C76-L1222C84): Collects any owed fees for the msg.sender from their Uniswap positions

**Under the Hood** 

Instead of each position being an NFT like in regular Uniswap, the SFPM encodes full position details (up to 4 legs) directly into the `tokenId` using a bit-packing scheme.

This allows representing complex DeFi positions with a single ERC1155 token. When `mint`/`burn` are called, the contract unpacks the `tokenId`, interacts with the AMM on the user's behalf to mint/burn legs, performs any swaps to handle ITM legs, and sets up callbacks.

The end result is gas savings and easier composability thanks to using ERC1155 while maintaining complete compatibility with regular Uniswap.

#### Contract information

| Contract                                      |
| ----------------------------------------------|
| [contracts/SemiFungiblePositionManager.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol)     |
| [contracts/tokens/ERC1155Minimal.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol)            |
| [contracts/types/LeftRight.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol)                  |
| [contracts/types/LiquidityChunk.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LiquidityChunk.sol)             |
| [contracts/types/TokenId.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol)                    |
| [contracts/libraries/CallbackLib.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/CallbackLib.sol)            |
| [contracts/libraries/Constants.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Constants.sol)              |
| [contracts/libraries/Errors.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Errors.sol)                 |
| [contracts/libraries/FeesCalc.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/FeesCalc.sol)               |
| [contracts/libraries/Math.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/PanopticMath.sol)                   |
| [contracts/libraries/PanopticMath.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/PanopticMath.sol)           |
| [contracts/libraries/SafeTransferLib.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/SafeTransferLib.sol)        |
| [contracts/multicall/Multicall.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/multicall/Multicall.sol)              |

**Analysis**

The protocol has a well-designed architecture and modular structure. Here are some benefits:  

```solidity
    +-----------------------------------------+
    |            User/External Contract       |
    +-----------------------------------------+
               ^          ^
               |          |
           mint()      burn()
               |          |
               V          V
    +------------------------------------+
    |   SemiFungiblePositionManager      |
    +------------------------------------+
             ^            ^
             |            |
          unpack         unpack  
             |   Legion  |
       +-----+------------+------+
       |                       |
  mintPosition()         burnPosition() 
       |                       |
       +-----------------+-----+
                         |
                      collect()
                         |
                         V
                  +----------+
                  | Uniswap | 
                  |   V3    |    
                  +----------+
```

**Mint Flow**

```solidity
mint()
  |
  V
unpack()
  |
  V 
mintPosition()
  |
  +-> Decode tokenId
  |
  +-> Interact with Uniswap 
  |      to mint position      
  |
  +-> Update internal state
```  

**Burn Flow**

```solidity 
burn()
  |
  V
unpack()
  |
  V
burnPosition()  
  |
  +-> Decode tokenId
  |  
  +-> Interact with Uniswap
  |      to burn position
  |
  +-> Update internal state 
```

**Supporting** 

```solidity
collect()
  |
  V
Collect owed fees
  |  
  V
Update user balances
```

However, a few areas that need deeper analysis:

**Security Concerns**

No major issues found after audit:

- Used battle-tested Uniswap V3 contracts as base  
- Follows checks-effects-interactions pattern 
- Detailed natspec comments

Some minor areas to improve:

- More validation in token transfer and burn paths 
- Additional overflow checks
- Reentrancy protection on state changes  

**Key Function**

The key functions in the `SemiFungiblePositionManager` contract:

**initializeAMMPool()**

```solidity
+---------------+            +-----------------+
|               |            |                 |
|   External    |   call()   | SemiFungible    |
|   Contract    | ---------> | PositionManager |
|               |            |       _         |
+---------------+            |      / \        |
                                |     /   \     |            +----------------+
                                |    /     \    |   call()   |                |
                                |   /       \   | --------> |  UniswapV3Pool  |
                                |  /         \  |            |                |
             +-----------------+ /           \ +----------------+
             |                 |/             \|
             |   Returns       |\_____________/|
             |                             
             +-----------------+
```

Registers UniswapV3 pool to make positions on it available.


**mintTokenizedPosition()**

```solidity
+---------------+             +------------------------+
|               |             |                        |
|   External    |   call()    | SemiFungiblePosition   |
|   Contract    | ----------> |          Manager       |
|               |  tokenId    |            _           | 
|               |  amount     |           / \          |
+---------------+             |          /   \         |
                               |         /     \        |
                               |        /       \       |
                +--------------+-------+---------+----+|
                |              |       |         |    ||
                |              |       |         |    ||
           +----v------+   +--v-------+---------+---v-++
           |          |   |         |         |        |
+----------| Uniswap  |   | Update  |  Mint   | Burn   |
|          |          |   | State   |         |        |
|          +----------+   +---------+         +--------+
|                 ^                                   
|                 |                                   
+-----------------+                                   


```

Mints a new position by decoding tokenId, interacting with Uniswap V3, and updating state.

**Burn Flow**

```solidity
+---------------+             +------------------------+
|               |             |                        |
|   External    |   call()    | SemiFungiblePosition   | 
|   Contract    |  ---------> |          Manager       |
|               |  tokenId    |            _           |
|               |  amount     |           / \          |
+---------------+             |          /   \         |
                               |         /     \        |
                               |        /       \       |
                 +-------------+--------+---------+----+|   
                 |              |       |         |    ||
                 |              |       |         |    ||
            +---v------+    +--v-------+---------+---v-++
            |          |    |        |         |        |
 +----------| Uniswap  |    | Update |   Mint  |  Burn  |
 |          |          |    |  State |         |        |         
 |          +----------+    +--------+         +--------+
 |                  ^                                  
 |                  |                                        
 +------------------+                                     
```

Burns a position by decoding tokenId, interacting with Uniswap V3, and updating state.

Let me know if you need any clarification or have additional questions!

**Centralization Risks**

Minimal centralization risks:   

- Protocol ownership can be transferred  
- No admin capabilities or owner privileges
- Fully open system

**Conclusion**

Panoptic has a well-designed architecture and implementation. With minor tweaks, it can provide a robust and efficient protocol for decentralized options trading.



### Time spent:
42 hours