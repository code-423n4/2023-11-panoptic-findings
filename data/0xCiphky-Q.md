## Title:  The ERC1155Minimal contract does not follow ERC1155 spec

## **Severity:**

- Low Risk

## **Relevant GitHub Links:**

- https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/tokens/ERC1155Minimal.sol#L90
- https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/tokens/ERC1155Minimal.sol#L128

## **Vulnerability Details:**

The ERC155Minimal contract does not fully comply with the EIP1155 standard, specifically it misses the rules listed below.

***safeTransferFrom rules:***

- MUST revert if `_to` is the zero address.

```
function safeTransferFrom(address from, address to, uint256 id, uint256 amount, bytes calldata data) public {
        if (!(msg.sender == from || isApprovedForAll[from][msg.sender])) revert NotAuthorized();

        // @audit-info doesn't check if to is the zero address

        balanceOf[from][id] -= amount;

        // balance will never overflow
        unchecked {
            balanceOf[to][id] += amount;
        }

        ...
    }
```

***safeBatchTransferFrom rules:*** 

- MUST revert if `_to` is the zero address.
- MUST revert if length of `_ids` is not the same as length of `_values`.

```solidity
function safeBatchTransferFrom(
        address from,
        address to,
        uint256[] calldata ids,
        uint256[] calldata amounts,
        bytes calldata data
    ) public virtual {
        if (!(msg.sender == from || isApprovedForAll[from][msg.sender])) revert NotAuthorized();

        // Storing these outside the loop saves ~15 gas per iteration.
        uint256 id;
        uint256 amount;

	// @audit-info doesn't check if to is the zero address

        // @audit-info does not revert if ids and amounts are of unequal length, per eip specifications

        for (uint256 i = 0; i < ids.length;) {
            id = ids[i];
            amount = amounts[i];

            ...
        }

        ...
    }
```

## **Recommendation:**

Add the following checks:

```
function safeTransferFrom(address from, address to, uint256 id, uint256 amount, bytes calldata data) public {
        if (!(msg.sender == from || isApprovedForAll[from][msg.sender])) revert NotAuthorized();

        // add here
	if (to == address(0)) revert ("...");

        balanceOf[from][id] -= amount;

        // balance will never overflow
        unchecked {
            balanceOf[to][id] += amount;
        }

        ...
    }
```

```solidity
function safeBatchTransferFrom(
        address from,
        address to,
        uint256[] calldata ids,
        uint256[] calldata amounts,
        bytes calldata data
    ) public virtual {
        if (!(msg.sender == from || isApprovedForAll[from][msg.sender])) revert NotAuthorized();

        // Storing these outside the loop saves ~15 gas per iteration.
        uint256 id;
        uint256 amount;

	// add here
	if (to == address(0)) revert ("...");

        // add here
	if (ids.length != amounts.length) revert ("...");

        for (uint256 i = 0; i < ids.length;) {
            id = ids[i];
            amount = amounts[i];

            ...
            unchecked {
                ++i;
            }
        }

        ...
    }
```

## Title:  ****Transfer Limitations with Multiple Legs Sharing Same Position Key****

## **Severity:**

- Low Risk

## **Relevant GitHub Links:**
- https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L578

## **Summary:**

The protocol allows users to mint ERC1155 tokens representing positions in Uniswap V3 pools, allowing up to four legs per token. These tokens are transferable under specific conditions, primarily requiring that the entire balance of a given tokenId is transferred, and the recipient does not already hold a position within that token.

A constraint arises during the transfer process. The **`registerTokenTransfer`** function, which manages the transfer logistics, checks if the total liquidity of the current leg equals the user’s total liquidity for that position. However, the position key, used to track liquidity, is not unique to each leg. Consequently, users who have multiple legs associated with the same position key cannot successfully transfer their positions.

```solidity
function registerTokenTransfer(address from, address to, uint256 id, uint256 amount) internal {
        // Extract univ3pool from the poolId map to Uniswap Pool
        IUniswapV3Pool univ3pool = s_poolContext[id.validate()].pool;

        uint256 numLegs = id.countLegs();
        for (uint256 leg = 0; leg < numLegs;) {
            // for this leg index: extract the liquidity chunk: a 256bit word containing the liquidity amount and upper/lower tick
            // @dev see `contracts/types/LiquidityChunk.sol`
            uint256 liquidityChunk = PanopticMath.getLiquidityChunk(id, leg, uint128(amount), univ3pool.tickSpacing());

            //construct the positionKey for the from and to addresses
            bytes32 positionKey_from = keccak256(
                abi.encodePacked(
                    address(univ3pool), from, id.tokenType(leg), liquidityChunk.tickLower(), liquidityChunk.tickUpper()
                )
            );
            ...

            // Revert if not all balance is transferred
            uint256 fromLiq = s_accountLiquidity[positionKey_from];
            if (fromLiq.rightSlot() != liquidityChunk.liquidity()) revert Errors.TransferFailed();

            ...
            }
        }
    }
```

This issue, while not leading to fund loss or severe security risks, can inconvenience users and restrict the fungibility of these ERC1155 tokens. It particularly affects users who wish to transfer positions but have created multiple legs with the same position key.

## **Recommendation:**

Clearly document to users about the specific conditions under which token transfers are possible, especially concerning the creation of multiple legs for the same position.

## Title:  ****Unrestricted Initialization of Uniswap Pools in the Protocol****

## **Severity:**

- Low Risk

## **Relevant GitHub Links:**
- https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L351

## **Summary:**

The protocol has specific guidelines regarding the pools that are permissible for use. However, the **`initializeAMMPool`** function, responsible for setting up Uniswap V3 pools within the protocol, does not enforce these guidelines. Specifically, it allows the initialization of any valid Uniswap pool, even those explicitly disallowed by the protocol's documentation.

“**Very large quantities of tokens are not supported. It should be assumed that for any given pool, the cumulative amount of tokens that enter the system (associated with that pool, through adding liquidity, collection, etc.) will not exceed 2^127 - 1.”**

“**Pools with a tick spacing of 1 are not currently supported. For the purposes of this audit, the only tick spacings that are supported are 10, 60, and 200”**

According to the protocol's documentation, pools with an very large token quantity (exceeding 2^127 - 1) and those with a tick spacing of 1 are not supported. Despite this, the **`initializeAMMPool`** function lacks mechanisms to filter out these disallowed pools, which could potentially cause underflow/overflow issues or precision loss errors in the protocol. Moreover, these pools have not undergone thorough testing, potentially exposing users to unanticipated risks.

```
function initializeAMMPool(address token0, address token1, uint24 fee) external {
        // compute the address of the Uniswap v3 pool for the given token0, token1, and fee tier
        address univ3pool = FACTORY.getPool(token0, token1, fee);

        // reverts if the Uni v3 pool has not been initialized
        if (address(univ3pool) == address(0)) revert Errors.UniswapPoolNotInitialized();

        ...
        if (s_AddrToPoolIdData[univ3pool] != 0) return;

        ...
        uint64 poolId = PanopticMath.getPoolId(univ3pool);

        while (address(s_poolContext[poolId].pool) != address(0)) {
            poolId = PanopticMath.getFinalPoolId(poolId, token0, token1, fee);
        }
        
    }
```

The absence of these restrictions in the initialization function presents a risk, as users might inadvertently interact with unsupported pools maliciously initialized, assuming them to be valid within the protocol.

## **Recommendation:**

Consider implementing a whitelist mechanism for pools, allowing only those pools that have been vetted and approved by the protocol administrators.