
| No |Issue| Instence|
|----|-----|---------|
|[L-01]| Some tokens do not consider type(uint256).max as an infinite approval|5
|[l-02]| Cap state variables at reasonable values|10


## [L-01] Some tokens do not consider type(uint256).max as an infinite approval
Some tokens such as COMP downcast such approvals to uint96 and use that as a raw value rather than interpreting it as an infinite approval. Eventually these approvals will reach zero, at which point the calling contract will no longer function properly.

There are 5  instance(s) of this issue:

```solidity

file: SemiFungiblePositionManager.sol

830  : Constants.MAX_V3POOL_SQRT_RATIO - 1,

917   if (amount0 > uint128(type(int128).max) || amount1 > uint128(type(int128).max))

1390  if (atTick < type(int24).max) {
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L830
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L917
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L1390

```solidity

file: LeftRight.sol

213  if (self > uint256(type(int256).max)) revert Errors.CastingError();

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L213

```solidity

file: libraries/Math.sol

86  if (tick > 0) sqrtR = type(uint256).max / sqrtR;


```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L86

## [L-02] Cap state variables at reasonable values
Consider adding maximum value checks to ensure that state variables cannot be set to values that may excessively harm users.

```solidity

file: SemiFungiblePositionManager.sol

809   swapAmount = -net0;

812   swapAmount = -itm0;

815   swapAmount = -itm1;

899    amount0 += Math.getAmount0ForLiquidity(liquidityChunk);

901    amount1 += Math.getAmount1ForLiquidity(liquidityChunk);

906   itmAmounts = itmAmounts.add(_itmAmounts);

1045   _itmAmounts = _itmAmounts.toLeftSlot(_moved.leftSlot());

1186  movedAmounts = int256(0).toRightSlot(-int128(int256(amount0))).toLeftSlot(
                -int128(int256(amount1))
            );
1214    amountToCollect = amountToCollect.sub(movedInLeg);    

1409   amountToCollect = feesBase.sub(s_accountFeesBase[positionKey]);
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L809
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L812
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L815
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L899
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L906
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L1045
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L1186
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L1214
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L1409

```solidity

file: tokens/ERC1155Minimal.sol

143  amount = amounts[i];


```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L143