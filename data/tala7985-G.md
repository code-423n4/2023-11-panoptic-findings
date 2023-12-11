## \[G-1\] Make 3 event parameters indexed when possible

It’s the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

```
event TokenizedPositionBurnt(
        address indexed recipient,
        uint256 indexed tokenId,
        uint128 positionSize
    );
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L86C5-L90C7

```
event TokenizedPositionMinted(
        address indexed caller,
        uint256 indexed tokenId,
        uint128 positionSize
    );
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L97C1-L101C7

```
event ApprovalForAll(address indexed owner, address indexed operator, bool approved);
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L44

## \[G‑2\] Using assembly to check for zero can save gas

Using assembly to check for zero can save gas by allowing more direct access to the evm and reducing some of the overhead associated with high-level operations in solidity.

```
if (address(univ3pool) == address(0)) revert Errors.UniswapPoolNotInitialized();
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L356

```
if (s_AddrToPoolIdData[univ3pool] != 0) return;
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L363

```
while (address(s_poolContext[poolId].pool) != address(0)) {
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L370

```
if (
                (s_accountLiquidity[positionKey_to] != 0) ||
                (s_accountFeesBase[positionKey_to] != 0)
            ) revert Errors.TransferFailed();
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L614C12-L617C46

```
s_accountLiquidity[positionKey_from] = 0;
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L627

```
s_accountFeesBase[positionKey_from] = 0;
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L630

```
if (positionSize == 0) revert Errors.OptionsBalanceZero();
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L671

```
if (univ3pool == IUniswapV3Pool(address(0))) revert Errors.UniswapPoolNotInitialized();
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L683

```
if ((itmAmounts != 0) && (swapAtMint)) {
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L706

```
if ((itm0 != 0) && (itm1 != 0)) {
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L774

```
} else if (itm0 != 0) {
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L810

```
if (swapAmount == 0) return int256(0);
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L820

```
if (isLong == 0) {
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L971

```
if (_tokenType == 0) {
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L1043

```
if (amountToCollect != 0) {
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L1217

```
if (netLiquidity != 0) {
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L1394

```
if (to.code.length != 0) {
```

&nbsp;https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L110

```
if (to.code.length != 0) {
```

&nbsp;https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L163

```
if (to.code.length != 0) {
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L222

```
if (
                legLowerTick % tickSpacing != 0 ||
                legUpperTick % tickSpacing != 0 ||
```

&nbsp;https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L396C10-L398C51

```
if (i == 0)
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L443

```
if (self.optionRatio(0) == 0) revert Errors.InvalidTokenIdParameter(1);
```

&nbsp;https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L464

```
if (self.optionRatio(i) == 0) {
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L469

```
if ((self >> (64 + 48 * i)) != 0) revert Errors.InvalidTokenIdParameter(1);
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L473

```
if ((self.width(i) == 0)) revert Errors.InvalidTokenIdParameter(5);
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L480

```
if (prod1 == 0) {
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L302

```
if (prod1 == 0) {
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L364

```
if (prod1 == 0) {
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L426

```
if (prod1 == 0) {
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L488

```
if (tokenId.asset(legIndex) == 0) {
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/PanopticMath.sol#L120

## \[G-03\] Pre-increment and pre-decrement are cheaper than +1 ,-1

```
? Constants.MIN_V3POOL_SQRT_RATIO + 1
: Constants.MAX_V3POOL_SQRT_RATIO - 1,
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L829C1-L830C59

## \[G-4\] += costs more gas than = + for state variables

```
amount0 += Math.getAmount0ForLiquidity(liquidityChunk);


amount1 += Math.getAmount1ForLiquidity(liquidityChunk);
                }
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L899C2-L902C18

```
balanceOf[from][id] -= amount;
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L99

```
balanceOf[to][id] += amount;
```

&nbsp;https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L103

```
balanceOf[from][id] -= amount;
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L145

```
balanceOf[to][id] += amount;
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L149

```
balanceOf[to][id] += amount;
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L217

```
balanceOf[from][id] -= amount;
```

&nbsp;https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L237

## \[G‑5\] Use a more recent version of solidity

```
pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L2

```
pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L2

```
pragma solidity ^0.8.0;
```

&nbsp;https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LiquidityChunk.sol#L2

```
pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L2

```
pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/CallbackLib.sol#L2

```
pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Constants.sol#L2

```
pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Errors.sol#L2C24-L2C24

```
pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/FeesCalc.sol#L2

```
pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L2

```
pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/PanopticMath.sol#L2

```
pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/SafeTransferLib.sol#L2

## \[G-6\] Public Functions To External

External call cost is less expensive than of public functions.  
Contracts are allowed to override their parents’ functions and change the visibility from external to public.  
The following functions could be set external to save gas and improve code quality. External call cost is less expensive than of public functions.

```
function setApprovalForAll(address operator, bool approved) public {
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L77

```
function safeTransferFrom(
        address from,
        address to,
        uint256 id,
        uint256 amount,
        bytes calldata data
    ) public {
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L90C2-L96C15

```
function safeBatchTransferFrom(
        address from,
        address to,
        uint256[] calldata ids,
        uint256[] calldata amounts,
        bytes calldata data
    ) public virtual {
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L128C4-L134C23

```
function balanceOfBatch(
        address[] calldata owners,
        uint256[] calldata ids
    ) public view returns (uint256[] memory balances) {
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L178C1-L181C56

```
function supportsInterface(bytes4 interfaceId) public pure returns (bool) {
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L200C80-L200C80

## \[G-7\] Avoid Unnecessary Type Conversions:

In the toRightSlot and toLeftSlot functions, there are unnecessary type conversions. You can simplify these operations to reduce gas consumption.

```diff
function toRightSlot(uint256 self, int128 right) internal pure returns (uint256) {
    require(right >= 0, Errors.LeftRightInputError());
-    return self + uint256(int256(right));
+     return self + uint256(right);
}
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L54C5-L59C6

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L67

## \[G-8\] Avoid Redundant Checks:

In the provided code, some checks like `(uint128(z) < uint128(x))` in the `add` function might be redundant. Since `z` is the sum of `x` and `y`, it should naturally be greater than or equal to both `x` and `y`. Unnecessary checks can be removed to reduce gas consumption.

```diff
-    if (z < x || (uint128(z) < uint128(x))) revert Errors.UnderOverFlow();

+     if (z < x) revert Errors.UnderOverFlow();
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L151

## \[G-9\] Combine Similar Functions:

If you notice patterns or similarities in different functions, consider consolidating them into more generic functions to reduce duplicate code.

```diff
-  function rightSlot(uint256 self) internal pure returns (uint128) {
    return uint128(self);
}

-  function leftSlot(uint256 self) internal pure returns (uint128) {
    return uint128(self >> 128);
}

// Combine similar logic into a single function
+  function getSlot(uint256 self, bool isLeft) internal pure returns (uint128) {
    if (isLeft) {
        return uint128(self >> 128);
    } else {
        return uint128(self);
    }
}
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L25C5-L27C6

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L89C4-L91C6

&nbsp;

```diff
-  function rightSlot(int256 self) internal pure returns (int128) {
        return int128(self);
    }
    
-    function leftSlot(int256 self) internal pure returns (int128) {
        return int128(self >> 128);
    }  
    
// Combine similar logic into a single function
+  function getSlot(int256 self, bool isLeft) internal pure returns (int128) {
    if (isLeft) {
        return int128(self >> 128);
    } else {
        return int128(self);
    }
}
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L32C5-L34C6

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L96C4-L98C6

## \[G-10\] Extra declaration in named returns that don't use them

Using both named returns and a return statement isn’t necessary, this consumes extra gas as one more variable that is not used is declared .

```
function mulDiv64(uint256 a, uint256 b) internal pure returns (uint256 result) {
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L286

```
function mulDiv128(uint256 a, uint256 b) internal pure returns (uint256 result) {
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L410

```
function mulDiv192(uint256 a, uint256 b) internal pure returns (uint256 result) {
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L472

```
) internal pure returns (uint256 liquidityChunk) {
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/PanopticMath.sol#L87