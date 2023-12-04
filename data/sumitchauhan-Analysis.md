Medium risk issues:-
1. Divide before multiply:
TokenId.asTicks(uint256,uint256,int24) (contracts/SemiFungiblePositionManager.sol#681-710) performs a multiplication on the result of a division:
        - minTick = (Constants.MIN_V3POOL_TICK / tickSpacing) * tickSpacing (contracts/SemiFungiblePositionManager.sol)
TokenId.asTicks(uint256,uint256,int24) (contracts/SemiFungiblePositionManager.sol#681-710) performs a multiplication on the result of a division:
        - maxTick = (Constants.MAX_V3POOL_TICK / tickSpacing) * tickSpacing (contracts/SemiFungiblePositionManager.sol)
Math.mulDiv(uint256,uint256,uint256) (contracts/SemiFungiblePositionManager.sol) performs a multiplication on the result of a division:
        - denominator = denominator / twos (contracts/SemiFungiblePositionManager.sol)
        - inv = (3 * denominator) ^ 2 (contracts/SemiFungiblePositionManager.sol)
Math.mulDiv(uint256,uint256,uint256) (contracts/SemiFungiblePositionManager.sol) performs a multiplication on the result of a division:
        - denominator = denominator / twos (contracts/SemiFungiblePositionManager.sol)
        - inv *= 2 - denominator * inv (contracts/SemiFungiblePositionManager.sol)
Math.mulDiv(uint256,uint256,uint256) (contracts/SemiFungiblePositionManager.sol) performs a multiplication on the result of a division:
        - denominator = denominator / twos (contracts/SemiFungiblePositionManager.sol)
        - inv *= 2 - denominator * inv (contracts/SemiFungiblePositionManager.sol)
Math.mulDiv(uint256,uint256,uint256) (contracts/SemiFungiblePositionManager.sol) performs a multiplication on the result of a division:
        - denominator = denominator / twos (contracts/SemiFungiblePositionManager.sol)
        - inv *= 2 - denominator * inv (contracts/SemiFungiblePositionManager.sol)
Math.mulDiv(uint256,uint256,uint256) (contracts/SemiFungiblePositionManager.sol) performs a multiplication on the result of a division:
        - denominator = denominator / twos (contracts/SemiFungiblePositionManager.sol)
        - inv *= 2 - denominator * inv (contracts/SemiFungiblePositionManager.sol)
Math.mulDiv(uint256,uint256,uint256) (contracts/SemiFungiblePositionManager.sol) performs a multiplication on the result of a division:
        - denominator = denominator / twos (contracts/SemiFungiblePositionManager.sol)
        - inv *= 2 - denominator * inv (contracts/SemiFungiblePositionManager.sol)
Math.mulDiv(uint256,uint256,uint256) (contracts/SemiFungiblePositionManager.sol) performs a multiplication on the result of a division:
        - denominator = denominator / twos (contracts/SemiFungiblePositionManager.sol)
        - inv *= 2 - denominator * inv (contracts/SemiFungiblePositionManager.sol)
Math.mulDiv(uint256,uint256,uint256) (contracts/SemiFungiblePositionManager.sol) performs a multiplication on the result of a division:
        - prod0 = prod0 / twos (contracts/SemiFungiblePositionManager.sol)
        - result = prod0 * inv (contracts/SemiFungiblePositionManager.sol)

2. Reentrancy:
Reentrancy in SemiFungiblePositionManager._createLegInAMM(IUniswapV3Pool,uint256,uint256,uint256,bool) (contracts/SemiFungiblePositionManager.sol):
        External calls:
        - _totalCollected = _collectAndWritePositionData(_liquidityChunk,_univ3pool,currentLiquidity,positionKey,_moved,isLong) (contracts/SemiFungiblePositionManager.sol)
                - (receivedAmount0,receivedAmount1) = univ3pool.collect(msg.sender,liquidityChunk.tickLower(),liquidityChunk.tickUpper(),uint128(amountToCollect.rightSlot()),uint128(amountToCollect.leftSlot())) (contracts/SemiFungiblePositionManager.sol)
        - _moved = _mintLiquidity(_liquidityChunk,_univ3pool) (contracts/SemiFungiblePositionManager.sol)
                - (amount0,amount1) = univ3pool.mint(address(this),liquidityChunk.tickLower(),liquidityChunk.tickUpper(),liquidityChunk.liquidity(),mintdata) (contracts/SemiFungiblePositionManager.sol)
        - _moved = _burnLiquidity(_liquidityChunk,_univ3pool) (contracts/SemiFungiblePositionManager.sol)
                - (amount0,amount1) = univ3pool.burn(liquidityChunk.tickLower(),liquidityChunk.tickUpper(),liquidityChunk.liquidity()) (contracts/SemiFungiblePositionManager.sol)
        State variables written after the call(s):
        - s_accountFeesBase[positionKey] = _getFeesBase(_univ3pool,updatedLiquidity,_liquidityChunk) (contracts/SemiFungiblePositionManager.sol)
        SemiFungiblePositionManager.s_accountFeesBase (contracts/SemiFungiblePositionManager.sol) can be used in cross function reentrancies:
        - SemiFungiblePositionManager._collectAndWritePositionData(uint256,IUniswapV3Pool,uint256,bytes32,int256,uint256) (contracts/SemiFungiblePositionManager.sol)
        - SemiFungiblePositionManager._createLegInAMM(IUniswapV3Pool,uint256,uint256,uint256,bool) (contracts/SemiFungiblePositionManager.sol)
        - SemiFungiblePositionManager.getAccountFeesBase(address,address,uint256,int24,int24) (contracts/SemiFungiblePositionManager.sol)
        - SemiFungiblePositionManager.getAccountPremium(address,address,uint256,int24,int24,int24,uint256) (contracts/SemiFungiblePositionManager.sol)
        - SemiFungiblePositionManager.registerTokenTransfer(address,address,uint256,uint256) (contracts/SemiFungiblePositionManager.sol)
Reentrancy in SemiFungiblePositionManager._collectAndWritePositionData(uint256,IUniswapV3Pool,uint256,bytes32,int256,uint256) (contracts/SemiFungiblePositionManager.sol):
        External calls:
        - (receivedAmount0,receivedAmount1) = univ3pool.collect(msg.sender,liquidityChunk.tickLower(),liquidityChunk.tickUpper(),uint128(amountToCollect.rightSlot()),uint128(amountToCollect.leftSlot())) (contracts/SemiFungiblePositionManager.sol)
        State variables written after the call(s):
        - _updateStoredPremia(positionKey,currentLiquidity,collectedOut) (contracts/SemiFungiblePositionManager.sol)
                - s_accountPremiumGross[positionKey] = s_accountPremiumGross[positionKey].add(deltaPremiumGross) (contracts/SemiFungiblePositionManager.sol)
        - _updateStoredPremia(positionKey,currentLiquidity,collectedOut) (contracts/SemiFungiblePositionManager.sol)
                - s_accountPremiumOwed[positionKey] = s_accountPremiumOwed[positionKey].add(deltaPremiumOwed) (contracts/SemiFungiblePositionManager.sol)
Reentrancy in SemiFungiblePositionManager.mintTokenizedPosition(uint256,uint128,int24,int24) (contracts/SemiFungiblePositionManager.sol):
        External calls:
        - _mint(msg.sender,tokenId,positionSize) (contracts/SemiFungiblePositionManager.sol#3085)
                - ERC1155Holder(to).onERC1155Received(msg.sender,address(0),id,amount,) != ERC1155Holder.onERC1155Received.selector (contracts/SemiFungiblePositionManager.sol)
        Event emitted after the call(s):
        - TokenizedPositionMinted(msg.sender,tokenId,positionSize) (contracts/SemiFungiblePositionManager.sol)

3. Uninitialized local variables:
SemiFungiblePositionManager._validateAndForwardToAMM(uint256,uint128,int24,int24,bool).swapAtMint (contracts/SemiFungiblePositionManager.sol) is a local variable never initialized
SemiFungiblePositionManager._createLegInAMM(IUniswapV3Pool,uint256,uint256,uint256,bool).updatedLiquidity (contracts/SemiFungiblePositionManager.sol) is a local variable never initialized

4. Unused return:
FeesCalc._getAMMSwapFeesPerLiquidityCollected(IUniswapV3Pool,int24,int24,int24) (contracts/SemiFungiblePositionManager.sol) ignores return value by (lowerOut0,lowerOut1) = univ3pool.ticks(tickLower) (contracts/SemiFungiblePositionManager.sol)
FeesCalc._getAMMSwapFeesPerLiquidityCollected(IUniswapV3Pool,int24,int24,int24) (contracts/SemiFungiblePositionManager.sol) ignores return value by (upperOut0,upperOut1) = univ3pool.ticks(tickUpper) (contracts/SemiFungiblePositionManager.sol)
SemiFungiblePositionManager._validateAndForwardToAMM(uint256,uint128,int24,int24,bool) (contracts/SemiFungiblePositionManager.sol) ignores return value by (None,newTick,None,None,None,None,None) = univ3pool.slot0() (contracts/SemiFungiblePositionManager.sol)
SemiFungiblePositionManager.swapInAMM(IUniswapV3Pool,int256) (contracts/SemiFungiblePositionManager.sol) ignores return value by (sqrtPriceX96) = _univ3pool.slot0() (contracts/SemiFungiblePositionManager.sol)
SemiFungiblePositionManager._getFeesBase(IUniswapV3Pool,uint128,uint256) (contracts/SemiFungiblePositionManager.sol) ignores return value by (feeGrowthInside0LastX128,feeGrowthInside1LastX128) = univ3pool.positions(keccak256(bytes)(abi.encodePacked(address(this),liquidityChunk.tickLower(),liquidityChunk.tickUpper()))) (contracts/SemiFungiblePositionManager.sol)


Informational issues or gas optimizations:-
1. `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables:
  SemiFungiblePositionManager.sol => balanceOf[from][id] -= amount;
  SemiFungiblePositionManager.sol => balanceOf[to][id] += amount;
  SemiFungiblePositionManager.sol => balanceOf[from][id] -= amount;
  SemiFungiblePositionManager.sol => balanceOf[to][id] += amount;
  SemiFungiblePositionManager.sol => balanceOf[to][id] += amount;
  SemiFungiblePositionManager.sol => balanceOf[from][id] -= amount;
  SemiFungiblePositionManager.sol => s_accountPremiumOwed += feeGrowthX128 * R * (1 + ν*R/N) / R
  SemiFungiblePositionManager.sol => += feeGrowthX128 * (T - R + ν*R)/N
  SemiFungiblePositionManager.sol => += feeGrowthX128 * T/N * (1 - R/T + ν*R/T)
  SemiFungiblePositionManager.sol => s_accountPremiumOwed += feesCollected * T/N^2 * (1 - R/T + ν*R/T)          (Eqn 3)
  SemiFungiblePositionManager.sol => s_accountPremiumGross += feesCollected * T/N^2 * (1 - R/T + ν*R^2/T^2)       (Eqn 4)
  SemiFungiblePositionManager.sol => amount0 += Math.getAmount0ForLiquidity(liquidityChunk);
  SemiFungiblePositionManager.sol => amount1 += Math.getAmount1ForLiquidity(liquidityChunk);
  SemiFungiblePositionManager.sol => removedLiquidity -= chunkLiquidity;
  SemiFungiblePositionManager.sol => removedLiquidity += chunkLiquidity;

2. Use scientific notation (e.g. `1e18`) rather than exponentiation (e.g. `10**18`):
  SemiFungiblePositionManager.sol => if (optionRatios < 2 ** 64) {
  SemiFungiblePositionManager.sol => } else if (optionRatios < 2 ** 112) {
  SemiFungiblePositionManager.sol => } else if (optionRatios < 2 ** 160) {
  SemiFungiblePositionManager.sol => } else if (optionRatios < 2 ** 208) {
  SemiFungiblePositionManager.sol => if (optionRatios < 2 ** 64) {
  SemiFungiblePositionManager.sol => } else if (optionRatios < 2 ** 112) {
  SemiFungiblePositionManager.sol => } else if (optionRatios < 2 ** 160) {
  SemiFungiblePositionManager.sol => } else if (optionRatios < 2 ** 208) {
  SemiFungiblePositionManager.sol => inv *= 2 - denominator * inv; // inverse mod 2**8
  SemiFungiblePositionManager.sol => inv *= 2 - denominator * inv; // inverse mod 2**16
  SemiFungiblePositionManager.sol => inv *= 2 - denominator * inv; // inverse mod 2**32
  SemiFungiblePositionManager.sol => inv *= 2 - denominator * inv; // inverse mod 2**64
  SemiFungiblePositionManager.sol => inv *= 2 - denominator * inv; // inverse mod 2**128
  SemiFungiblePositionManager.sol => inv *= 2 - denominator * inv; // inverse mod 2**256
  SemiFungiblePositionManager.sol => require(2 ** 64 > prod1);
  SemiFungiblePositionManager.sol => prod0 |= prod1 * 2 ** 192;
  SemiFungiblePositionManager.sol => require(2 ** 96 > prod1);
  SemiFungiblePositionManager.sol => prod0 |= prod1 * 2 ** 160;
  SemiFungiblePositionManager.sol => require(2 ** 128 > prod1);
  SemiFungiblePositionManager.sol => prod0 |= prod1 * 2 ** 128;
  SemiFungiblePositionManager.sol => require(2 ** 192 > prod1);
  SemiFungiblePositionManager.sol => prod0 |= prod1 * 2 ** 64;
  SemiFungiblePositionManager.sol => .mulDiv(Math.absUint(amount), 2 ** 192, uint256(sqrtPriceX96) ** 2)
  SemiFungiblePositionManager.sol => 2 ** 128,
  SemiFungiblePositionManager.sol => s_AddrToPoolIdData[univ3pool] = uint256(poolId) + 2 ** 255;
  SemiFungiblePositionManager.sol => .mulDiv(collected0, totalLiquidity * 2 ** 64, netLiquidity ** 2)
  SemiFungiblePositionManager.sol => .mulDiv(collected1, totalLiquidity * 2 ** 64, netLiquidity ** 2)
  SemiFungiblePositionManager.sol => uint256 numerator = netLiquidity + (removedLiquidity / 2 ** VEGOID);

3. Use multiple `require()` and `if` statements instead of `&&`:
  SemiFungiblePositionManager.sol=> if ((isLong == isLongP) && (tokenType == tokenTypeP))
  SemiFungiblePositionManager.sol=> if ((isLong != isLongP) && (tokenType != tokenTypeP))
  SemiFungiblePositionManager.sol=> if ((itmAmounts != 0) && (swapAtMint)) {
  SemiFungiblePositionManager.sol => if ((itm0 != 0) && (itm1 != 0)) {



### Time spent:
8 hours