## \[G-01\] you can save storage by reordering Holding struct fields in the following way.

```diff
struct PoolFeatures {
        address token0;
+        uint24 fee;
        address token1;
-        uint24 fee;
    }
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/CallbackLib.sol#L12

&nbsp;

## \[G-02\] Use constants instead of type(uintx).max

it's generally more gas-efficient to use constants instead of type(uintX).max when you need to set the maximum value of an unsigned integer type.

The reason for this is that the type(uintX).max expression involves a computation at runtime, whereas a constant is evaluated at compile-time. This means that using type(uintX).max can result in additional gas costs for each transaction that involves the expression.

By using a constant instead of type(uintX).max, you can avoid these additional gas costs and make your code more efficient.

Here's an example of how you can use a constant instead of type(uintX).max:

```
contract MyContract {
    uint120 constant MAX_VALUE = 2**120 - 1;
    
    function doSomething(uint120 value) public {
        require(value <= MAX_VALUE, "Value exceeds maximum");
        
        // Do something
    }
}
```

In the above example, we have a contract with a constant MAX\_VALUE that represents the maximum value of a uint120. When the doSomething function is called with a value parameter, it checks whether the value is less than or equal to MAX\_VALUE using the <= operator.

By using a constant instead of type(uint120).max, we can make our code more efficient and reduce the gas cost of our contract.

It's important to note that using constants can make your code more readable and maintainable, since the value is defined in one place and can be easily updated if necessary. However, constants should be used with caution and only when their value is known at compile-time.

```
917:   if (amount0 > uint128(type(int128).max) || amount1 > uint128(type(int128).max))
```

&nbsp;https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L917

```
if (tick > 0) sqrtR = type(uint256).max / sqrtR;
```

https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/libraries/Math.sol#L86

## \[G-03\] Use assembly for loops

We can use assembly to write a more gas efficient loop.

```
370: while (address(s_poolContext[poolId].pool) != address(0)) { //@audit
            poolId = PanopticMath.getFinalPoolId(poolId, token0, token1, fee);
        }
```

&nbsp;https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L370

## G-04\]use Mappings Instead of Arrays

There are two data types to describe lists of data in Solidity, arrays and maps, and their syntax and structure are quite different, allowing each to serve a distinct purpose. While arrays are packable and iterable, mappings are less expensive.

```
547:	uint256[] memory ids,  
548:    uint256[] memory amounts
```

&nbsp;https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L547

## \[G-05\] Mappings used within a function more than once should be cached to save gas

Cache such mappings and perform operations on them, if operations include modifications to the mapping(s) then remember to equate the mapping to it's cached counterpart at the end

```Solidity
614: if (
                (s_accountLiquidity[positionKey_to] != 0) ||
                (s_accountFeesBase[positionKey_to] != 0)
            ) revert Errors.TransferFailed();

            // Revert if not all balance is transferred
            uint256 fromLiq = s_accountLiquidity[positionKey_from];
            if (fromLiq.rightSlot() != liquidityChunk.liquidity()) revert Errors.TransferFailed();

            int256 fromBase = s_accountFeesBase[positionKey_from];

            //update+store liquidity and fee values between accounts
            s_accountLiquidity[positionKey_to] = fromLiq;
            s_accountLiquidity[positionKey_from] = 0;

            s_accountFeesBase[positionKey_to] = fromBase;
            s_accountFeesBase[positionKey_from] = 0;
```

&nbsp;https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L614C12-L630C53

```
1083: s_accountPremiumOwed[positionKey] = s_accountPremiumOwed[positionKey].add(deltaPremiumOwed);  
        s_accountPremiumGross[positionKey] = s_accountPremiumGross[positionKey].add(
            deltaPremiumGross
        );
```

&nbsp;https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1083C8-L1086C11

&nbsp;

## \[G-06\] The use of assembly code to load the left and right siblings into memory is more gas-efficient

```
sqrtPriceX96 = uint160((sqrtR >> 32) + (sqrtR % (1 << 32) == 0 ? 0 : 1));
```

https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/libraries/Math.sol#L89

## \[G-07\] Use assembly to perform efficient back-to-back calls

If similar external calls are performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer), which can potentially allow us to avoid memory expansion costs. In this case, we are also able to efficiently store the function signatures together in memory as one word, saving multiple MLOADs in the process.

Note: In order to do this optimization safely we will cache the free memory pointer value and restore it once we are done with our function calls. We will also set the zero slot back to 0 if neccessary.

&nbsp;

```
1083: s_accountPremiumOwed[positionKey] = s_accountPremiumOwed[positionKey].add(deltaPremiumOwed);  
        s_accountPremiumGross[positionKey] = s_accountPremiumGross[positionKey].add(
            deltaPremiumGross
        );
