# G-01. Using a positive conditional flow to save a NOT opcode #

Estimated savings: 3 gas per Instance.

Reference : https://code4rena.com/reports/2023-07-basin#g-13-using-a-positive-conditional-flow-to-save-a-not-opcode

Instances : 
```
file : contracts/multicall
/Multicall.sol

17 :             if (!success) {
```
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/multicall/Multicall.sol#L17
```
file : contracts/libraries
/SafeTransferLib.sol

40 :         if (!success) revert Errors.TransferFailed();
```
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/SafeTransferLib.sol#L40 
```
file : contracts
/SemiFungiblePositionManager.sol

998 :                 if (!_isBurn) {
```
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L998
```
file : contracts/types
/LeftRight.sol

199 :         if (!((selfAsInt128 = int128(self)) == self)) revert Errors.CastingError();

206 :         if (!((selfAsUint128 = uint128(self)) == self)) revert Errors.CastingError();
```
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/LeftRight.sol#L199

https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/LeftRight.sol#L206
```
file : contracts/tokens
/ERC1155Minimal.sol

97 :         if (!(msg.sender == from || isApprovedForAll[from][msg.sender])) revert NotAuthorized();

135 :         if (!(msg.sender == from || isApprovedForAll[from][msg.sender])) revert NotAuthorized();
```
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/tokens/ERC1155Minimal.sol#L97

https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/tokens/ERC1155Minimal.sol#L135

# G-02. OR in if-condition can be rewritten to two single if conditions #

Refactoring the if-condition in a way it won’t be containing the || operator will save more gas.

Reference : https://code4rena.com/reports/2023-08-shell#g-05-or-in-if-condition-can-be-rewritten-to-two-single-if-conditions

Instances : 
```
file : contracts
/SemiFungiblePositionManager.sol

614 - 617 :             if (
                (s_accountLiquidity[positionKey_to] != 0) ||
                (s_accountFeesBase[positionKey_to] != 0)
            ) revert Errors.TransferFailed();

713 :         if ((newTick >= tickLimitHigh) || (newTick <= tickLimitLow)) revert Errors.PriceBoundFa

917 :         if (amount0 > uint128(type(int128).max) || amount1 > uint128(type(int128).max))
```
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L614C13-L617C14

https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L614C13-L617C14

https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L917
```
file : contracts/types
/TokenId.sol

482 -485 :                 if (
                    (self.strike(i) == Constants.MIN_V3POOL_TICK) ||
                    (self.strike(i) == Constants.MAX_V3POOL_TICK)
                ) revert Errors.InvalidTokenIdParameter(4);
				
497 - 500 :                     if (
                        (self.asset(riskPartnerIndex) != self.asset(i)) ||
                        (self.optionRatio(riskPartnerIndex) != self.optionRatio(i))
                    ) revert Errors.InvalidTokenIdParameter(3);
```
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/TokenId.sol#L482C16-L485C18

https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/TokenId.sol#L497C21-L500C21
```
file : /contracts/types
/LeftRight.sol

151 :             if (z < x || (uint128(z) < uint128(x))) revert Errors.UnderOverFlow();

167 :             if (left128 != left256 || right128 != right256) revert Errors.UnderOverFlow();

185 :             if (left128 != left256 || right128 != right256) revert Errors.UnderOverFlow();
```
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/LeftRight.sol#L151

https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/LeftRight.sol#L167

https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/LeftRight.sol#L185
```
file : contracts/tokens
/ERC1155Minimal.sol

97 :         if (!(msg.sender == from || isApprovedForAll[from][msg.sender])) revert NotAuthorized();

135 :         if (!(msg.sender == from || isApprovedForAll[from][msg.sender])) revert NotAuthorized();
```
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/tokens/ERC1155Minimal.sol#L97

https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/tokens/ERC1155Minimal.sol#L135

# G-03. Use constants instead of type(uintx).max #

type(uint120).max or type(uint112).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.

Reference : https://code4rena.com/reports/2023-08-goodentry#g-20-use-constants-instead-of-typeuintxmax

Instances : 
```
file : /contracts
/SemiFungiblePositionManager.sol

917 :         if (amount0 > uint128(type(int128).max) || amount1 > uint128(type(int128).max))

1390 :         if (atTick < type(int24).max) {
```
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L917

https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L1390
```
file : /contracts/types
/LeftRight.sol

213 :         if (self > uint256(type(int256).max)) revert Errors.CastingError();
```
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/LeftRight.sol#L213
```
file : /contracts/libraries
/Math.sol

86 :             if (tick > 0) sqrtR = type(uint256).max / sqrtR;
```
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/Math.sol#L86

