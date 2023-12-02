https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol

1)Loop Optimization and State Updates:
Reducing Operations within the Loop: Consider using logic that avoids the internal for loop inside registerTokenTransfer. Calculations can be performed outside the loop to decrease the number of operations inside the iteration.
Grouping State Updates: Evaluate the possibility of grouping state updates to minimize the number of storage write operations. Reducing storage writes can potentially decrease gas costs.
 The optimized version of the registerTokenTransfer function focusing on reducing operations within the loop and grouping state updates outside the loop
function registerTokenTransfer(address from, address to, uint256 id, uint256 amount) internal {
    // Extract univ3pool from the poolId map to Uniswap Pool
    IUniswapV3Pool univ3pool = s_poolContext[id.validate()].pool;
    uint256 numLegs = id.countLegs();
    // Variables to update state outside the loop
    bytes32[] memory positionKeys_from = new bytes32[](numLegs);
    bytes32[] memory positionKeys_to = new bytes32[](numLegs);
    uint256[] memory fromLiquidityUpdates = new uint256[](numLegs);
    uint256[] memory toLiquidityUpdates = new uint256[](numLegs);
    int256[] memory fromBaseUpdates = new int256[](numLegs);
    int256[] memory toBaseUpdates = new int256[](numLegs);
    uint256 totalFromLiquidity;
    int256 totalFromBase;
    for (uint256 leg = 0; leg < numLegs; leg++) {
        // ... (rest of the code to calculate values)

        // Calculation of values before state update

        positionKeys_from[leg] = keccak256(
            abi.encodePacked(
                address(univ3pool),
                from,
                id.tokenType(leg),
                liquidityChunk.tickLower(),
                liquidityChunk.tickUpper()
            )
        );
        positionKeys_to[leg] = keccak256(
            abi.encodePacked(
                address(univ3pool),
                to,
                id.tokenType(leg),
                liquidityChunk.tickLower(),
                liquidityChunk.tickUpper()
            )
        );
        fromLiquidityUpdates[leg] = s_accountLiquidity[positionKeys_from[leg]];
        toLiquidityUpdates[leg] = liquidityChunk.liquidity();
        fromBaseUpdates[leg] = s_accountFeesBase[positionKeys_from[leg]];
        toBaseUpdates[leg] = 0;

        totalFromLiquidity += fromLiquidityUpdates[leg];
        totalFromBase += fromBaseUpdates[leg];
    }
    // Update state outside the loop
    for (uint256 i = 0; i < numLegs; i++) {
        s_accountLiquidity[positionKeys_to[i]] = toLiquidityUpdates[i];
        s_accountLiquidity[positionKeys_from[i]] = 0;
        s_accountFeesBase[positionKeys_to[i]] = toBaseUpdates[i];
        s_accountFeesBase[positionKeys_from[i]] = 0;
    }
    // Perform other operations with the calculated values
}
This optimized approach calculates necessary values before state updates and then performs grouped state updates outside the loop. This reduces the number of storage write operations within the loop, potentially decreasing gas costs. 

2)Loop Optimization: The _createPositionInAMM function has a for loop that iterates through legs. To optimize, ensure calculations and variable initializations are moved outside the loop where possible. This reduces redundant computations.
State Updates and Gas Optimization: Consolidating state updates and minimizing storage writes can reduce gas costs. Consider batching state updates where applicable to minimize write operations.
Optimizing the state updates within the loop in the _createPositionInAMM function
function _createPositionInAMM(
    IUniswapV3Pool univ3pool,
    uint256 tokenId,
    uint128 positionSize,
    bool isBurn
) internal returns (int256 totalMoved, int256 totalCollected, int256 itmAmounts) {
    uint256 amount0;
    uint256 amount1;

    uint256 numLegs = tokenId.countLegs();
    IUniswapV3Pool _univ3pool = univ3pool;
    uint256 _tokenId = tokenId;
    uint128 _positionSize = positionSize;
    bool _isBurn = isBurn;
    uint256 liquidityChunk;
    uint128 chunkLiquidity;
    uint256 isLong;
    uint256 currentLiquidity;
    bytes32 positionKey;
    uint128 updatedLiquidity;
    uint128 removedLiquidity;
    uint256 _moved;
    uint256 _itmAmounts;
    uint256 _totalCollected;

    for (uint256 leg = 0; leg < numLegs; leg++) {
        unchecked {
            uint256 _leg = _isBurn ? numLegs - leg - 1 : leg;

            liquidityChunk = PanopticMath.getLiquidityChunk(
                _tokenId,
                _leg,
                _positionSize,
                _univ3pool.tickSpacing()
            );
            (_moved, _itmAmounts, _totalCollected) = _createLegInAMM(
                _univ3pool,
                _tokenId,
                _leg,
                liquidityChunk,
                _isBurn
            );
            amount0 += Math.getAmount0ForLiquidity(liquidityChunk);
            amount1 += Math.getAmount1ForLiquidity(liquidityChunk);
        }
        totalMoved += _moved;
        itmAmounts += _itmAmounts;
        totalCollected += _totalCollected;
    }
    // Check if the position size exceeds the allowed maximum
    if (amount0 > uint128(type(int128).max) || amount1 > uint128(type(int128).max)) {
        revert Errors.PositionTooLarge();
    }
    return (totalMoved, totalCollected, itmAmounts);
}
function _createLegInAMM(
    IUniswapV3Pool _univ3pool,
    uint256 _tokenId,
    uint256 _leg,
    uint256 _liquidityChunk,
    bool _isBurn
) internal returns (int256 _moved, int256 _itmAmounts, int256 _totalCollected) {
    // ... existing function code remains unchanged
    // Ensure to keep the existing logic within the function
}
This refactoring aims to optimize the _createPositionInAMM function by moving unchanged computations outside the loop and preserving the existing logic within the _createLegInAMM function.

