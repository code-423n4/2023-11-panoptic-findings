## Summary

### Gas Optimization

no | Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] | Avoid contract existence checks by using low level calls | 14 | - |
| [G-02] | <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES or ( -= ) | 4 | - |
| [G-03] | Not using the named return variable when a function returns, wastes deployment gas | 4 | - |
| [G-04] | Can make the variable outside the loop to save gas | 4 | - |
| [G-05] | Require() or revert() statements that check input arguments should be at the top of the function | 3 | - |
| [G-06] | Using storage instead of memory for structs/arrays saves gas | 3 | - |
| [G-07] | With assembly, .call (bool success)  transfer can be done gas-optimized | 1 | - |
| [G-08] | Use constants instead of type(uintx).max / type(intx).max | 4 | - |
| [G-09] | A modifier used only once and not being inherited should be inlined to save gas | 1 | - |
| [G-10] | abi.encode() is less efficient than  abi.encodepacked() | 2 | - |
| [G-11] | Using delete statement can save gas | 1 | - |
| [G-12] | Sort Solidity operations using short-circuit mode | 1 | - |
| [G-13] | Use hardcode address instead address(this) | 2 | - |
| [G-14] | Use assembly for math (add, sub, mul, div) | 4 | - |
| [G-15] | Shorten the array rather than copying to a new one | 1 | - |
| [G-16] | Make 3 event parameters indexed when possible | 4 | - |
| [G-17] | Pre-increment and pre-decrement are cheaper than +1 ,-1 | 3 | - |

## Gas Optimizations  

## [G-01] Avoid contract existence checks by using low level calls		

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.			

```solidity
file: /contracts/SemiFungiblePositionManager.sol

323        if (s_poolContext[poolId].locked) revert Errors.ReentrantCall();


353        address univ3pool = FACTORY.getPool(token0, token1, fee);

356        if (address(univ3pool) == address(0)) revert Errors.UniswapPoolNotInitialized();

368        uint64 poolId = PanopticMath.getPoolId(univ3pool);

371            poolId = PanopticMath.getFinalPoolId(poolId, token0, token1, fee);


586            uint256 liquidityChunk = PanopticMath.getLiquidityChunk(
                id,
                leg,
                uint128(amount),
                univ3pool.tickSpacing()
            );

621            if (fromLiq.rightSlot() != liquidityChunk.liquidity()) revert Errors.TransferFailed();

675            tokenId = tokenId.flipToBurnToken();

1004            s_accountLiquidity[positionKey] = uint256(0).toLeftSlot(removedLiquidity).toRightSlot(
                updatedLiquidity
            )

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L323C1-L323C73


```solidity
file: /contracts/tokens/ERC1155Minimal.sol

