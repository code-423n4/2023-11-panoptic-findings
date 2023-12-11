
##  Gas Summary

| Number | Issue | Instances|
|--------|-------|----------|
|[G-01]| Remove ternary operator and use if stament instead of ternary operator  | 2 |
|[G-02]| Switching between 1 and 2 instead of 0 and 1 (or false and true) is more gas efficient  | 2 |
|[G-03]| State variable read in a loop  | 14 |
|[G-04]| Default value initialization  | 6 |
|[G-05]| Use constants instead of type(uintX).max  | 4 |
|[G-06]| Using delete statement can save gas | 2 |
|[G-07]| Using calldata instead of memory for read-only arguments in external functions saves gas  | 6 |
|[G-08]| Use uint256(1)/uint256(2) instead for true and false boolean states  | 9 |
|[G-09]| Unused Named Return Variables  | 6 |
|[G-10]| Use modifiers rather than invoking functions to perform checks  | 3 |
|[G-11]| IF’s/require() statements that check input arguments should be at the top of the function  | 2 |
|[G-12]| Use assembly in place of abi.decode to extract calldata values more efficiently  | 2 |
|[G-13]| Save loop calls  | 1 |
|[G-14]| Use hardcode address instead address(this)  | 3 |
|[G-15]| Use the existing Local variable/global variable when equal to a state variable to avoid reading from state  | 3 |
|[G-16]| A modifier used only once and not being inherited should be inlined to save gas  | 1 |
|[G-17]| Use assembly for loops  | 3 |
|[G-18]| Use assembly to check for address(0)  | 6 |
|[G-19]| Amounts should be checked for 0 before calling a transfer  | 4 |
|[G-20]| abi.encode() is less efficient than abi.encodePacked()  | 2 |
|[G-21]| Assigning keccak operations to constant variables results in extra gas costs  | 1 |
|[G-22]| >= costs less gas than >  | 7 |
|[G-23]| Declare the variables outside the loop  | 3 |
|[G-24]| Don’t cache calls that are only used once  | 2 |
|[G-25]| Loop best practice to save gas  | 2 |
|[G-26]| Counting down in for statements is more gas efficient  | 7 |
|[G-27]| Create immutable variable to avoid an external call  | 1 |
|[G-28]| public functions not called by the contract should be declared external instead  | 7 |


## [G-01] Remove ternary operator and use if stament instead of ternary operator 

```solidity
file:  main/contracts/SemiFungiblePositionManager.sol

1385     uint256 acctPremia = isLong == 1
            ? s_accountPremiumOwed[positionKey]
            : s_accountPremiumGross[positionKey];
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L1385-L1387

Before Gas value:
[PASS] test_Success_getAccountPremium_getAccountFeesBase_ShortOnly(uint256,uint256,int256,uint256,uint256) (runs: 1, μ: 2087130, ~: 2087130)

After Gas Value:
[PASS] test_Success_getAccountPremium_getAccountFeesBase_ShortOnly(uint256,uint256,int256,uint256,uint256) (runs: 1, μ: 2066368, ~: 2066368)


### Recommanded code 

```solidity

uint256 acctPremia;

        if(isLong == 1){
            s_accountPremiumOwed[positionKey];
        }else {
            s_accountPremiumGross[positionKey];
        }

```

```solidity
file: main/contracts/libraries/Math.sol

40    uint256 absTick = tick < 0 ? uint256(-int256(tick)) : uint256(int256(tick));

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L40

Before Gas Value:
[PASS] test_Fail_getSqrtRatioAtTick() (gas: 10723)

After Gas value:        
[PASS] test_Fail_getSqrtRatioAtTick() (gas: 8432)

## [G-02] Switching between 1 and 2 instead of 0 and 1 (or false and true) is more gas efficient

SSTORE from 0 to 1 (or any non-zero value) costs 20000 gas. SSTORE from 1 to 2 (or any other non-zero value) costs 5000 gas.

