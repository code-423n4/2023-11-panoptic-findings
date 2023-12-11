# Low Risk and Non-Critical Issues

| Number |                                                                                              |
| :----: | :------------------------------------------------------------------------------------------- |
| [L-01] | There is no restriction for SemiFungiblePositionManager::initializeAMMPool function          |
| [L-02] | Potential for Abusing Fee Accumulation Logic                                                 |
| [L-03] | Missing Contract Existence Checks                                                            |
| [L-04] | Incorrect Token Amount Calculations                                                          |
| [L-05] | No checks for duplicate mappings ERC1155Minimal::balanceOf, ERC1155Minimal::isApprovedForAll |
| [L-06] | Approval bypass vulnerability in ERC1155 token transfer functions                            |
| [L-07] | Misaligned Ticks and Liquidity Issue                                                         |
| [L-08] | Incorrect Rounding FeesCalc::calculateAMMSwapFeesLiquidityChunk function                     |
| [N-09] | Pool Initialized Event Missing Sender Information                                            |
| [N-10] | RegisterTokenTransfer Loops Over Unbounded Input Array                                       |

## [L-01] There is no restriction for SemiFungiblePositionManager::initializeAMMPool function

Anyone can call the initializeAMMPool function to add new pools to the protocol, even if they are manipulating pool parameters. This lack of authorization for important administration functions could allow bad actors to undermine the intended operation of the protocol.

The initializeAMMPool function does not check the caller's permissions:

```solidity
File: contracts/SemiFungiblePositionManager.sol

351    function initializeAMMPool(address token0, address token1, uint24 fee) external {
352        // compute the address of the Uniswap v3 pool for the given token0, token1, and fee tier
353        address univ3pool = FACTORY.getPool(token0, token1, fee);
354
355        // reverts if the Uni v3 pool has not been initialized
356        if (address(univ3pool) == address(0)) revert Errors.UniswapPoolNotInitialized();
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L351-L397

## [L-02] Potential for Abusing Fee Accumulation Logic

Without proper protection mechanisms, traders could potentially "game" the system by rapidly depositing and withdrawing liquidity to manipulate how fees accumulate over time. This could enable strategies to skew premium calculations through abnormal trading activity.

Currently nothing prevents a trader from:

- Depositing a large amount of liquidity
- Withdrawing it a short time later
- Repeating this process rapidly

This could artificially inflate the total trading volume and collected fees for a short time period. When withdrawn, the trader could then argue they are owed inflated premiums based on short-lived abnormal market activity they directly influenced.

Introduce rate limiting on deposits/withdrawals from the same address within a time period.

## [L-03] Missing Contract Existence Checks

Without existence checks, low-level calls may succeed even if the target contract is removed, resulting in failed transactions without errors. This could lead to lost or stolen funds if users are unaware the transfer did not complete as expected.

In SafeTransferLib.sol:

solidity

If the token contract at token is removed, this call will not revert as expected.

```solidity
File: contracts/libraries/SafeTransferLib.sol

//@audit Call token contract without extcodesize check
36                call(gas(), token, 0, p, 100, 0, 32)
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/SafeTransferLib.sol#L19-L38

Add extcodesize check before calls:

```solidity
if (extcodesize(token) == 0) {
  revert("Token contract not found");
}

36                call(gas(), token, 0, p, 100, 0, 32)
```

Consider using a library like OpenZeppelin's Address.sol to encapsulate existence checks.

## [L-04] Incorrect Token Amount Calculations

The swapping and netting logic aims to properly account for token amounts when minting/burning ITM positions. However, bugs could result in traders receiving incorrect token balances, leading to financial losses.

In the function `_validateAndForwardToAMM()`:

The netting swap amount calculation is complex, splitting signed token amounts across multiple operations.
Edge cases around price changes are not fully tested.