3)Optimizing _mintLiquidity and _burnLiquidity functions can involve several steps to minimize gas usage
_mintLiquidity Optimization:
function _mintLiquidity(
    uint256 liquidityChunk,
    IUniswapV3Pool univ3pool
) internal returns (int256 movedAmounts) {
    bytes memory mintdata = abi.encode(
        CallbackLib.CallbackData({
            poolFeatures: CallbackLib.PoolFeatures({
                token0: univ3pool.token0(),
                token1: univ3pool.token1(),
                fee: univ3pool.fee()
            }),
            payer: msg.sender
        })
    );
    (uint256 amount0, uint256 amount1) = univ3pool.mint(
        address(this),
        liquidityChunk.tickLower(),
        liquidityChunk.tickUpper(),
        liquidityChunk.liquidity(),
        mintdata
    );
    movedAmounts = int256(0).toRightSlot(int128(int256(amount0))).toLeftSlot(
        int128(int256(amount1))
    );
}

_burnLiquidity Optimization:
function _burnLiquidity(
    uint256 liquidityChunk,
    IUniswapV3Pool univ3pool
) internal returns (int256 movedAmounts) {
    (uint256 amount0, uint256 amount1) = univ3pool.burn(
        liquidityChunk.tickLower(),
        liquidityChunk.tickUpper(),
        liquidityChunk.liquidity()
    );
    unchecked {
        movedAmounts = int256(0).toRightSlot(-int128(int256(amount0))).toLeftSlot(
            -int128(int256(amount1))
        );
    }
}
These optimizations aim to minimize gas usage by refactoring the code and removing unnecessary operations where possible.

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol

1)Storage Optimization: Consider using uint128 instead of uint256 if the number of tokens is limited to 128 bits. This can save storage gas costs.
For example in Line 62: mapping(address account => mapping(uint256 tokenId => uint256 balance)) public balanceOf;
this could be refactored to:
mapping(address account => mapping(uint128 tokenId => uint 128 balance)) public balanceOf;

2)Line 97: Use Require inestead of if for gas saving.
3)Line 135: Use Rquire inestead of if for gas saving.

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol

1)Consider using bitwise operations instead of mathematical operations where possible. Bitwise operations are often more gas-efficient.
Instead of:
function add(uint256 x, uint256 y) internal pure returns (uint256 z) {
    unchecked {
        z = x + y;
        if (z < x || (uint128(z) < uint128(x))) revert Errors.UnderOverFlow();
    }
}
We can use bitwise operations for checks:
function add(uint256 x, uint256 y) internal pure returns (uint256 z) {
    unchecked {
        z = x + y;
        if ((z < x) || ((z & uint256(uint128(-1))) < (x & uint256(uint128(-1))))) revert Errors.UnderOverFlow();
    }
}
2)Optimize Casting: For casting, consider using bitwise operations to ensure efficiency and handle edge cases.
Instead of:
function toInt128(int256 self) internal pure returns (int128 selfAsInt128) {
    if (!((selfAsInt128 = int128(self)) == self)) revert Errors.CastingError();
}
We can use bitwise operations:
function toInt128(int256 self) internal pure returns (int128 selfAsInt128) {
    selfAsInt128 = int128(self);
    if (int256(selfAsInt128) != self) revert Errors.CastingError();
}
By incorporating bitwise operations where applicable and optimizing arithmetic calculations, the contract can become more gas-efficient and potentially perform better in terms of computation.
