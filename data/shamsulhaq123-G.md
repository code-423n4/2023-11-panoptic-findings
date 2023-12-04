## Gas optimization

## Sumary

| No|Issue|Instances|
|-|:-|:-:|
|[G-01]|State variables only set in the constructor should be declared immutable|1|
|[G-02]|Use calldata instead of memory for function arguments|3|
|[G-03]|Use constants instead of type(uintx).max|4|
|[G-04]| Use hardcode address instead of address(this)|3|
|[G-05]|Use assembly for math (add, sub, mul, div)|5|
|[G-06]|Do not calculate constant|3|
|[G-07]|Non efficient zero initialization|4|
|[G-08]|Pre-increments and pre-decrements are cheaper than post-increments and post-decrements|5|
|[G-09]|Use assembly for loops|4|
|[G-10]|Use assembly to perform efficient back-to-back calls|2|
|[G-11]|Unnecessary look up in if condition|11|
|[G-12]|+= costs more gas than = + for state variables|5|
|[G-13]| Counting down in for statements is more gas efficient|7|
|[G-14]|Use assembly to check for address(0)|6|

## [G-01] State variables only set in the constructor should be declared immutable

```Solidity

file: SemiFungiblePositionManager.sol

342   constructor(IUniswapV3Factory _factory) {
343        FACTORY = _factory;
    }


```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L343

## [G-02] Use calldata instead of memory for function arguments that do not get mutated
When you specify a data location as memory, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. If the value is a large, complex type, using memory may result in extra memory expansion costs.

```solidity

file: 

547   uint256[] memory ids,
548        uint256[] memory amounts


```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L547-L548

```solidity

file: contracts/tokens/ERC1155Minimal.sol

255  uint256[] memory ids,
256  uint256[] memory amounts

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L255-L256

```solidity

file: contracts/libraries/CallbackLib.sol

31  PoolFeatures memory features
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/CallbackLib.sol#L31

## [G-03] Use constants instead of type(uintx).max

type(uint120).max or type(uint128).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.


```solidity

file: contracts/SemiFungiblePositionManager.sol

917  if (amount0 > uint128(type(int128).max) || amount1 > uint128(type(int128).max))

1390     if (atTick < type(int24).max)
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L917
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L1390

```solidity

file: contracts/types/LeftRight.sol

213  if (self > uint256(type(int256).max)) revert Errors.CastingError();
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L213

```solidity

file: contracts/libraries/Math.sol

86   if (tick > 0) sqrtR = type(uint256).max / sqrtR;
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L86

## [G-4]  Use hardcode address instead of address(this)
 
 Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.
 
```solidity

file: contracts/SemiFungiblePositionManager.sol