```solidity
File: contracts/SemiFungiblePositionManager.sol

function _validateAndForwardToAMM(
        uint256 tokenId,
        uint128 positionSize,
        int24 tickLimitLow,
        int24 tickLimitHigh,
        bool isBurn
    ) internal returns (int256 totalCollectedFromAMM, int256 totalMoved, int24 newTick) {
        // Reverts if positionSize is 0 and user did not own the position before minting/burning
        if (positionSize == 0) revert Errors.OptionsBalanceZero();

        /// @dev the flipToBurnToken() function flips the isLong bits
        if (isBurn) {
            tokenId = tokenId.flipToBurnToken();
        }

        // Validate tokenId
        // Extract univ3pool from the poolId map to Uniswap Pool
        IUniswapV3Pool univ3pool = s_poolContext[tokenId.validate()].pool;

        // Revert if the pool not been previously initialized
        if (univ3pool == IUniswapV3Pool(address(0))) revert Errors.UniswapPoolNotInitialized();

        bool swapAtMint;
        {
            if (tickLimitLow > tickLimitHigh) {
                swapAtMint = true;
                (tickLimitLow, tickLimitHigh) = (tickLimitHigh, tickLimitLow);
            }
        }
        // initialize some variables returned by the _createPositionInAMM function
        int256 itmAmounts;

        {
            // calls a function that loops through each leg of tokenId and mints/burns liquidity in Uni v3 pool
            (totalMoved, totalCollectedFromAMM, itmAmounts) = _createPositionInAMM(
                univ3pool,
                tokenId,
                positionSize,
                isBurn
            );
        }

        // if the in-the-money amount is not zero (i.e. positions were minted ITM) and the user did provide tick limits LOW > HIGH, then swap necessary amounts
        if ((itmAmounts != 0) && (swapAtMint)) {
            totalMoved = swapInAMM(univ3pool, itmAmounts).add(totalMoved);
        }

        // Get the current tick of the Uniswap pool, check slippage
        (, newTick, , , , , ) = univ3pool.slot0();

        if ((newTick >= tickLimitHigh) || (newTick <= tickLimitLow)) revert Errors.PriceBoundFail();

        return (totalCollectedFromAMM, totalMoved, newTick);
    }
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L663-L716

This introduces opportunity for off-by-one errors or missed conditions.

## [L-05] No checks for duplicate mappings ERC1155Minimal::balanceOf, ERC1155Minimal::isApprovedForAll

The balanceOf and isApprovedForAll mappings allow duplicate mappings to be added without validation. This could overwrite existing balances or approvals in unexpected ways. It breaks the assumption that mappings are keyed by their parameters, which risks logical errors.

```solidity
File: contracts/tokens/ERC1155Minimal.sol

62    mapping(address account => mapping(uint256 tokenId => uint256 balance)) public balanceOf;

67    mapping(address owner => mapping(address operator => bool approvedForAll))
68        public isApprovedForAll;
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L62-L68

Recommended: Check for existing mappings before updates

## [L-06] Approval bypass vulnerability in ERC1155 token transfer functions

The ERC1155Minimal contract's token transfer functions safeTransferFrom and safeBatchTransferFrom only check if the sender is the token owner or an approved operator, but do not validate that the amount being transferred does not exceed the approved amount.

This allows an approved operator to transfer more tokens than were approved, bypassing the intended permission model.

```solidity
File: tokens/ERC1155Minimal.sol

    function safeTransferFrom(
        address from,
        address to,
        uint256 id,
        uint256 amount,
        bytes calldata data
    ) public {
        if (!(msg.sender == from || isApprovedForAll[from][msg.sender])) revert NotAuthorized();

        balanceOf[from][id] -= amount;

        // balance will never overflow
        unchecked {
            balanceOf[to][id] += amount;
        }

        afterTokenTransfer(from, to, id, amount);

        emit TransferSingle(msg.sender, from, to, id, amount);

        if (to.code.length != 0) {
            if (
                ERC1155Holder(to).onERC1155Received(msg.sender, from, id, amount, data) !=
                ERC1155Holder.onERC1155Received.selector
            ) {
                revert UnsafeRecipient();
            }
        }
    }


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

        for (uint256 i = 0; i < ids.length; ) {
            id = ids[i];
            amount = amounts[i];

            balanceOf[from][id] -= amount;

            // balance will never overflow
            unchecked {
                balanceOf[to][id] += amount;
            }

            // An array can't have a total length
            // larger than the max uint256 value.
            unchecked {
                ++i;
            }
        }

        afterTokenTransfer(from, to, ids, amounts);

        emit TransferBatch(msg.sender, from, to, ids, amounts);

        if (to.code.length != 0) {
            if (
                ERC1155Holder(to).onERC1155BatchReceived(msg.sender, from, ids, amounts, data) !=
                ERC1155Holder.onERC1155BatchReceived.selector
            ) {
                revert UnsafeRecipient();
            }
        }
    }

```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L90-L118

## [L-07] Misaligned Ticks and Liquidity Issue

The FeesCalc.sol reads the liquidity chunk ticks (lower/upper) separately from the starting liquidity amount. This opens up a reentrancy/race condition bug where the ticks and liquidity could become misaligned if another contract is able to call in between.

- Attacker calls calculateAMMSwapFeesLiquidityChunk(), passing a valid liquidityChunk
- calculateAMMSwapFeesLiquidityChunk() reads ticks from chunk
- Attacker reverts chunk ticks before returning
- calculateAMMSwapFeesLiquidityChunk() proceeds to calculate fees using invalid ticks

Incorrect fee amounts could be calculated if ticks and liquidity are out of sync. This could lead to losses for the pool or position holders.

