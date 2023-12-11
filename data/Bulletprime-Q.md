|  | issues | instance |
|---|---|---|
| [L-01]| error in interface import| 1 |
|[L-02] | consider validating the `payer` address| 1 |
|[L-03] | consider adding a zero check | 1 |
DETAILS

[L-01] Error in interface import
Error due to the fact that the specified files or modules are not found in the given paths. The `IUniswapV3Factory` and `IUniswapV3Pool` are interfaces that are being imported from specified paths. If these files are not found at the given paths, the imports will fail.
Ensure that the specified files exist at the given paths. If they don't, you might need to change the import paths to point to the correct locations of these files. As This could impact the entire codebase because if these interfaces are used across the code, the incorrect import paths will lead to errors in the entire codebase.  

```
// Interfaces 
import {IUniswapV3Factory} from "univ3-core/interfaces/IUniswapV3Factory.sol";
import {IUniswapV3Pool} from "univ3-core/interfaces/IUniswapV3Pool.sol";
```

[L-02] Consider validating the `payer` address
The `uniswapV3MintCallback` function does not explicitly validate or verify the `payer` address that is passed in the `data` parameter.
from the code, the `decoded.payer` address is used as the `from` address in the `SafeTransferLib.safeTransferFrom` function. However, the `decoded.payer` address is not explicitly validated or verified anywhere in the function. This means that an attacker could potentially set the `decoded.payer` address to an address they control, and then call the `uniswapV3MintCallback` function to transfer tokens to themselves. A correct implementation can be; the `decoded.payer` address should be validated and verified to ensure that it matches the `payer` address of the `minted` liquidity. This would prevent an attacker from setting the `decoded.payer` address to an address they control.

```
solidity
// Decode the mint callback data
CallbackLib.CallbackData memory decoded = abi.decode(data, (CallbackLib.CallbackData));
// Validate caller to ensure we got called from the AMM pool
CallbackLib.validateCallback(msg.sender, address(FACTORY), decoded.poolFeatures);
// Sends the amount0Owed and amount1Owed quantities provided
if (amount0Owed > 0)
    SafeTransferLib.safeTransferFrom(
        decoded.poolFeatures.token0,
        decoded.payer,
        msg.sender,
        amount0Owed
    );
if (amount1Owed > 0)
    SafeTransferLib.safeTransferFrom(
        decoded.poolFeatures.token1,
        decoded.payer,
        msg.sender,
        amount1Owed
    );
```
L3 Consider adding a zero check before division 
The `_getPremiaDeltas` function calculates the change in Premia owed and gross Premia based on the current liquidity and the collected amounts, here there is no explicit check to see if both the net and total liquidity are zero, If both the `netLiquidity` and `totalLiquidity` are zero, it means that the position has no liquidity, and the contract has no exposure to the Uniswap pool. Although this threat can potentially arise from the funtion  that calculates the `totalLiquidity` and `netLiquidity` based on the current liquidity. This is because the `totalLiquidity` and `netLiquidity` are calculated using division, and division by zero is not allowed in Solidity. Therefore, it's important to ensure that the `totalLiquidity` and `netLiquidity` are not zero before performing the division.
```
solidity
uint256 totalLiquidity = netLiquidity + removedLiquidity;

            uint128 premium0X64_base;
            uint128 premium1X64_base;

            {
                uint128 collected0 = uint128(collectedAmounts.rightSlot());
                uint128 collected1 = uint128(collectedAmounts.leftSlot());

                // compute the base premium as collected * total / net^2 (from Eqn 3)
                premium0X64_base = Math
                    .mulDiv(collected0, totalLiquidity * 2 ** 64, netLiquidity ** 2)
                    .toUint128();
                premium1X64_base = Math
                    .mulDiv(collected1, totalLiquidity * 2 ** 64, netLiquidity ** 2)
                    .toUint128();
            }

            {
                uint128 premium0X64_owed;
                uint128 premium1X64_owed;
                {
                    // compute the owed premium (from Eqn 3)
                    uint256 numerator = netLiquidity + (removedLiquidity / 2 ** VEGOID);

                    premium0X64_owed = Math
                        .mulDiv(premium0X64_base, numerator, totalLiquidity)
                        .toUint128();
                    premium1X64_owed = Math
                        .mulDiv(premium1X64_base, numerator, totalLiquidity)
                        .toUint128();

                    deltaPremiumOwed = uint256(0).toRightSlot(premium0X64_owed).toLeftSlot(
                        premium1X64_owed
                    );
```
 However, you can add this check to the `_getPremiaDeltas` function. Where the check for zero before division should be made. 
 ```
solidity
 // Check if both the net and total liquidity are zero
if (netLiquidity == 0 && totalLiquidity == 0) {
    // Handle the case where both the net and total liquidity are zero
}
```