164            if (
                ERC1155Holder(to).onERC1155BatchReceived(msg.sender, from, ids, amounts, data) !=
                ERC1155Holder.onERC1155BatchReceived.selector

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L164C1-L166C62


```solidity
file: /contracts/types/LeftRight.sol

213        if (self > uint256(type(int256).max)) revert Errors.CastingError();

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L213C1-L213C76


```solidity
file: /contracts/types/TokenId.sol

507                    uint256 tokenType = self.tokenType(i);
508                    uint256 tokenTypeP = self.tokenType(riskPartnerIndex);

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L507C1-L508C1


```solidity
file: /contracts/libraries/PanopticMath.sol

119        uint256 amount = uint256(positionSize) * tokenId.optionRatio(legIndex);

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/PanopticMath.sol#L119C1-L119C80

## [G-02] <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES or ( -= )

Using compound assignment operators for state variables (like State += X or State -= X …) it’s more expensive than using operator assignment (like State = State + X or State = State - X …)

```solidity 
file: /contracts/tokens/ERC1155Minimal.sol

///@audit  ' balacof ' is state var.

99        balanceOf[from][id] -= amount;

103            balanceOf[to][id] += amount;

145            balanceOf[from][id] -= amount;

149                balanceOf[to][id] += amount;

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L99C1-L99C39


## [G-03] Not using the named return variable when a function returns, wastes deployment gas

When you execute a function that returns values in Solidity, the EVM still performs the necessary operations to execute and return those values. This includes the cost of allocating memory and packing the return values. If the returned values are not utilized, it can be seen as wasteful since you are incurring gas costs for operations that have no effect.


```solidity
file: /contracts/types/LeftRight.sol

25    function rightSlot(uint256 self) internal pure returns (uint128) {
        return uint128(self);
    }


32    function rightSlot(int256 self) internal pure returns (int128) {
        return int128(self);
    }


89    function rightSlot(int256 self) internal pure returns (int128) {
        return int128(self);
    }


212    function toInt256(uint256 self) internal pure returns (int256) {
        if (self > uint256(type(int256).max)) revert Errors.CastingError();
        return int256(self);

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L25C1-L27C6


## [G-04]  Can make the variable outside the loop to save gas

Consider making the stack variables before the loop which gonna save gas

```solidity
file: /contracts/SemiFungiblePositionManager.sol

816            int256 _moved;
817            int256 _itmAmounts;
818            int256 _totalCollected;

586            uint256 liquidityChunk = PanopticMath.getLiquidityChunk(

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L861C1-L863C36


## [G-05]  Require() or revert() statements that check input arguments should be at the top of the function  

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas*) in a function that may ultimately revert in the unhappy case.


```solidity
file: /contracts/libraries/Math.sol

216            require(denominator > prod1);


435            require(2 ** 128 > prod1);


497            require(2 ** 192 > prod1);

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L216C1-L216C42


## [G-06] Using storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct.

```solidity
file: /contracts/SemiFungiblePositionManager.sol
 
547        uint256[] memory ids,
548        uint256[] memory amounts

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L547C1-L548C33


```solidity
file: /contracts/tokens/ERC1155Minimal.sol

181    ) public view returns (uint256[] memory balances) {

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L181C1-L181C56


## [G-07] With assembly, .call (bool success)  transfer can be done gas-optimized

return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), 
this storage disappears and gas optimization is provided.

```solidity
file: /contracts/multicall/Multicall.sol

15            (bool success, bytes memory result) = address(this).delegatecall(data[i]);

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/multicall/Multicall.sol#L15C1-L15C87


## [G-08] Use constants instead of type(uintx).max / type(intx).max

type(uint120).max or type(uint128).max / type(int120).max or type(int128).max  etc. it uses more gas in the distribution process and also for each transaction than constant usage.

```solidity 
file: /contracts/SemiFungiblePositionManager.sol

917        if (amount0 > uint128(type(int128).max) || amount1 > uint128(type(int128).max))

1390        if (atTick < type(int24).max) {

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L917C1-L917C88

```solidity
file: /contracts/libraries/Math.sol

86            if (tick > 0) sqrtR = type(uint256).max / sqrtR;

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L86C1-L86C61

```solidity
file: /contracts/types/LeftRight.sol

213        if (self > uint256(type(int256).max)) revert Errors.CastingError();

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L213C1-L213C76


## [G-09] A modifier used only once and not being inherited should be inlined to save gas

```solidity
file: /contracts/SemiFungiblePositionManager.sol

306    modifier ReentrancyLock(uint64 poolId) {

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L306C1-L306C45


## [G-10] abi.encode() is less efficient than  abi.encodepacked()

In terms of efficiency, abi.encodePacked() is generally considered to be more gas-efficient than abi.encode(), because it skips the step of adding function signatures and other metadata to the encoded data. However, this comes at the cost of reduced safety, as abi.encodePacked() does not perform any type checking or padding of data.

Refference: https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison

```solidity
file: /contracts/SemiFungiblePositionManager.sol

760            data = abi.encode(
                CallbackLib.CallbackData({
                    poolFeatures: CallbackLib.PoolFeatures({
                        token0: _univ3pool.token0(),
                        token1: _univ3pool.token1(),
                        fee: _univ3pool.fee()
                    }),
                    payer: msg.sender
                })
            );


1134        bytes memory mintdata = abi.encode(
            CallbackLib.CallbackData({ // compute by reading values from univ3pool every time
                    poolFeatures: CallbackLib.PoolFeatures({
                        token0: univ3pool.token0(),
                        token1: univ3pool.token1(),
                        fee: univ3pool.fee()
                    }),
                    payer: msg.sender
                })
        );            

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L760C1-L769C15

## [G-11] Using delete statement can save gas

```solidity
file: /contracts/types/TokenId.sol

338                optionRatios = 0;

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L338C1-L338C34


## [G-12] Sort Solidity operations using short-circuit mode

Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

```solidity
//f(x) is a low gas cost operation 
//g(y) is a high gas cost operation 
//Sort operations with different gas costs as follows
 f(x) || g(y) 
 f(x) && g(y)
```
In general, the left-hand side of the && operator is more likely to be more gas-consuming because it involves a comparison operation (itmAmounts != 0). Comparison operations usually require additional computational steps, such as fetching the values of itmAmounts and 0, performing the comparison, and then evaluating the result.
 
```solidity
file: /contracts/SemiFungiblePositionManager.sol

706        if ((itmAmounts != 0) && (swapAtMint)) {

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L706C1-L706C49


## [G-13] Use hardcode address instead address(this)
 
 Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.
References: https://book.getfoundry.sh/reference/forge-std/compute-create-address

an example :
```solidity
contract MyContract {
    address constant public CONTRACT_ADDRESS = 0x1234567890123456789012345678901234567890;
    
    function getContractAddress() public view returns (address) {
        return CONTRACT_ADDRESS;
    }
}
```

```solidity
file: /contracts/SemiFungiblePositionManager.sol

1106              address(this),

1148            address(this),

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L1106C1-L1106C39

## [G-14] Use assembly for math (add, sub, mul, div)

Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety. If using Solidity versions < 0.8.0 and you are using Safemath, you can gain significant gas savings by using assembly to calculate values and checking for overflow/underflow.

```solidity
file: /contracts/SemiFungiblePositionManager.sol

1280          premium0X64_base = Math
                    .mulDiv(collected0, totalLiquidity * 2 ** 64, netLiquidity ** 2)
                    .toUint128();
                premium1X64_base = Math
                    .mulDiv(collected1, totalLiquidity * 2 ** 64, netLiquidity ** 2)
                    .toUint128();
            }


1313           uint256 numerator = totalLiquidity ** 2 -
                        totalLiquidity *
                        removedLiquidity +
                        ((removedLiquidity ** 2) / 2 ** (VEGOID));
                    premium0X64_gross = Math

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L1280C1-L1286C14


```solidity
file: /contracts/types/LeftRight.sol

146            z = x + y;

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L146C1-L146C23


```solidity
file: /contracts/libraries/Math.sol

260            uint256 inv = (3 * denominator) ^ 2;

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L260C1-L260C49


## [G-15] Shorten the array rather than copying to a new one

Inline-assembly can be used to shorten the array by changing the length slot, so that the entries don't have to be copied to a new, shorter array

```solidity
file: /contracts/multicall/Multicall.sol

12    function multicall(bytes[] calldata data) public payable returns (bytes[] memory results) {
13        results = new bytes[](data.length);

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/multicall/Multicall.sol#L12C1-L13C44


## [G-16] Make 3 event parameters indexed when possible

It's the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

```solidity
file: /contracts/SemiFungiblePositionManager.sol

79    event PoolInitialized(address indexed uniswapPool);


86  event TokenizedPositionBurnt(
        address indexed recipient,
        uint256 indexed tokenId,
        uint128 positionSize
    );


97    event TokenizedPositionMinted(
        address indexed caller,
        uint256 indexed tokenId,
        uint128 positionSize
    );

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L79C1-L79C56


```solidity
file: /contracts/tokens/ERC1155Minimal.sol

44    event ApprovalForAll(address indexed owner, address indexed operator, bool approved);

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L44C1-L44C90


## [G-17] Pre-increment and pre-decrement are cheaper than +1 ,-1

```solidity
file: /contracts/SemiFungiblePositionManager.sol

82                    ? Constants.MIN_V3POOL_SQRT_RATIO + 1
83                    : Constants.MAX_V3POOL_SQRT_RATIO - 1,

877                    _leg = _isBurn ? numLegs - leg - 1 : leg;

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L829C1-L830C59