# G-04. Use assembly for math (add, sub, mul, div) #

Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety. If using Solidity versions < 0.8.0 and you are using Safemath, you can gain significant gas savings by using assembly to calculate values and checking for overflow/underflow.

Reference : https://code4rena.com/reports/2023-07-basin#g-17-use-assembly-for-math-add-sub-mul-div

Instances : 
```
file : contracts
/SemiFungiblePositionManager.sol

384 :             s_AddrToPoolIdData[univ3pool] = uint256(poolId) + 2 ** 255;

1313 - 1316 :                     uint256 numerator = totalLiquidity ** 2 -
                        totalLiquidity *
                        removedLiquidity +
                        ((removedLiquidity ** 2) / 2 ** (VEGOID));
```
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L384

https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L1313C16-L1316C25
```
file : contracts/types
/TokenId.sol

95 :             return uint256((self >> (64 + legIndex * 48)) % 2);

105 :             return uint256((self >> (64 + legIndex * 48 + 1)) % 128);

115 :             return uint256((self >> (64 + legIndex * 48 + 8)) % 2);

125 :             return uint256((self >> (64 + legIndex * 48 + 9)) % 2);

141 :             return uint256((self >> (64 + legIndex * 48 + 10)) % 4);

151 :             return int24(int256(self >> (64 + legIndex * 48 + 12)));

162 :             return int24(int256((self >> (64 + legIndex * 48 + 36)) % 4096));

195 :             return self + (uint256(_asset % 2) << (64 + legIndex * 48));

210 :             return self + (uint256(_optionRatio % 128) << (64 + legIndex * 48 + 1));

226 :             return self + ((_isLong % 2) << (64 + legIndex * 48 + 8));

240 :             return self + (uint256(_tokenType % 2) << (64 + legIndex * 48 + 9));

254 :             return self + (uint256(_riskPartner % 4) << (64 + legIndex * 48 + 10));

268 :             return self + uint256((int256(_strike) & BITMASK_INT24) << (64 + legIndex * 48 + 12));

283 :             return self + (uint256(uint24(_width) % 4096) << (64 + legIndex * 48 + 36));

```


https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/TokenId.sol#L95

https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/TokenId.sol#L105

https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/TokenId.sol#L115

https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/TokenId.sol#L125

https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/TokenId.sol#L141

https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/TokenId.sol#L151

https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/TokenId.sol#L162

https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/TokenId.sol#L195

https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/TokenId.sol#L210

https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/TokenId.sol#L226

https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/TokenId.sol#L240

https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/TokenId.sol#L254

https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/TokenId.sol#L268

https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/TokenId.sol#L283

# G-05. Using bools for storage incurs overhead #

Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas) for the extra SLOAD, and to avoid Gsset (20000 gas) when changing from false to true, after having been true in the past.

Reference : https://code4rena.com/reports/2023-07-basin#g-11---using-bools-for-storage-incurs-overhead

Instances : 
```
file : contracts
/SemiFungiblePositionManager.sol

127 :     bool internal constant MINT = false;

128 :     bool internal constant BURN = true;
```
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L127

https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L128
```
file : contracts/multicall
/Multicall.sol

15 :             (bool success, bytes memory result) = address(this).delegatecall(data[i]);
```
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/multicall/Multicall.sol#L15

# G-06. abi.encode() is less efficient than abi.encodepacked() #

In terms of efficiency, abi.encodePacked() is generally considered to be more gas-efficient than abi.encode(), because it skips the step of adding function signatures and other metadata to the encoded data. However, this comes at the cost of reduced safety, as abi.encodePacked() does not perform any type checking or padding of data.

Reference : https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison

Instances : 
```
file : contracts
/SemiFungiblePositionManager.sol

760 - 766 :             data = abi.encode(
                CallbackLib.CallbackData({
                    poolFeatures: CallbackLib.PoolFeatures({
                        token0: _univ3pool.token0(),
                        token1: _univ3pool.token1(),
                        fee: _univ3pool.fee()
                    }),
					
1134 - 1140 :         bytes memory mintdata = abi.encode(
            CallbackLib.CallbackData({ // compute by reading values from univ3pool every time
                    poolFeatures: CallbackLib.PoolFeatures({
                        token0: univ3pool.token0(),
                        token1: univ3pool.token1(),
                        fee: univ3pool.fee()
                    }),
```					
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L760C10-L767C17
                 
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L1134C8-L1140C17
```
file : contracts/libraries
/CallbackLib.sol

43 :                                 keccak256(abi.encode(features)),
```
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/CallbackLib.sol#L43

