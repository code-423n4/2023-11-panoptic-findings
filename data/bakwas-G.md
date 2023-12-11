1. Inefficient Gas Usage During Burns

Inefficient gas usage could result in higher transaction costs for users.

The function may perform unnecessary computations or state updates that consume extra gas.

Optimize the function to minimize gas usage. 
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L476C22-L476C22

2. Suboptimal Gas Usage
Inefficient gas usage could lead to higher costs for users when minting positions
The function may perform redundant operations or state updates that consume extra gas
Review and optimize the function's logic to reduce gas consumption.

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L510

3. Incorrect Slippage Limit Handling

Improper handling of slippage limits could result in transactions failing unexpectedly or not providing the intended price protection.

The function assumes that slippage limits are always provided in a specific order and does not handle cases where they might be reversed or equal.
Implement robust handling of slippage limit parameters to ensure they are used correctly.

require(slippageTickLimitLow < slippageTickLimitHigh, 'Invalid slippage limits');

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L663

4. Inefficient Loop

Inefficient Handling of Batch Transfers

May lead to increased gas costs when transferring multiple tokens due to loop inefficiencies.

The function iterates over all transferred token IDs without optimizing for batch transfers.

Optimize the loop to handle batch transfers more efficiently.



5. 	Inefficient Gas Usage

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L743
The function may use more gas than necessary, leading to higher costs for users.

The function could be optimized to reduce gas consumption by minimizing state changes and external calls.

6. Suboptimal Gas Usage in Loop
Inefficient looping over position legs could lead to higher than necessary gas costs.

The function loops over each leg of the position without optimizing for gas usage.

Optimize the loop to reduce gas consumption, such as by minimizing state reads and writes within the loop.

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L860

7. Gas Optimization	Inefficient Gas Usage During Fee Collection

The function may perform unnecessary operations or state updates that consume extra gas.

Optimize the function to reduce gas consumption, such as by minimizing state updates and reusing computed values.

Minimizing state updates and reusing computed values can significantly optimize gas usage in your code. Here are some areas where these optimizations could be applied:

Computed Value Reuse: Identify any calculations or values that are reused within the function. Store these values in local variables to prevent recomputation. For instance, if the same value is used multiple times in arithmetic operations, compute it once and reuse the result.

Batch State Updates: Instead of updating state variables multiple times within the function, batch these updates where possible. If several state variables need to be changed based on the same conditions or calculations, perform these updates together to minimize the number of state-changing operations.

Lazy Evaluation: Evaluate conditions and perform state updates only if necessary. If certain conditions don’t require state changes or if they’re not crucial for the function's logic, avoid executing unnecessary state updates.

Suggestion: 

function _collectAndWritePositionData(
    uint256 liquidityChunk,
    IUniswapV3Pool univ3pool,
    uint256 currentLiquidity,
    bytes32 positionKey,
    int256 movedInLeg,
    uint256 isLong
) internal returns (int256 collectedOut) {
    uint128 startingLiquidity = currentLiquidity.rightSlot();
    int256 amountToCollect = _getFeesBase(univ3pool, startingLiquidity, liquidityChunk).sub(
        s_accountFeesBase[positionKey]
    );

    if (isLong == 1) {
        amountToCollect = amountToCollect.sub(movedInLeg);
    }

    if (amountToCollect != 0) {
        // Compute fee collection details
        (uint128 receivedAmount0, uint128 receivedAmount1) = univ3pool.collect(
            msg.sender,
            liquidityChunk.tickLower(),
            liquidityChunk.tickUpper(),
            uint128(amountToCollect.rightSlot()),
            uint128(amountToCollect.leftSlot())
        );

        uint128 collected0;
        uint128 collected1;
        unchecked {
            collected0 = movedInLeg.rightSlot() < 0
                ? receivedAmount0 - uint128(-movedInLeg.rightSlot())
                : receivedAmount0;
            collected1 = movedInLeg.leftSlot() < 0
                ? receivedAmount1 - uint128(-movedInLeg.leftSlot())
                : receivedAmount1;
        }

        // Optimize state updates by batching
        int256 deltaCollected0 = int256(0).toRightSlot(collected0);
        int256 deltaCollected1 = int256(0).toLeftSlot(collected1);

        // Batch state updates
        int256 deltaCollected = deltaCollected0.toLeftSlot(deltaCollected1);
        collectedOut = deltaCollected;

        _updateStoredPremia(positionKey, currentLiquidity, deltaCollected);
    }
}
