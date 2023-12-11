## [GAS-1] Use calldata instead of memory for function arguments that do not get mutated
Mark data types as calldata instead of memory where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as calldata. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies memory storage.

Instances (4):
```
 function afterTokenTransfer(
        address from,
        address to,
        uint256[] memory ids,
        uint256[] memory amounts
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L544C4-L548C33

```
 function afterTokenTransfer(
        address from,
        address to,
        uint256[] memory ids,
        uint256[] memory amounts
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L252C4-L256C33

```
 function validateCallback(
        address sender,
        address factory,
        PoolFeatures memory features
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/CallbackLib.sol#L28C4-L31C37

```
  function multicall(bytes[] calldata data) public payable returns (bytes[] memory results) {
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/multicall/Multicall.sol#L12C3-L12C96

## [GAS-02] internal functions not called by the contract should be removed
If the functions are required by an interface, the contract should inherit from that interface and use the override keyword

Instances (6):
```
function rightSlot(uint256 self) internal pure returns (uint128) {
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L25C5-L25C71

```
function rightSlot(int256 self) internal pure returns (int128) {
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L32C5-L32C69

```
   function leftSlot(uint256 self) internal pure returns (uint128) {
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L89C2-L89C70

```
 function leftSlot(int256 self) internal pure returns (int128) {
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L96C4-L96C68

```
function getPoolId(address univ3pool) internal pure returns (uint64) {
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/PanopticMath.sol#L38C5-L38C75

```
 function getLiquidityChunk(
        uint256 tokenId,
        uint256 legIndex,
        uint128 positionSize,
        int24 tickSpacing
    ) internal pure returns (uint256 liquidityChunk) {
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/PanopticMath.sol#L82C4-L87C55

## [G-03] Expressions for constant values such as a call to keccak256() ,
should use immutable rather than constant.

Instances (1):
```
 bytes32 internal constant V3POOL_INIT_CODE_HASH =
        keccak256(
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Constants.sol#L25C4-L26C19

## [G-04] Using delete statement can save gas.

Instances (2):
```
 s_accountLiquidity[positionKey_from] = 0;
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L627C12-L627C54

```
 s_accountFeesBase[positionKey_from] = 0;
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L630C12-L630C53

## [G-05] Use hardcode address instead address(this)
Instead of using address(this) , it is more gas-efficient to pre-calculate and use the hardcoded
address.
References: https://book.getfoundry.sh/reference/forge-std/compute-create-address
https://twitter.com/transmissions11/status/1518507047943245824

Instances (1)
```
 (, uint256 feeGrowthInside0LastX128, uint256 feeGrowthInside1LastX128, , ) = univ3pool
            .positions(
                keccak256(
                    abi.encodePacked(
                        address(this),
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L1102C8-L1106C39

## [G-06] Gas optimizations by using external over public
Using public over external has an impact on execution cost.
If we run the following methods on Remix, we can see the difference
// transaction cost 21448 gas
// execution cost 176 gas
function tt() external returns(uint256) {
return 0;
}
// transaction cost 21558 gas
// execution cost 286 gas
function tt_public() public returns(uint256) {
return 0;
}

```
 function safeTransferFrom(
        address from,
        address to,
        uint256 id,
        uint256 amount,
        bytes calldata data
    ) public {
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L90C4-L96C15

```
 function balanceOfBatch(
        address[] calldata owners,
        uint256[] calldata ids
    ) public view returns (uint256[] memory balances) {
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L178C4-L181C56

```
 function calculateAMMSwapFeesLiquidityChunk(
        IUniswapV3Pool univ3pool,
        int24 currentTick,
        uint128 startingLiquidity,
        uint256 liquidityChunk
    ) public view returns (int256 feesEachToken) {
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/FeesCalc.sol#L54C4-L59C51

```
   function multicall(bytes[] calldata data) public payable returns (bytes[] memory results) {
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/multicall/Multicall.sol#L12C2-L12C96

## [G-07] abi.encode() is less efficient than abi.encodePacked()

```
  data = abi.encode(
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L760C11-L760C31

