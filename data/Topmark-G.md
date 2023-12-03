### Report 1:
Repetition of "amount0Delta > 0" condition validation takes up gas, a more efficient way is provided in the code below
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L452-L457
```solidity
 function uniswapV3SwapCallback(
        int256 amount0Delta,
        int256 amount1Delta,
        bytes calldata data
    ) external {
        ...
---        // Extract the address of the token to be sent (amount0 -> token0, amount1 -> token1)
---        address token = amount0Delta > 0
---            ? address(decoded.poolFeatures.token0)
---            : address(decoded.poolFeatures.token1);

---        // Transform the amount to pay to uint256 (take positive one from amount0 and amount1)
---        uint256 amountToPay = amount0Delta > 0 ? uint256(amount0Delta) : uint256(amount1Delta);
+++   if ( amount0Delta > 0 ){
+++        address token = address(decoded.poolFeatures.token0);
+++        uint256 amountToPay = uint256(amount0Delta);
+++     }
+++     else {
+++      address token = address(decoded.poolFeatures.token1);
+++      uint256 amountToPay = uint256(amount1Delta);
+++     }
        // Pay the required token from the payer to the caller of this contract
        SafeTransferLib.safeTransferFrom(token, decoded.payer, msg.sender, amountToPay);
    }
```