1106    address(this)

          (uint256 amount0, uint256 amount1) = univ3pool.mint(
 1148            address(this),
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L1106
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L1148

```solidity

file: contracts/multicall/Multicall.sol

15   (bool success, bytes memory result) = address(this).delegatecall(data[i]);
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/multicall/Multicall.sol#L15

## [G-05] Use assembly for math (add, sub, mul, div)

Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety. If using Solidity versions < 0.8.0 and you are using Safemath, you can gain significant gas savings by using assembly to calculate values and checking for overflow/underflow


```solidity

file: contracts/SemiFungiblePositionManager.sol

1209   int256 amountToCollect = _getFeesBase(univ3pool, startingLiquidity, liquidityChunk).sub(
            s_accountFeesBase[positionKey]
        );

1409   amountToCollect = feesBase.sub(s_accountFeesBase[positionKey]);        

1418 ? acctPremia.add(deltaPremiumOwed)
                    : acctPremia.add(deltaPremiumGross);
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L1209
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L1409
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L1418-L1419

```solidity

file: contracts/types/LeftRight.sol

177    function sub(int256 x, int256 y) internal pure returns (int256 z) 

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L177

```solidity

file: contracts/libraries/Math.sol

201   prod0 := mul(a, b)
202   prod1 := sub(sub(mm, prod0), lt(mm, prod0))

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L201

## [G-06] Do not calculate constant
When you define a constant in Solidity, the compiler can calculate its value at compile-time and replace all references to the constant with its computed value. This can be helpful for readability and reducing the size of the compiled code, but it can also increase gas usage at runtime.

```solidity

file: contracts/SemiFungiblePositionManager.sol

830   ? Constants.MIN_V3POOL_SQRT_RATIO + 1
830   : Constants.MAX_V3POOL_SQRT_RATIO - 1,

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L829-L830

```solidity

file: contracts/types/TokenId.sol

385  int24 minTick = (Constants.MIN_V3POOL_TICK / tickSpacing) * tickSpacing;
386   int24 maxTick = (Constants.MAX_V3POOL_TICK / tickSpacing) * tickSpacing;
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L385-L386

```solidity

file: contracts/libraries/Constants.sol

11   int24 internal constant MIN_V3POOL_TICK = -887272;
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Constants.sol#L11

## [G-07] Non efficient zero initialization
Solidity does not recognize null as a value, so uint variables are initialized to zero. Setting a uint variable to zero is redundant and can waste gas.

```solidity

file: contracts/SemiFungiblePositionManager.sol

550    for (uint256 i = 0; i < ids.length; )
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L550

```solidity

file: contracts/tokens/ERC1155Minimal.sol

141   for (uint256 i = 0; i < ids.length; )

187   for (uint256 i = 0; i < owners.length; ++i) 
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L141
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L187

```solidity

file: contracts/types/TokenId.sol

468    for (uint256 i = 0; i < 4; ++i) 
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L468

```solidity

file: contracts/multicall/Multicall.sol

14    for (uint256 i = 0; i < data.length; ) 
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/multicall/Multicall.sol#L14

## [G-08] Pre-increments and pre-decrements are cheaper than post-increments and post-decrements
Saves 5 gas per iteration

```solidity

file: contracts/SemiFungiblePositionManager.sol

552   unchecked {
                ++i;
            }
631    unchecked {
                ++leg;
            }
909   unchecked {
                ++leg;
            }
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L552
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L631
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L909

```solidity

file: contracts/tokens/ERC1155Minimal.sol

154   unchecked {
                ++i;
            }
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L154
```solidity

file: contracts/multicall/Multicall.sol

32 unchecked {
                ++i;
            }
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/multicall/Multicall.sol#L32

## [G-09] Use assembly for loops
In the following instances, assembly is used for more gas efficient loops.

```solidity

file: main/contracts/tokens/ERC1155Minimal.sol

141    for (uint256 i = 0; i < ids.length; ) 

187   for (uint256 i = 0; i < owners.length; ++i) 
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L141
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L187

```solidity

file: contracts/types/TokenId.sol

468   for (uint256 i = 0; i < 4; ++i)
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L468

```solidity

file: contracts/multicall/Multicall.sol

14    for (uint256 i = 0; i < data.length; ) 
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/multicall/Multicall.sol#L14

## [G-10] Use assembly to perform efficient back-to-back calls
If a similar external call is performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer + zero slot), which can potentially allow us to avoid memory expansion costs.

```solidity

file: contracts/types/LeftRight.sol

198    function toInt128(int256 self) internal pure returns (int128 selfAsInt128) {
        if (!((selfAsInt128 = int128(self)) == self)) revert Errors.CastingError();
    }

    /// @notice Downcast uint256 to a uint128, revert on overflow
    /// @param self the uint256 to be downcasted to uint128
    /// @return selfAsUint128 the downcasted uint256 now as uint128
    function toUint128(uint256 self) internal pure returns (uint128 selfAsUint128) {
        if (!((selfAsUint128 = uint128(self)) == self)) revert Errors.CastingError();
    }

    /// @notice Cast a uint256 to an int256, revert on overflow
    /// @param self the uint256 to be downcasted to uint128
    /// @return the incoming uint256 but now of type int256
    function toInt256(uint256 self) internal pure returns (int256) {
        if (self > uint256(type(int256).max)) revert Errors.CastingError();
        return int256(self);
215   }
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L198-L216

```solidity

file: contracts/types/LiquidityChunk.sol

78  function addLiquidity(uint256 self, uint128 amount) internal pure returns (uint256) {
        unchecked {
            return self + uint256(amount);
        }
    }

    /// @notice Add the lower tick to this chunk.
    /// @param self the LiquidityChunk
    /// @param _tickLower the lower tick to add
    /// @return the chunk with added lower tick
    function addTickLower(uint256 self, int24 _tickLower) internal pure returns (uint256) {
        unchecked {
            return self + (uint256(uint24(_tickLower)) << 232);
        }
    }

    /// @notice Add the upper tick to this chunk.
    /// @param self the LiquidityChunk
    /// @param _tickUpper the upper tick to add
    /// @return the chunk with added upper tick
    function addTickUpper(uint256 self, int24 _tickUpper) internal pure returns (uint256) {
        unchecked {
            // convert tick upper to uint24 as explicit conversion from int24 to uint256 is not allowed
            return self + ((uint256(uint24(_tickUpper))) << 208);
        }
    }

    /*//////////////////////////////////////////////////////////////
                                DECODING
    //////////////////////////////////////////////////////////////*/

    /// @notice Get the lower tick of a chunk.
    /// @param self the LiquidityChunk uint256
    /// @return the lower tick of this chunk
    function tickLower(uint256 self) internal pure returns (int24) {
        unchecked {
            return int24(int256(self >> 232));
        }
    }

    /// @notice Get the upper tick of a chunk.
    /// @param self the LiquidityChunk uint256
    /// @return the upper tick of this chunk
    function tickUpper(uint256 self) internal pure returns (int24) {
        unchecked {
            return int24(int256(self >> 208));
        }
    }

    /// @notice Get the amount of liquidity/size of a chunk.
    /// @param self the LiquidityChunk uint256
    /// @return the size of this chunk
    function liquidity(uint256 self) internal pure returns (uint128) {
        unchecked {
            return uint128(self);
        }
135    }


```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LiquidityChunk.sol#L78-L135

## [G-11] Unnecessary look up in if condition
If the || condition isn’t required, the second condition will have been looked up unnecessarily.

```solidity

file:contracts/SemiFungiblePositionManager.sol

614   if (
                (s_accountLiquidity[positionKey_to] != 0) ||

713  if ((newTick >= tickLimitHigh) || (newTick <= tickLimitLow)) revert Errors.PriceBoundFail();

917   if (amount0 > uint128(type(int128).max) || amount1 > uint128(type(int128).max))
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L614
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L713
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L917

```solidity

file: contracts/tokens/ERC1155Minimal.sol

97    if (!(msg.sender == from || isApprovedForAll[from][msg.sender])) revert NotAuthorized();

135   if (!(msg.sender == from || isApprovedForAll[from][msg.sender])) revert NotAuthorized();
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L97
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L135


```solidity

file: contracts/types/LeftRight.sol

151  if (z < x || (uint128(z) < uint128(x))) revert Errors.UnderOverFlow();

167  if (left128 != left256 || right128 != right256) revert Errors.UnderOverFlow();

185   if (left128 != left256 || right128 != right256) revert Errors.UnderOverFlow();
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L151
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L167
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L185

```solidity

file: contracts/types/TokenId.sol

396   if (
                legLowerTick % tickSpacing != 0 ||
                legUpperTick % tickSpacing != 0 ||
399             legLowerTick < minTick ||

482 if (
                    (self.strike(i) == Constants.MIN_V3POOL_TICK) ||

497    if (
                        (self.asset(riskPartnerIndex) != self.asset(i)) ||                    
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L396-L399
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L482
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L497

## [G-12] += costs more gas than = + for state variables
use = + or = - instead to save gas

```solidity

file: contracts/SemiFungiblePositionManager.sol

899  amount0 += Math.getAmount0ForLiquidity(liquidityChunk);

901  amount1 += Math.getAmount1ForLiquidity(liquidityChunk);

999   removedLiquidity += chunkLiquidity;
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L899-L901
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L999

```solidity

file: contracts/tokens/ERC1155Minimal.sol

103   balanceOf[to][id] += amount;

149    balanceOf[to][id] += amount;

217  balanceOf[to][id] += amount;
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L103
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L149
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L217

## [G‑13] Counting down in for statements is more gas efficient
Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

```solidity

file: contracts/SemiFungiblePositionManager.sol

550  for (uint256 i = 0; i < ids.length; ) 

583  for (uint256 leg = 0; leg < numLegs; ) 

860     for (uint256 leg = 0; leg < numLegs; ) 
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L550
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L583
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L860

```solidity

file: contracts/tokens/ERC1155Minimal.sol#

141  for (uint256 i = 0; i < ids.length; )

187  for (uint256 i = 0; i < owners.length; ++i)
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L141
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L187

```solidity

file: contracts/types/TokenId.sol

468  for (uint256 i = 0; i < 4; ++i)

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L468

```solidity

file: contracts/multicall/Multicall.sol

14  for (uint256 i = 0; i < data.length; )

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/multicall/Multicall.sol#L14

## [G-14] Use assembly to check for address(0)
Saves 6 gas per instance

```solidity

file: contracts/SemiFungiblePositionManager.sol

356  if (address(univ3pool) == address(0)) revert Errors.UniswapPoolNotInitialized();
 
370    while (address(s_poolContext[poolId].pool) != address(0)) {
      
683 if (univ3pool == IUniswapV3Pool(address(0))) revert Errors.UniswapPoolNotInitialized();
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L356
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L370
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L683

```solidity

file: contracts/tokens/ERC1155Minimal.sol

220   emit TransferSingle(msg.sender, address(0), to, id, amount);

224  ERC1155Holder(to).onERC1155Received(msg.sender, address(0), id, amount, "") !=

239   emit TransferSingle(msg.sender, from, address(0), id, amount);
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L220
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L224
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L239

```solidity

file: 



```
