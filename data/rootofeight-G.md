Using `calldata` instead of `memory` for read-only arguments in `external` functions saves gas

when a function uses an array and is called from outside the contract, using abi.decode() to copy data into memory can cost a lot of gas due to the loop needed for each index. However, if you pass the array directly through calldata, it bypasses this loop and saves gas.

Even if an interface says a function uses memory, the contract implementing it can still use calldata instead.

If you pass the array through internal functions and modify it, using calldata in the external function with modifiers is still more gas-efficient. This is because modifiers might prevent those internal functions from being called, and using calldata avoids unnecessary memory copying.

Also, remember that structs have the same gas cost as an array with one element.

In simpler terms, using calldata directly can save gas when dealing with arrays in functions called externally.

```
Source : contracts/SemiFungiblePositionManager.sol

    function uniswapV3SwapCallback(
        int256 amount0Delta,
        int256 amount1Delta,
        bytes calldata data
    ) external {
        // Decode the swap callback data, checks that the UniswapV3Pool has the correct address.
        - CallbackLib.CallbackData memory decoded = abi.decode(data, (CallbackLib.CallbackData)); // @audit gas: memory to calldata as no modifications are made
        // Validate caller to ensure we got called from the AMM pool
        CallbackLib.validateCallback(msg.sender, address(FACTORY), decoded.poolFeatures);

        // Extract the address of the token to be sent (amount0 -> token0, amount1 -> token1)
        address token = amount0Delta > 0
            ? address(decoded.poolFeatures.token0)
            : address(decoded.poolFeatures.token1);

        // Transform the amount to pay to uint256 (take positive one from amount0 and amount1)
        uint256 amountToPay = amount0Delta > 0 ? uint256(amount0Delta) : uint256(amount1Delta);

        // Pay the required token from the payer to the caller of this contract
        SafeTransferLib.safeTransferFrom(token, decoded.payer, msg.sender, amountToPay);
    }

```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L447