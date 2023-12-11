## What is Panoptic?

The Panoptic protocol consists of smart contracts on the Ethereum blockchain that handle the minting, trading, and market-making of perpetual put and call options.

Panoptic is the first permissionless options protocol that overcomes the technically challenging task of implementing an options protocol on the Ethereum blockchain.

In a permissionless options protocol like Panoptic on the Ethereum blockchain, users can engage in options trading without needing approval from a central authority. This decentralization is achieved through smart contracts and blockchain technology, allowing users to interact with the options protocol directly and without relying on intermediaries. It aligns with the principles of decentralized finance, providing users with more control over their financial activities.

## Overview

The SemiFungiblePositionManager is a gas-efficient alternative to Uniswap’s NonFungiblePositionManager that manages complex, multi-leg Uniswap positions encoded in ERC1155 tokenIds, performs swaps allowing users to mint positions with only one type of token, and, most crucially, supports the minting of both typical LP positions where liquidity is added to Uniswap and “long” positions where Uniswap liquidity is burnt.

This contract is a component of the Panoptic V1 protocol, but also serves as a standalone liquidity manager open for use by any user or protocol.

# Audit approach

1.  Read the documentation.
2.  Try to understand how the system works by looking at the docs and  website
3.  Look at each code individually and focus on the internal function calls.
4.  Read the test files to get a better idea of the end-to-end scenarios.
5.  Write a Report by compiling all the insights I gained throughout the line-by-line code review.

&nbsp;

### Contracts in Scope

- [ ] **SemiFungiblePositionManager.sol :** it is an ERC1155 token contract designed to manage positions on Uniswap V3. It allows users to wrap Uniswap V3 positions with up to 4 legs behind an ERC1155 token, aiming to be a gas-efficient alternative to Uniswap's NonfungiblePositionManager (ERC721).

core functions:

- `mintTokenizedPosition`: Allows users to create a new position with up to four legs, each with its own range and liquidity amount. The position is represented as an ERC1155 token.
    
- `burnTokenizedPosition`: Allows users to destroy their position, removing liquidity from the Uniswap V3 pool and collecting any accumulated fees.
    
- `initializeAMMPool`: Initializes a Uniswap V3 pool within the contract, enabling it to manage positions in that pool.
    
- `getAccountLiquidity`, `getAccountPremium`, and `getAccountFeesBase`: Provide information about the liquidity, premium, and fees associated with a user's position.
    
- `getUniswapV3PoolFromId` and `getPoolId`: Utility functions to retrieve the Uniswap V3 pool associated with a given pool ID and vice versa.
    

* * *

- [ ] **ERC1155Minimal.sol:** A minimalist implementation of the ERC1155 token standard without metadata

core functions:

- `balanceOf`: A mapping that keeps track of the number of tokens each account holds for each token ID.
- `isApprovedForAll`: A mapping that tracks which operators are approved to manage all of an owner's tokens.
- `setApprovalForAll`: Allows a token owner to grant or revoke permission to an operator to manage all of their tokens.
- `safeTransferFrom`: Enables a user to transfer a specific amount of a token type to another user, with safety checks for smart contract recipients.
- `safeBatchTransferFrom`: Similar to `safeTransferFrom`, but allows for the transfer of multiple token types in a single transaction.
- `balanceOfBatch`: Allows querying the balance of multiple token types for multiple users in a single call.
- `supportsInterface`: A function to indicate support for the ERC1155 and ERC165 interfaces.
- `_mint` and `_burn`: Internal functions to create (mint) new tokens or destroy (burn) existing tokens.
- `afterTokenTransfer`: Internal hooks that can be overridden by inheriting contracts to add custom logic after transfers.

* * *

- [ ] **LeftRight.sol : it** designed to pack and manipulate two separate 128-bit data chunks within a single 256-bit slot.

core functions:

The core function of the `LeftRight` library is to efficiently pack and manipulate two 128-bit values within a single 256-bit storage slot in a Solidity smart contract. This is done to save on storage costs and optimize data handling by taking advantage of the Ethereum Virtual Machine's (EVM) 256-bit word size. The library provides methods to:

- Retrieve the left or right 128-bit portion of a 256-bit number.
- Insert or overwrite a 128-bit value into the left or right side of a 256-bit number.
- Perform addition and subtraction on these packed values while handling overflows and underflows.
- Safely cast larger integer types to smaller ones, ensuring that the values fit within the target type's range.

* * *

- [ ] **LiquidityChunk.sol :** it is used to encode and decode information about a "liquidity chunk" within a concentrated liquidity Automated Market Maker (AMM).

core functions:

- `createChunk`: Initialize a new liquidity chunk with specified lower and upper ticks and a liquidity amount.
- `addLiquidity`: Add liquidity to an existing chunk.
- `addTickLower`: Set the lower tick of a chunk.
- `addTickUpper`: Set the upper tick of a chunk.
- `tickLower`: Retrieve the lower tick from a chunk.
- `tickUpper`: Retrieve the upper tick from a chunk.
- `liquidity`: Retrieve the liquidity amount from a chunk.