# G-07. Use assembly in place of abi.decode to extract calldata values more efficiently #

Instead of using abi.decode, we can use assembly to decode our desired calldata values directly. This will allow us to avoid decoding calldata values that we will not use.

Reference : https://code4rena.com/reports/2023-08-goodentry#g-02-use-assembly-in-place-of-abidecode-to-extract-calldata-values-more-efficiently

Instances : 
```
file : /contracts
/SemiFungiblePositionManager.sol

413 :         CallbackLib.CallbackData memory decoded = abi.decode(data, (CallbackLib.CallbackData));

447 :         CallbackLib.CallbackData memory decoded = abi.decode(data, (CallbackLib.CallbackData));
```
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L413

https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L447

# G-08. Use hardcoded address instead of address(this) #

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.

Reference : https://code4rena.com/reports/2023-08-goodentry#g-10-use-hardcoded-address-instead-of-addressthis

Instances :
```
file : contracts
/SemiFungiblePositionManager.sol

1148 :             address(this),
```
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L1148
```
file : /contracts/multicall
/Multicall.sol

15 :             (bool success, bytes memory result) = address(this).delegatecall(data[i]);
```
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/multicall/Multicall.sol#L15


# G-09. Use else if instead of multiple if's #

Using else if instead of multiple if's saves gas 

Reference : https://tom-sol.notion.site/Use-else-if-instead-of-if-if-if-d5b7ab6a8d0e4ee2b0e4792bac1e15c8