```

&nbsp;https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1083C8-L1086C11

```
1276:    uint128 collected0 = uint128(collectedAmounts.rightSlot());
         uint128 collected1 = uint128(collectedAmounts.leftSlot());
```

&nbsp;https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1276C17-L1277C75

```
1423:  premiumToken0 = acctPremia.rightSlot();
        premiumToken1 = acctPremia.leftSlot();
```

&nbsp;https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1423C6-L1424C47

```
1448:  feesBase0 = feesBase.rightSlot();
        feesBase1 = feesBase.leftSlot();
```

&nbsp;https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1448

```
uint160 lowPriceX96 = getSqrtRatioAtTick(liquidityChunk.tickLower());
 uint160 highPriceX96 = getSqrtRatioAtTick(liquidityChunk.tickUpper());
```

https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/libraries/Math.sol#L122C9-L123C79

## \[G-08\] Do not calculate constants

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

```
File: contracts/libraries/Math.sol

311: require(2 ** 64 > prod1);

338: prod0 |= prod1 * 2 ** 192;

373: require(2 ** 96 > prod1);

400: prod0 |= prod1 * 2 ** 160;

435: require(2 ** 128 > prod1);

497:  require(2 ** 192 > prod1);

524: prod0 |= prod1 * 2 ** 64;
```

&nbsp;https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/libraries/Math.sol#L435

```
if (optionRatios < 2 ** 64) {  
                optionRatios = 0;
            } else if (optionRatios < 2 ** 112) {
                optionRatios = 1;
            } else if (optionRatios < 2 ** 160) {
                optionRatios = 2;
            } else if (optionRatios < 2 ** 208) {
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L337C12-L344C34

```
if (optionRatios < 2 ** 64) {  
            return 0;
        } else if (optionRatios < 2 ** 112) {
            return 1;
        } else if (optionRatios < 2 ** 160) {
            return 2;
        } else if (optionRatios < 2 ** 208) {
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L417C9-L424C22

## \[G-09\] Amounts should be checked for 0 before calling a transfer

It is considered a best practice to verify for zero values before executing any transfers within smart contract functions. This approach helps prevent unnecessary external calls and can effectively reduce gas costs.

This practice is particularly crucial when dealing with the transfer of tokens or ether, as sending these assets to an address with a zero value will result in the loss of those assets.

In Solidity, you can determine whether a value is zero by utilizing the == operator. Here's an example demonstrating how to check for a zero value before performing a transfer:

&nbsp;

```
105: afterTokenTransfer(from, to, id, amount);


108: emit TransferSingle(msg.sender, from, to, id, amount);
```

&nbsp;https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L105C1-L108C63

```
afterTokenTransfer(from, to, ids, amounts);

 emit TransferBatch(msg.sender, from, to, ids, amounts);
```

https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/tokens/ERC1155Minimal.sol#L159C7-L161C64

&nbsp;

## \[G-10\] IF’s/require() statements that check input arguments should be at the top of the function

```
if (to.code.length != 0) {
            if (
                ERC1155Holder(to).onERC1155BatchReceived(msg.sender, from, ids, amounts, data) !=
                ERC1155Holder.onERC1155BatchReceived.selector
            ) {
                revert UnsafeRecipient();
            }
```

https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/tokens/ERC1155Minimal.sol#L163C9-L169C14

## \[G‑11\] Save gas by preventing zero amount in  and burn()

```
function _burn(address from, uint256 id, uint256 amount) internal {  @audit
        balanceOf[from][id] -= amount;

        emit TransferSingle(msg.sender, from, address(0), id, amount);
    }
```

&nbsp;https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L236

&nbsp;

## \[G‑12\] Using `bool`s for storage incurs overhead

note:  missed from the bot

```
// Booleans are more expensive than uint256 or any type that takes up a full
    // word because each write operation emits an extra SLOAD to first read the
    // slot's contents, replace the bits taken up by the boolean, and then write
    // back. This is the compiler's defense against contract upgrades and
    // pointer aliasing, and it cannot be disabled.
```

Use `uint256(1)` and `uint256(2)` for true/false to avoid a Gwarmaccess (**<ins>100 gas</ins>**) for the extra SLOAD, and to avoid Gsset (**20000 gas**) when changing from `false` to `true`, after having been `true` in the past.

```
119: bool locked;
```

&nbsp;https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L119

```
127: 	bool internal constant MINT = false;   
128:    bool internal constant BURN = true;
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L127

&nbsp;