* * *

- [ ] **TokenId.sol** : it is  used to encode and decode option position data into and from a single `uint256` token ID. This token ID is used in an ERC1155 contract to represent complex option positions on a Uniswap v3 pool.

The library provides methods to:

1.  **Encode (Pack) Data**: Combine various pieces of information about an option position, such as the Uniswap v3 pool address, asset type, option ratio, long/short status, token type, risk partner, strike price, and width, into a single `uint256` token ID.
    
2.  **Decode (Unpack) Data**: Extract the individual pieces of information from the token ID to understand the details of the option position.
    
3.  **Validation**: Ensure that the encoded option positions are valid according to certain rules, such as no gaps between active legs and mutual risk partners.
    
4.  **Helpers**: Provide utility functions to manipulate the token ID, such as flipping the long/short status of positions or counting the number of active legs.
    

&nbsp;

* * *

- [ ] **<ins>contracts/libraries</ins>**

1.  **CallbackLib.sol** : for verifying and decoding callbacks from Uniswap V3 pools.

core function:

The core function  is `validateCallback`     :

- It receives the address that initiated the callback (`sender`), the Uniswap V3 factory address (`factory`), and the claimed features of the pool (`features`).
    
- It calculates the expected address of the Uniswap pool using the `factory` address and the `features` of the pool, along with a known constant (`Constants.V3POOL_INIT_CODE_HASH`) that is specific to Uniswap V3 pools.
    
- It checks if the calculated address matches the `sender` address. If they match, the callback is considered valid. If not, the function throws an error, indicating the callback is not from a valid Uniswap V3 pool.
    

2.  **Constants.sol**: ontains a set of internal constants used in a project called Panoptic  
    3\. **Errors.sol** : Errors that contains custom error types for a smart contract

4: **FeesCalc.sol** , 5: **Math.sol** , 6: **PanopticMath.sol**, 7: **SafeTransferLib.sol**

calculates the swap/trading fees accumulated for a specific liquidity position (referred to as a "liquidity chunk") within a Uniswap V3 pool

* * *

- [ ] **Multicall.sol :** allows multiple function calls to be made within a single transaction

core function:

`multicall()`

Performs multiple calls on the inheritor contract in a single transaction and returns the data from each call.

- - It initializes an array `results` to store the results of each call.
        - It iterates through the array of calldata (`data`).
        - For each calldata, it uses `delegatecall` to execute the corresponding function call on the current contract.
        - If the function call is unsuccessful (reverts), it extracts and reverts with the revert reason.
        - If successful, it stores the result in the `results` array.
        - The loop continues until all function calls are processed.

* * *

list of functions that are explicitly declared public or payable

| 🌐Public | 💰Payable |
| :--- | :--- |
| 17  | 1   |

| External | Internal | Private | Pure | View |
| :--- | :--- | :--- | :--- | :--- |
| 10  | 110 | 3   | 67  | 9   |

* * *

## Test analysis

The audit scope of the contracts to be reviewed is 100%.

### Tests

Tests are clearly structured and cover most of the core functionality.

## Codebase Quality

- **Modular Design:** The code is organized into different contracts, each responsible for specific functionality. This makes the codebase easier to understand and maintain.
    
- **Clear Comments:** There are comments throughout the code that explain the purpose of functions, variables, and sections. This enhances readability and comprehension.
    
- **Events:** Events are used to provide transparency and enable easier tracking of important contract interactions.
    

&nbsp;

## Centralization Risk

No centralization Risk

## Gas Optimization

Panoptic Protocol is generally efficient in terms of gas optimizations, many generally accepted gas optimizations have been implemented, gas optimizations with minor effects are already mentioned in automatic finding, here is some more.

\[G-01\] you can save storage by reordering Holding struct fields .

\[G-02\] Use constants instead of type(uintx).max

\[G-03\] Use assembly for loops

G-04\]use Mappings Instead of Arrays

\[G-05\] Mappings used within a function more than once should be cached to save gas

\[G-06\] The use of assembly code to load the left and right siblings into memory is more gas-efficient

\[G-07\] Use assembly to perform efficient back-to-back calls

\[G-08\] Do not calculate constants

\[G-09\] Amounts should be checked for 0 before calling a transfer

\[G-10\] IF’s/require() statements that check input arguments should be at the top of the function

\[G‑11\] Save gas by preventing zero amount in  and burn()

\[G‑12\] Using `bool`s for storage incurs overhead

## **Documentation**

The documentation provided for the Panoptic contract is quite comprehensive and detailed in terms of explaining its functionality, parameter usage, purpose, and overall architecture.

## Time Spent on the Audit

- 27 hours

&nbsp;

### Time spent:
27 hours