By storing the original value once again, a refund is triggered (https://eips.ethereum.org/EIPS/eip-2200).

Since refunds are capped to a percentage of the total transaction’s gas, it is best to keep them low, to increase the likelihood of the full refund coming into effect.

Therefore, switching between 1, 2 instead of 0, 1 will be more gas efficient.

See: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/86bd4d73896afcb35a205456e361436701823c7a/contracts/security/ReentrancyGuard.sol#L29-L33

```solidity
file: main/contracts/SemiFungiblePositionManager.sol

127   bool internal constant MINT = false;

128   bool internal constant BURN = true;

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L127

## [G-03] State variable read in a loop

The state variable should be cached in a local variable rather than reading it on every iteration of the for-loop, which will replace each Gwarmaccess (100 gas) with a much cheaper stack read.

```solidity
File: contracts/SemiFungiblePositionManager.sol

370          while (address(s_poolContext[poolId].pool) != address(0)) {

615          (s_accountLiquidity[positionKey_to] != 0) ||

616          (s_accountFeesBase[positionKey_to] != 0)

620          uint256 fromLiq = s_accountLiquidity[positionKey_from];

623          int256 fromBase = s_accountFeesBase[positionKey_from];

626          s_accountLiquidity[positionKey_to] = fromLiq;

627          s_accountLiquidity[positionKey_from] = 0;

629          s_accountFeesBase[positionKey_to] = fromBase;

630          s_accountFeesBase[positionKey_from] = 0;

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L370

```solidity
File: contracts/tokens/ERC1155Minimal.sol

145         balanceOf[from][id] -= amount;

149         balanceOf[to][id] += amount;

188         balances[i] = balanceOf[owners[i]][ids[i]];

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L145


```solidity
File: contracts/types/TokenId.sol

483       (self.strike(i) == Constants.MIN_V3POOL_TICK) ||

484       (self.strike(i) == Constants.MAX_V3POOL_TICK)

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L483

## [G-04] Default value initialization

If a variable is not set/initialized, it is assumed to have the default value(0, false, 0x0 etc depending on the data type). Explicitly initializing it with its default value is an anti-pattern and wastes gas.

```solidity
file: main/contracts/SemiFungiblePositionManager.sol

550  for (uint256 i = 0; i < ids.length; ) {

583  for (uint256 leg = 0; leg < numLegs; ) {
    
860  for (uint256 leg = 0; leg < numLegs; ) {

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L550

```solidity
file: main/contracts/tokens/ERC1155Minimal.sol

141    for (uint256 i = 0; i < data.length; ) {

187   for (uint256 i = 0; i < owners.length; ++i) {

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L141


After gas value:
[PASS] testFail_safeBatchTransferFrom_insufficientBalance(address,uint256[10],uint256[10],uint256[10],uint256) (runs: 1, μ: 314489, ~: 314489)

Before gas value:
[PASS] testFail_safeBatchTransferFrom_insufficientBalance(address,uint256[10],uint256[10],uint256[10],uint256) (runs: 1, μ: 225285, ~: 225285)


```solidity
file: main/contracts/types/TokenId.sol

468   for (uint256 i = 0; i < 4; ++i) {

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L468



## [G-05] Use constants instead of type(uintX).max

Saves 52 GAS in 4 instances.

Using constants instead of type(uintX).max saves gas in Solidity. This is because the type(uintX).max function has to dynamically calculate the maximum value of a uint256, which can be expensive in terms of gas. Constants, on the other hand, are stored in the bytecode of your contract, so they do not have to be recalculated every time you need them. Saves 13 GAS.


```solidity
file:  main/contracts/SemiFungiblePositionManager.sol

917   if (amount0 > uint128(type(int128).max) || amount1 > uint128(type(int128).max))

1390  if (atTick < type(int24).max) {

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L917


```solidity
file: main/contracts/libraries/Math.sol

86   if (tick > 0) sqrtR = type(uint256).max / sqrtR;

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L86


```solidity
file: main/contracts/types/LeftRight.sol

213  if (self > uint256(type(int256).max)) revert Errors.CastingError();

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L213


## [G-06] Using delete statement can save gas

```solidity
file: main/contracts/SemiFungiblePositionManager.sol

627   s_accountLiquidity[positionKey_from] = 0;

630    s_accountFeesBase[positionKey_from] = 0;

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L627

## [G-07] Using calldata instead of memory for read-only arguments in external functions saves gas


```solidity
file: main/contracts/SemiFungiblePositionManager.sol

413    CallbackLib.CallbackData memory decoded = abi.decode(data, (CallbackLib.CallbackData));

444    CallbackLib.CallbackData memory decoded = abi.decode(data, (CallbackLib.CallbackData));

544    function afterTokenTransfer(
        address from,
        address to,
        uint256[] memory ids,
        uint256[] memory amounts
    ) internal override {

750  bytes memory data;

1134  bytes memory mintdata = abi.encode(

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L413


```solidity
file: main/contracts/tokens/ERC1155Minimal.sol

252    function afterTokenTransfer(
        address from,
        address to,
        uint256[] memory ids,
        uint256[] memory amounts
    ) internal virtual;

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L252-L257


## [G-08] Use uint256(1)/uint256(2) instead for true and false boolean states

Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. see source:
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27


```solidity
file: main/contracts/SemiFungiblePositionManager.sol

127    bool internal constant MINT = false;

128    bool internal constant BURN = true;

119    bool locked;

668    bool isBurn

685    bool swapAtMint;

748    bool zeroForOne; 

852    bool isBurn

941    bool _isBurn

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L127

```solidity
file: main/contracts/libraries/SafeTransferLib.sol

17   bool success;

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/SafeTransferLib.sol#L17

## [G-09] Unused Named Return Variables

Description
Named return variables allow for clear and explicit naming of values to be returned from a function. However, when these variables are unused, it can lead to confusion and make the code less maintainable.


```solidity
file: main/contracts/SemiFungiblePositionManager.sol

476    function burnTokenizedPosition(
        uint256 tokenId,
        uint128 positionSize,
        int24 slippageTickLimitLow,
        int24 slippageTickLimitHigh
    )
        external
        ReentrancyLock(tokenId.univ3pool())
        returns (int256 totalCollected, int256 totalSwapped, int24 newTick)
    {

510  function mintTokenizedPosition(
        uint256 tokenId,
        uint128 positionSize,
        int24 slippageTickLimitLow,
        int24 slippageTickLimitHigh
    )
        external
        ReentrancyLock(tokenId.univ3pool())
        returns (int256 totalCollected, int256 totalSwapped, int24 newTick)
    {

1343  function getAccountLiquidity(
        address univ3pool,
        address owner,
        uint256 tokenType,
        int24 tickLower,
        int24 tickUpper
    ) external view returns (uint256 accountLiquidities) {
    
1371   function getAccountPremium(
        address univ3pool,
        address owner,
        uint256 tokenType,
        int24 tickLower,
        int24 tickUpper,
        int24 atTick,
        uint256 isLong
    ) external view returns (uint128 premiumToken0, uint128 premiumToken1) {

1437   function getAccountFeesBase(
        address univ3pool,
        address owner,
        uint256 tokenType,
        int24 tickLower,
        int24 tickUpper
    ) external view returns (int128 feesBase0, int128 feesBase1) {

1468 function getPoolId(address univ3pool) external view returns (uint64 poolId) {

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L476-L485

## [G-10] Use modifiers rather than invoking functions to perform checks

By using modifiers in place of functions to perform checks, we could reduce the gas cost by up to 40 units. In using modifiers, the solidity compiler would inline the operations of the modifier in the called function. Using functions to perform checks would incur two JUMPI instructions, plus the stack setup, which could cost up to 40 gas units. Implementing this change would increase deployment cost, but would reduce the gas cost of the called functions. In the long run, using modifiers would be cheaper. The functions below can also be made more gas efficient by making them payable functions:

```solidity
file: main/contracts/types/LeftRight.sol

198  function toInt128(int256 self) internal pure returns (int128 selfAsInt128) {
        if (!((selfAsInt128 = int128(self)) == self)) revert Errors.CastingError();
    }

205   function toUint128(uint256 self) internal pure returns (uint128 selfAsUint128) {
        if (!((selfAsUint128 = uint128(self)) == self)) revert Errors.CastingError();
    }

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol#L198-L200

```solidity
file: main/contracts/libraries/Math.sol

172  function toUint128(uint256 toDowncast) internal pure returns (uint128 downcastedInt) {
        if ((downcastedInt = uint128(toDowncast)) != toDowncast) revert Errors.CastingError();
    }

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L172-L174

## [G-11] IF’s/require() statements that check input arguments should be at the top of the function

```solidity
file: main/contracts/SemiFungiblePositionManager.sol

417  if (amount0Owed > 0)
            SafeTransferLib.safeTransferFrom(
                decoded.poolFeatures.token0,
                decoded.payer,
                msg.sender,
                amount0Owed
            );
424  if (amount1Owed > 0)
            SafeTransferLib.safeTransferFrom(
                decoded.poolFeatures.token1,
                decoded.payer,
                msg.sender,
                amount1Owed
            );

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L417-L430

## [G-12] Use assembly in place of abi.decode to extract calldata values more efficiently

Instead of using abi.decode, we can use assembly to decode our desired calldata values directly. This will allow us to avoid decoding calldata values that we will not use. we can use assembly to decode our desired calldata values directly. This will allow us to avoid decoding calldata values that we will not use

```solidity
file:  main/contracts/SemiFungiblePositionManager.sol

413    CallbackLib.CallbackData memory decoded = abi.decode(data, (CallbackLib.CallbackData));

447    CallbackLib.CallbackData memory decoded = abi.decode(data, (CallbackLib.CallbackData));

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L413

## [G-13] Save loop calls

Instead of calling s_accountLiquidity[positionKey_to] 2 times in each loop for fetching data, it can be saved as a variable.

```solidity
file:  main/contracts/SemiFungiblePositionManager.sol

583    for (uint256 leg = 0; leg < numLegs; ) {
            // for this leg index: extract the liquidity chunk: a 256bit word containing the liquidity amount and upper/lower tick
            // @dev see `contracts/types/LiquidityChunk.sol`
            uint256 liquidityChunk = PanopticMath.getLiquidityChunk(
                id,
                leg,
                uint128(amount),
                univ3pool.tickSpacing()
            );

            //construct the positionKey for the from and to addresses
            bytes32 positionKey_from = keccak256(
                abi.encodePacked(
                    address(univ3pool),
                    from,
                    id.tokenType(leg),
                    liquidityChunk.tickLower(),
                    liquidityChunk.tickUpper()
                )
            );
            bytes32 positionKey_to = keccak256(
                abi.encodePacked(
                    address(univ3pool),
                    to,
                    id.tokenType(leg),
                    liquidityChunk.tickLower(),
                    liquidityChunk.tickUpper()
                )
            );

            // Revert if recipient already has that position
            if (
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
            unchecked {
                ++leg;
            }
        }
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L583-L634


## [G-14] Use hardcode address instead address(this)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.

References:

https://book.getfoundry.sh/reference/forge-std/compute-create-address https://twitter.com/transmissions11/status/1518507047943245824


```solidity
file: main/contracts/multicall/Multicall.sol

15    (bool success, bytes memory result) = address(this).delegatecall(data[i]);

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/multicall/Multicall.sol#L15

```solidity
file:  main/contracts/SemiFungiblePositionManager.sol

1106   address(this),

1148    address(this),

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L1106

## [G-15] Use the existing Local variable/global variable when equal to a state variable to avoid reading from state

### Local variable fromLiq should be used instead of reading s_accountLiquidity[positionKey_from]

```solidity
file:   main/contracts/SemiFungiblePositionManager.sol

620         uint256 fromLiq = s_accountLiquidity[positionKey_from];
            if (fromLiq.rightSlot() != liquidityChunk.liquidity()) revert Errors.TransferFailed();

            int256 fromBase = s_accountFeesBase[positionKey_from];

            //update+store liquidity and fee values between accounts
            s_accountLiquidity[positionKey_to] = fromLiq;
            s_accountLiquidity[positionKey_from] = 0;

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L620-L627

### Local variable fromBase should be used instead of reading s_accountFeesBase[positionKey_from]

```solidity
file:  main/contracts/SemiFungiblePositionManager.sol

623   int256 fromBase = s_accountFeesBase[positionKey_from];

            //update+store liquidity and fee values between accounts
            s_accountLiquidity[positionKey_to] = fromLiq;
            s_accountLiquidity[positionKey_from] = 0;

            s_accountFeesBase[positionKey_to] = fromBase;
            s_accountFeesBase[positionKey_from] = 0;
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L623-L630

### Local variable currentLiquidity should be used instead of reading s_accountLiquidity[positionKey]

```solidity
file: main/contracts/SemiFungiblePositionManager.sol

959            uint256 currentLiquidity = s_accountLiquidity[positionKey]; //cache

        unchecked {
            // did we have liquidity already deployed in Uniswap for this chunk range from some past mint?

            // s_accountLiquidity is a LeftRight. The right slot represents the liquidity currently sold (added) in the AMM owned by the user
            // the left slot represents the amount of liquidity currently bought (removed) that has been removed from the AMM - the user owes it to a seller
            // the reason why it is called "removedLiquidity" is because long options are created by removing -ie.short selling LP positions
            uint128 startingLiquidity = currentLiquidity.rightSlot();
            uint128 removedLiquidity = currentLiquidity.leftSlot();
            uint128 chunkLiquidity = _liquidityChunk.liquidity();

            if (isLong == 0) {
                // selling/short: so move from msg.sender *to* uniswap
                // we're minting more liquidity in uniswap: so add the incoming liquidity chunk to the existing liquidity chunk
                updatedLiquidity = startingLiquidity + chunkLiquidity;

                /// @dev If the isLong flag is 0=short but the position was burnt, then this is closing a long position
                /// @dev so the amount of short liquidity should decrease.
                if (_isBurn) {
                    removedLiquidity -= chunkLiquidity;
                }
            } else {
                // the _leg is long (buying: moving *from* uniswap to msg.sender)
                // so we seek to move the incoming liquidity chunk *out* of uniswap - but was there sufficient liquidity sitting in uniswap
                // in the first place?
                if (startingLiquidity < chunkLiquidity) {
                    // the amount we want to move (liquidityChunk.legLiquidity()) out of uniswap is greater than
                    // what the account that owns the liquidity in uniswap has (startingLiquidity)
                    // we must ensure that an account can only move its own liquidity out of uniswap
                    // so we revert in this case
                    revert Errors.NotEnoughLiquidity();
                } else {
                    // we want to move less than what already sits in uniswap, no problem:
                    updatedLiquidity = startingLiquidity - chunkLiquidity;
                }

                /// @dev If the isLong flag is 1=long and the position is minted, then this is opening a long position
                /// @dev so the amount of short liquidity should increase.
                if (!_isBurn) {
                    removedLiquidity += chunkLiquidity;
                }
            }

            // update the starting liquidity for this position for next time around
            s_accountLiquidity[positionKey] = uint256(0).toLeftSlot(removedLiquidity).toRightSlot(
                updatedLiquidity
            );
        }

        // track how much liquidity we need to collect from uniswap
        // add the fees that accumulated in uniswap within the liquidityChunk:
        {
            /** if the position is NOT long (selling a put or a call), then _mintLiquidity to move liquidity
                from the msg.sender to the uniswap v3 pool:
                Selling(isLong=0): Mint chunk of liquidity in Uniswap (defined by upper tick, lower tick, and amount)
                       ┌─────────────────────────────────┐
                ▲     ┌▼┐ liquidityChunk                 │
                │  ┌──┴─┴──┐                         ┌───┴──┐
                │  │       │                         │      │
                └──┴───────┴──►                      └──────┘
                   Uniswap v3                      msg.sender
            
             else: the position is long (buying a put or a call), then _burnLiquidity to remove liquidity from univ3
                Buying(isLong=1): Burn in Uniswap
                       ┌─────────────────┐
                ▲     ┌┼┐                │
                │  ┌──┴─┴──┐         ┌───▼──┐
                │  │       │         │      │
                └──┴───────┴──►      └──────┘
                    Uniswap v3      msg.sender 
            */
            _moved = isLong == 0
                ? _mintLiquidity(_liquidityChunk, _univ3pool)
                : _burnLiquidity(_liquidityChunk, _univ3pool); // from msg.sender to Uniswap
            // add the moved liquidity chunk to amount we need to collect from uniswap:

            // Is this _leg ITM?
            // if tokenType is 1, and we transacted some token0: then this leg is ITM!
            if (_tokenType == 1) {
                // extract amount moved out of UniswapV3 pool
                _itmAmounts = _itmAmounts.toRightSlot(_moved.rightSlot());
            }
            // if tokenType is 0, and we transacted some token1: then this leg is ITM
            if (_tokenType == 0) {
                // Add this in-the-money amount transacted.
                _itmAmounts = _itmAmounts.toLeftSlot(_moved.leftSlot());
            }
        }

        // if there was liquidity at that tick before the transaction, collect any accumulated fees
        if (currentLiquidity.rightSlot() > 0) {
            _totalCollected = _collectAndWritePositionData(
                _liquidityChunk,
                _univ3pool,
                currentLiquidity,
                positionKey,
                _moved,
                isLong
            );
        }

        // position has been touched, update s_accountFeesBase with the latest values from the pool.positions
        s_accountFeesBase[positionKey] = _getFeesBase(
            _univ3pool,
            updatedLiquidity,
            _liquidityChunk
        );
    }
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L959-L1067

## [G-16] A modifier used only once and not being inherited should be inlined to save gas

```solidity
File: contracts/SemiFungiblePositionManager.sol

306      modifier ReentrancyLock(uint64 poolId) {
          // check if the pool is already locked
          // init lock if not
          beginReentrancyLock(poolId);
  
          // execute function
          _;
  
          // remove lock
          endReentrancyLock(poolId);
      }

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L306-L316

## [G-17] Use assembly for loops

In the following instances, assembly is used for more gas efficient loops. The only memory slots that are manually used in the loops are scratch space (0x00-0x20), the free memory pointer (0x40), and the zero slot (0x60). This allows us to avoid using the free memory pointer to allocate new memory, which may result in memory expansion costs.

```solidity
file: main/contracts/SemiFungiblePositionManager.sol

550  for (uint256 i = 0; i < ids.length; ) {
            registerTokenTransfer(from, to, ids[i], amounts[i]);
            unchecked {
                ++i;
            }
        }


```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L550-L555


```solidity
file: main/contracts/tokens/ERC1155Minimal.sol

141    for (uint256 i = 0; i < ids.length; ) {
            id = ids[i];
            amount = amounts[i];

            balanceOf[from][id] -= amount;

            // balance will never overflow
            unchecked {
                balanceOf[to][id] += amount;
            }

            // An array can't have a total length
            // larger than the max uint256 value.
            unchecked {
                ++i;
            }
        }

186        for (uint256 i = 0; i < owners.length; ++i) {
                balances[i] = balanceOf[owners[i]][ids[i]];
            }

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L141-157


## [G-18] Use assembly to check for address(0)

```solidity
file: main/contracts/SemiFungiblePositionManager.sol
 
356   if (address(univ3pool) == address(0)) revert Errors.UniswapPoolNotInitialized();

370   while (address(s_poolContext[poolId].pool) != address(0)) 

683   if (univ3pool == IUniswapV3Pool(address(0))) revert Errors.UniswapPoolNotInitialized();

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L356


```solidity
file:  main/contracts/tokens/ERC1155Minimal.sol

220    emit TransferSingle(msg.sender, address(0), to, id, amount);

224    ERC1155Holder(to).onERC1155Received(msg.sender, address(0), id, amount, "") !=

239    emit TransferSingle(msg.sender, from, address(0), id, amount);

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L220

## [G-19] Amounts should be checked for 0 before calling a transfer

Checking non-zero transfer values can avoid an expensive external call and save gas. While this is done at some places, it’s not consistently done in the solution. I suggest adding a non-zero-value check here:

```solidity
file: main/contracts/SemiFungiblePositionManager.sol

460   SafeTransferLib.safeTransferFrom(token, decoded.payer, msg.sender, amountToPay);

569   registerTokenTransfer(from, to, id, amount);

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L460


```solidity
file: main/contracts/tokens/ERC1155Minimal.sol

106   afterTokenTransfer(from, to, id, amount);

159   afterTokenTransfer(from, to, ids, amounts);

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L106

## [G-20] abi.encode() is less efficient than abi.encodePacked()

Consider changing it if possible.

```solidity
file: main/contracts/SemiFungiblePositionManager.sol

760    data = abi.encode(

1134   bytes memory mintdata = abi.encode(

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L760

## [G-21] Assigning keccak operations to constant variables results in extra gas costs

"constants" expressions are expressions. As such, keccak assigned to a constant variable are re-computed at each use of the variable, which will consume gas unnecessarily. To save gas, Changing the variable from constant to immutable will make the computation run only once and therefore save gas.

```solidity
file: main/contracts/libraries/Constants.sol

25    bytes32 internal constant V3POOL_INIT_CODE_HASH =
        keccak256(

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Constants.sol#L25-L26

## [G‑22] >= costs less gas than >

The compiler uses opcodes GT and ISZERO for solidity code that uses >, but only requires LT for >=, which saves 3 gas

```solidity
file:

687    if (tickLimitLow > tickLimitHigh) {

917    if (amount0 > uint128(type(int128).max) || amount1 > uint128(type(int128).max))

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L687


```solidity
file: main/contracts/libraries/Math.sol

41   if (absTick > uint256(int256(Constants.MAX_V3POOL_TICK))) revert Errors.InvalidTick();

216  require(denominator > prod1);

373  require(2 ** 96 > prod1);

435  require(2 ** 128 > prod1);

497   require(2 ** 192 > prod1);

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L41

## [G-23] Declare the variables outside the loop

Per iterations saves 26 GAS

```solidity
file:  main/contracts/SemiFungiblePositionManager.sol

583       for (uint256 leg = 0; leg < numLegs; ) {
            // for this leg index: extract the liquidity chunk: a 256bit word containing the liquidity amount and upper/lower tick
            // @dev see `contracts/types/LiquidityChunk.sol`
            uint256 liquidityChunk = PanopticMath.getLiquidityChunk(
                id,
                leg,
                uint128(amount),
                univ3pool.tickSpacing()
            );

            //construct the positionKey for the from and to addresses
            bytes32 positionKey_from = keccak256(
                abi.encodePacked(
                    address(univ3pool),
                    from,
                    id.tokenType(leg),
                    liquidityChunk.tickLower(),
                    liquidityChunk.tickUpper()
                )
            );
            bytes32 positionKey_to = keccak256(
                abi.encodePacked(
                    address(univ3pool),
                    to,
                    id.tokenType(leg),
                    liquidityChunk.tickLower(),
                    liquidityChunk.tickUpper()
                )
            );

            // Revert if recipient already has that position
            if (
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
            unchecked {
                ++leg;
            }
        }

860          for (uint256 leg = 0; leg < numLegs; ) {
            int256 _moved;
            int256 _itmAmounts;
            int256 _totalCollected;

            {
                // cache the univ3pool, tokenId, isBurn, and _positionSize variables to get rid of stack too deep error
                IUniswapV3Pool _univ3pool = univ3pool;
                uint256 _tokenId = tokenId;
                bool _isBurn = isBurn;
                uint128 _positionSize = positionSize;
                uint256 _leg;

                unchecked {
                    // Reverse the order of the legs if this call is burning a position (LIFO)
                    // We loop in reverse order if burning a position so that any dependent long liquidity is returned to the pool first,
                    // allowing the corresponding short liquidity to be removed
                    _leg = _isBurn ? numLegs - leg - 1 : leg;
                }

                // for this _leg index: extract the liquidity chunk: a 256bit word containing the liquidity amount and upper/lower tick
                // @dev see `contracts/types/LiquidityChunk.sol`
                uint256 liquidityChunk = PanopticMath.getLiquidityChunk(
                    _tokenId,
                    _leg,
                    _positionSize,
                    _univ3pool.tickSpacing()
                );

                (_moved, _itmAmounts, _totalCollected) = _createLegInAMM(
                    _univ3pool,
                    _tokenId,
                    _leg,
                    liquidityChunk,
                    _isBurn
                );

                unchecked {
                    // increment accumulators of the upper bound on tokens contained across all legs of the position at any given tick
                    amount0 += Math.getAmount0ForLiquidity(liquidityChunk);

                    amount1 += Math.getAmount1ForLiquidity(liquidityChunk);
                }
            }

            totalMoved = totalMoved.add(_moved);
            itmAmounts = itmAmounts.add(_itmAmounts);
            totalCollected = totalCollected.add(_totalCollected);

            unchecked {
                ++leg;
            }
        }
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L583-L634


```solidity
file:  main/contracts/types/TokenId.sol

468            for (uint256 i = 0; i < 4; ++i) {
                if (self.optionRatio(i) == 0) {
                    // final leg in this position identified;
                    // make sure any leg above this are zero as well
                    // (we don't allow gaps eg having legs 1 and 4 active without 2 and 3 is not allowed)
                    if ((self >> (64 + 48 * i)) != 0) revert Errors.InvalidTokenIdParameter(1);

                    break; // we are done iterating over potential legs
                }
                // now validate this ith leg in the position:

                // The width cannot be 0; the minimum is 1
                if ((self.width(i) == 0)) revert Errors.InvalidTokenIdParameter(5);
                // Strike cannot be MIN_TICK or MAX_TICK
                if (
                    (self.strike(i) == Constants.MIN_V3POOL_TICK) ||
                    (self.strike(i) == Constants.MAX_V3POOL_TICK)
                ) revert Errors.InvalidTokenIdParameter(4);

                // In the following, we check whether the risk partner of this leg is itself
                // or another leg in this position.
                // Handles case where riskPartner(i) != i ==> leg i has a risk partner that is another leg
                uint256 riskPartnerIndex = self.riskPartner(i);
                if (riskPartnerIndex != i) {
                    // Ensures that risk partners are mutual
                    if (self.riskPartner(riskPartnerIndex) != i)
                        revert Errors.InvalidTokenIdParameter(3);

                    // Ensures that risk partners have 1) the same asset, and 2) the same ratio
                    if (
                        (self.asset(riskPartnerIndex) != self.asset(i)) ||
                        (self.optionRatio(riskPartnerIndex) != self.optionRatio(i))
                    ) revert Errors.InvalidTokenIdParameter(3);

                    // long/short status of associated legs
                    uint256 isLong = self.isLong(i);
                    uint256 isLongP = self.isLong(riskPartnerIndex);

                    // token type status of associated legs (call/put)
                    uint256 tokenType = self.tokenType(i);
                    uint256 tokenTypeP = self.tokenType(riskPartnerIndex);

                    // if the position is the same i.e both long calls, short put's etc.
                    // then this is a regular position, not a defined risk position
                    if ((isLong == isLongP) && (tokenType == tokenTypeP))
                        revert Errors.InvalidTokenIdParameter(4);

                    // if the two token long-types and the tokenTypes are both different (one is a short call, the other a long put, e.g.), this is a synthetic position
                    // A synthetic long or short is more capital efficient than each leg separated because the long+short premia accumulate proportionally
                    if ((isLong != isLongP) && (tokenType != tokenTypeP))
                        revert Errors.InvalidTokenIdParameter(5);
                }
            } // end for loop over legs
```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L468-L520

## [G-24] Don’t cache calls that are only used once

```solidity
file: main/contracts/libraries/Math.sol

122   uint160 lowPriceX96 = getSqrtRatioAtTick(liquidityChunk.tickLower());

123   uint160 highPriceX96 = getSqrtRatioAtTick(liquidityChunk.tickUpper());

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol#L122

## [G-25] Loop best practice to save gas

```solidity
function Plusi() public view {
		for(uint i=0; i<10;) {
			console.log("Number ==",i);
			unchecked{
				++i;
			}
		}
	}
best practice
-----------------------------------------------------
for (uint i = 0; i < length; i = unchecked_inc(i)) {
    // do something that doesn't change the value of i
}
function unchecked_inc(uint i) returns (uint) {
    unchecked {
        return i + 1;
    }
}
```

```solidity
file: main/contracts/tokens/ERC1155Minimal.sol

187    for (uint256 i = 0; i < owners.length; ++i) {

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L187


```solidity
file:  main/contracts/types/TokenId.sol
 
468    for (uint256 i = 0; i < 4; ++i) {

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L468


## [G‑26] Counting down in for statements is more gas efficient

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

by changing this logic you can save 12171 gas per one for loop 

Tools used Remix

```solidity
file: main/contracts/SemiFungiblePositionManager.sol

550   for (uint256 i = 0; i < ids.length; ) {

583   for (uint256 leg = 0; leg < numLegs; ) {

860   for (uint256 leg = 0; leg < numLegs; ) {

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L550


```solidity
file: main/contracts/tokens/ERC1155Minimal.sol

14    for (uint256 i = 0; i < data.length; ) {

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L14


```solidity
file: main/contracts/tokens/ERC1155Minimal.sol

141   for (uint256 i = 0; i < ids.length; ) {

187   for (uint256 i = 0; i < owners.length; ++i) {

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L141


```solidity
file:  main/contracts/types/TokenId.sol

468    for (uint256 i = 0; i < 4; ++i) {

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol#L468


### Test Code

```solidity
contract GasTest is DSTest {
    Contract0 c0;
    Contract1 c1;
    function setUp() public {
        c0 = new Contract0();
        c1 = new Contract1();
    }
    function testGas() public {
        c0.AddNum();
        c1.AddNum();
    }
}
contract Contract0 {
    uint256 num = 3;
    function AddNum() public {
        uint256 _num = num;
        for(uint i=0;i<=9;i++){
            _num = _num +1;
        }
        num = _num;
    }
}
contract Contract1 {
    uint256 num = 3;
    function AddNum() public {
        uint256 _num = num;
        for(uint i=9;i>=0;i--){
            _num = _num +1;
        }
        num = _num;
    }
}
```

## [G-27] Create immutable variable to avoid an external call

Instead of performing an external call to get the root address each time _enableNode is invoked, we can perform this external call once in the constructor and store the root as an immutable variable. Doing this will save 1 external call each time _enableNode is invoked.

```solidity
file: main/contracts/SemiFungiblePositionManager.sol

580   IUniswapV3Pool univ3pool = s_poolContext[id.validate()].pool;

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L580

## [G-28] public functions not called by the contract should be declared external instead

when a function is declared as public, it is generated with an internal and an external interface. This means the function can be called both internally (within the contract) and externally (by other contracts or accounts). However, if a public function is never called internally and is only expected to be invoked externally, it is more gas-efficient to explicitly declare it as external.

```solidity
file: main/contracts/libraries/FeesCalc.sol

54    function calculateAMMSwapFeesLiquidityChunk(
        IUniswapV3Pool univ3pool,
        int24 currentTick,
        uint128 startingLiquidity,
        uint256 liquidityChunk
    ) public view returns (int256 feesEachToken) {

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/FeesCalc.sol#L54-L59


```solidity
file: main/contracts/multicall/Multicall.sol

12   function multicall(bytes[] calldata data) public payable returns (bytes[] memory results) {

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/multicall/Multicall.sol#L12


```solidity
file: main/contracts/libraries/SafeTransferLib.sol

77   function setApprovalForAll(address operator, bool approved) public {

90       function safeTransferFrom(
        address from,
        address to,
        uint256 id,
        uint256 amount,
        bytes calldata data
    ) public {

128   function safeBatchTransferFrom(
        address from,
        address to,
        uint256[] calldata ids,
        uint256[] calldata amounts,
        bytes calldata data
    ) public virtual {

178  function balanceOfBatch(
        address[] calldata owners,
        uint256[] calldata ids
    ) public view returns (uint256[] memory balances) {

200  function supportsInterface(bytes4 interfaceId) public pure returns (bool) {

```
https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/SafeTransferLib.sol#L77