```solidity
File:  contracts/libraries/FeesCalc.sol

  function calculateAMMSwapFeesLiquidityChunk(
        IUniswapV3Pool univ3pool,
        int24 currentTick,
        uint128 startingLiquidity,
        uint256 liquidityChunk
    ) public view returns (int256 feesEachToken) {
        // extract the amount of AMM fees collected within the liquidity chunk`
        // note: the fee variables are *per unit of liquidity*; so more "rate" variables
        (
            uint256 ammFeesPerLiqToken0X128,
            uint256 ammFeesPerLiqToken1X128
        ) = _getAMMSwapFeesPerLiquidityCollected(
                univ3pool,
                currentTick,
                liquidityChunk.tickLower(),
                liquidityChunk.tickUpper()
            );

        // Use the fee growth (rate) variable to compute the absolute fees accumulated within the chunk:
        //   ammFeesToken0X128 * liquidity / (2**128)
        // to store the (absolute) fees as int128:
        feesEachToken = feesEachToken
            .toRightSlot(int128(int256(Math.mulDiv128(ammFeesPerLiqToken0X128, startingLiquidity))))
            .toLeftSlot(int128(int256(Math.mulDiv128(ammFeesPerLiqToken1X128, startingLiquidity))));
    }
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/FeesCalc.sol#L54-L78

Read all required data atomically from a single source

## [L-08] Incorrect Rounding FeesCalc::calculateAMMSwapFeesLiquidityChunk function

Integers are converted between types like 128-bit and 256-bit without specifying a rounding mode. This could lead to small inaccuracies accumulating over many conversions.

```solidity
File: contracts/libraries/FeesCalc.sol

        feesEachToken = feesEachToken
            .toRightSlot(int128(int256(Math.mulDiv128(ammFeesPerLiqToken0X128, startingLiquidity))))
            .toLeftSlot(int128(int256(Math.mulDiv128(ammFeesPerLiqToken1X128, startingLiquidity))));
    }
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/FeesCalc.sol#L75-L78

## [N-09] Pool Initialized Event Missing Sender Information

The PoolInitialized event emitted in the initializeAMMPool function does not include the address of the sender/initiator of the pool initialization.

Not including the msg.sender in this event makes it difficult to track which user/contract triggered the pool initialization. This reduces the usefulness of the event for monitoring and analytics purposes.

```solidity
File: contracts/SemiFungiblePositionManager.sol

    function initializeAMMPool(address token0, address token1, uint24 fee) external {
        // compute the address of the Uniswap v3 pool for the given token0, token1, and fee tier
        address univ3pool = FACTORY.getPool(token0, token1, fee);

        // reverts if the Uni v3 pool has not been initialized
        if (address(univ3pool) == address(0)) revert Errors.UniswapPoolNotInitialized();

        // return if the pool has already been initialized in SFPM
        // @dev pools can be initialized from a Panoptic pool or by calling initializeAMMPool directly, reverting
        // would prevent any PanopticPool from being deployed
        // @dev some pools may not be deployable if the poolId has a collision (since we take only 8 bytes)
        // if poolId == 0, we have a bit on the left set if it was initialized, so this will still return properly
        if (s_AddrToPoolIdData[univ3pool] != 0) return;

        // Set the base poolId as last 8 bytes of the address (the first 16 hex characters)
        // @dev in the unlikely case that there is a collision between the first 8 bytes of two different Uni v3 pools
        // @dev increase the poolId by a pseudo-random number
        uint64 poolId = PanopticMath.getPoolId(univ3pool);

        while (address(s_poolContext[poolId].pool) != address(0)) {
            poolId = PanopticMath.getFinalPoolId(poolId, token0, token1, fee);
        }
        // store the poolId => UniswapV3Pool information in a mapping
        // `locked` can be initialized to false because the pool address makes the slot nonzero
        s_poolContext[poolId] = PoolAddressAndLock({
            pool: IUniswapV3Pool(univ3pool),
            locked: false
        });

        // store the UniswapV3Pool => poolId information in a mapping
        // add a bit on the end to indicate that the pool is initialized
        // (this is for the case that poolId == 0, so we can make a distinction between zero and uninitialized)
        unchecked {
            s_AddrToPoolIdData[univ3pool] = uint256(poolId) + 2 ** 255;
        }
        emit PoolInitialized(univ3pool);

        return;

        // this disables `memoryguard` when compiling this contract via IR
        // it is classed as a potentially unsafe assembly block by the compiler, but is in fact safe
        // we need this because enabling `memoryguard` and therefore StackLimitEvader increases the size of the contract significantly beyond the size limit
        assembly {
            mstore(0, 0xFA20F71C)
        }
    }
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L351-L396

```diff
event PoolInitialized(
   address indexed pool,
+   address indexed sender
);



-        emit PoolInitialized(univ3pool);
+        emit PoolInitialized(univ3pool, msg.sender);
```

## [N-10] RegisterTokenTransfer Loops Over Unbounded Input Array

The afterTokenTransfer function iterates over the ids and amounts arrays passed in without bounding the length. This could allow gas exhaustion attacks by supplying very long input arrays.

By not validating the array length, a malicious actor could craft transactions with extremely long input arrays, wasting gas as the loops continue processing. This degrades the user experience.

```solidity
File: contracts/SemiFungiblePositionManager.sol

    function afterTokenTransfer(
        address from,
        address to,
        uint256[] memory ids,
        uint256[] memory amounts
    ) internal override {
        for (uint256 i = 0; i < ids.length; ) {
            registerTokenTransfer(from, to, ids[i], amounts[i]);
            unchecked {
                ++i;
            }
        }
    }
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L544-L556