Instances :
```
file : contracts/types
/TokenId.sol

442 - 452 :     function clearLeg(uint256 self, uint256 i) internal pure returns (uint256) {
        if (i == 0)
            return self & 0xFFFFFFFFFFFF_FFFFFFFFFFFF_FFFFFFFFFFFF_000000000000_FFFFFFFFFFFFFFFF;
        if (i == 1)
            return self & 0xFFFFFFFFFFFF_FFFFFFFFFFFF_000000000000_FFFFFFFFFFFF_FFFFFFFFFFFFFFFF;
        if (i == 2)
            return self & 0xFFFFFFFFFFFF_000000000000_FFFFFFFFFFFF_FFFFFFFFFFFF_FFFFFFFFFFFFFFFF;
        if (i == 3)
            return self & 0x000000000000_FFFFFFFFFFFF_FFFFFFFFFFFF_FFFFFFFFFFFF_FFFFFFFFFFFFFFFF;


        return self;
```	
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/TokenId.sol#L442C5-L452C8
```
file : contracts/libraries
/Math.sol

41 - 87 :             if (absTick > uint256(int256(Constants.MAX_V3POOL_TICK))) revert Errors.InvalidTick();


            uint256 sqrtR = absTick & 0x1 != 0
                ? 0xfffcb933bd6fad37aa2d162d1a594001
                : 0x100000000000000000000000000000000;
            // RealV: 0xfffcb933bd6fad37aa2d162d1a594001
            if (absTick & 0x2 != 0) sqrtR = (sqrtR * 0xfff97272373d413259a46990580e213a) >> 128;
            // RealV: 0xfff97272373d413259a46990580e2139
            if (absTick & 0x4 != 0) sqrtR = (sqrtR * 0xfff2e50f5f656932ef12357cf3c7fdcc) >> 128;
            // RealV: 0xfff2e50f5f656932ef12357cf3c7fdca
            if (absTick & 0x8 != 0) sqrtR = (sqrtR * 0xffe5caca7e10e4e61c3624eaa0941cd0) >> 128;
            // RealV: 0xffe5caca7e10e4e61c3624eaa0941ccd
            if (absTick & 0x10 != 0) sqrtR = (sqrtR * 0xffcb9843d60f6159c9db58835c926644) >> 128;
            // RealV: 0xffcb9843d60f6159c9db58835c92663e
            if (absTick & 0x20 != 0) sqrtR = (sqrtR * 0xff973b41fa98c081472e6896dfb254c0) >> 128;
            // RealV: 0xff973b41fa98c081472e6896dfb254b6
            if (absTick & 0x40 != 0) sqrtR = (sqrtR * 0xff2ea16466c96a3843ec78b326b52861) >> 128;
            // RealV: 0xff2ea16466c96a3843ec78b326b5284f
            if (absTick & 0x80 != 0) sqrtR = (sqrtR * 0xfe5dee046a99a2a811c461f1969c3053) >> 128;
            // RealV: 0xfe5dee046a99a2a811c461f1969c3032
            if (absTick & 0x100 != 0) sqrtR = (sqrtR * 0xfcbe86c7900a88aedcffc83b479aa3a4) >> 128;
            // RealV: 0xfcbe86c7900a88aedcffc83b479aa363
            if (absTick & 0x200 != 0) sqrtR = (sqrtR * 0xf987a7253ac413176f2b074cf7815e54) >> 128;
            // RealV: 0xf987a7253ac413176f2b074cf7815dd0
            if (absTick & 0x400 != 0) sqrtR = (sqrtR * 0xf3392b0822b70005940c7a398e4b70f3) >> 128;
            // RealV: 0xf3392b0822b70005940c7a398e4b6ff1
            if (absTick & 0x800 != 0) sqrtR = (sqrtR * 0xe7159475a2c29b7443b29c7fa6e889d9) >> 128;
            // RealV: 0xe7159475a2c29b7443b29c7fa6e887f2
            if (absTick & 0x1000 != 0) sqrtR = (sqrtR * 0xd097f3bdfd2022b8845ad8f792aa5825) >> 128;
            // RealV: 0xd097f3bdfd2022b8845ad8f792aa548c
            if (absTick & 0x2000 != 0) sqrtR = (sqrtR * 0xa9f746462d870fdf8a65dc1f90e061e5) >> 128;
            // RealV: 0xa9f746462d870fdf8a65dc1f90e05b52
            if (absTick & 0x4000 != 0) sqrtR = (sqrtR * 0x70d869a156d2a1b890bb3df62baf32f7) >> 128;
            // RealV: 0x70d869a156d2a1b890bb3df62baf27ff
            if (absTick & 0x8000 != 0) sqrtR = (sqrtR * 0x31be135f97d08fd981231505542fcfa6) >> 128;
            // RealV: 0x31be135f97d08fd981231505542fbfe8
            if (absTick & 0x10000 != 0) sqrtR = (sqrtR * 0x9aa508b5b7a84e1c677de54f3e99bc9) >> 128;
            // RealV: 0x9aa508b5b7a84e1c677de54f3e988fe
            if (absTick & 0x20000 != 0) sqrtR = (sqrtR * 0x5d6af8dedb81196699c329225ee604) >> 128;
            // RealV: 0x5d6af8dedb81196699c329225ed28d
            if (absTick & 0x40000 != 0) sqrtR = (sqrtR * 0x2216e584f5fa1ea926041bedfe98) >> 128;
            // RealV: 0x2216e584f5fa1ea926041bedeaf4
            if (absTick & 0x80000 != 0) sqrtR = (sqrtR * 0x48a170391f7dc42444e8fa2) >> 128;
            // RealV: 0x48a170391f7dc42444e7be7


            if (tick > 0) sqrtR = type(uint256).max / sqrtR;
```
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/Math.sol#L41C12-L87C1

# G-10. Use Shift Right/Left instead of Division/Multiplication # 

A division/multiplication by any number `x` being a power of 2 can be calculated by shifting `log2(x)` to the right/left.

While the `DIV` opcode uses 5 gas, the `SHR` opcode only uses 3 gas. Furthermore, Solidity's division operation also includes a division-by-0 prevention which is bypassed using shifting.

Reference : https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md/#g008---use-shift-rightleft-instead-of-divisionmultiplication-if-possible

Instances : 
```
file : contracts
/SemiFungiblePositionManager.sol

1293 :                     uint256 numerator = netLiquidity + (removedLiquidity / 2 ** VEGOID);

1281 :                     .mulDiv(collected0, totalLiquidity * 2 ** 64, netLiquidity ** 2)

1284 :                     .mulDiv(collected1, totalLiquidity * 2 ** 64, netLiquidity ** 2)
```
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L1293

https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L1281

https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L1284C2-L1284C2
```
file : contracts/types
/TokenId.sol

389 :             int24 oneSidedRange = (selfWidth * tickSpacing) / 2;
```
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/TokenId.sol#L389
```
file : contracts/libraries
/Math.sol

400 :             prod0 |= prod1 * 2 ** 160;

462 :             prod0 |= prod1 * 2 ** 128;

524 :             prod0 |= prod1 * 2 ** 64;
```
https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/Math.sol#L400

https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/Math.sol#L462

https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/Math.sol#L524
