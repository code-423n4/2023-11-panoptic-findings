# Low Risk

| Number |                                                                                              |
| :----: | :------------------------------------------------------------------------------------------- |
| [L-01] | There is no restriction for SemiFungiblePositionManager::initializeAMMPool function          |
| [L-02] | Potential for Abusing Fee Accumulation Logic                                                 |
| [L-03] | Missing Contract Existence Checks                                                            |
| [L-04] | Incorrect Token Amount Calculations                                                          |
| [L-05] | No checks for duplicate mappings ERC1155Minimal::balanceOf, ERC1155Minimal::isApprovedForAll |

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
