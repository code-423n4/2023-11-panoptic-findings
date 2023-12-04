## [G-01] abi.encode() is less efficient than abi.encodePacked()

Changing abi.encode function to abi.encodePacked can save gas since the abi.encode function pads extra null bytes at the end of the call data, which is unnecessary. Also, in general, abi.encodePacked is more gas-efficient (see [Solidity-Encode-Gas-Comparison](https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison)).

There are 3 instances of this issue in 2 files:

```
File: contracts/SemiFungiblePositionManager.sol	

760: data = abi.encode(

1134: bytes memory mintdata = abi.encode(
```

    diff --git a/contracts/SemiFungiblePositionManager.sol b/contracts/SemiFungiblePositionManager.sol
    index 0ea05b4..3160618 100644
    --- a/contracts/SemiFungiblePositionManager.sol
    +++ b/contracts/SemiFungiblePositionManager.sol
    @@ -757,7 +757,7 @@ contract SemiFungiblePositionManager is ERC1155, Multicall {
                 int128 itm1 = itmAmounts.leftSlot();

                 // construct the swap callback struct
    -            data = abi.encode(
    +            data = abi.encodePacked(
                     CallbackLib.CallbackData({
                         poolFeatures: CallbackLib.PoolFeatures({
                             token0: _univ3pool.token0(),
    @@ -1131,7 +1131,7 @@ contract SemiFungiblePositionManager is ERC1155, Multicall {
             IUniswapV3Pool univ3pool
         ) internal returns (int256 movedAmounts) {
             // build callback data
    -        bytes memory mintdata = abi.encode(
    +        bytes memory mintdata = abi.encodePacked(
                 CallbackLib.CallbackData({ // compute by reading values from univ3pool every time
                         poolFeatures: CallbackLib.PoolFeatures({
                             token0: univ3pool.token0(),

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol

```
File: contracts/libraries/CallbackLib.sol	

43: keccak256(abi.encode(features)),
```

    diff --git a/contracts/libraries/CallbackLib.sol b/contracts/libraries/CallbackLib.sol
    index 807513c..72c9784 100644
    --- a/contracts/libraries/CallbackLib.sol
    +++ b/contracts/libraries/CallbackLib.sol
    @@ -40,7 +40,7 @@ library CallbackLib {
                                 abi.encodePacked(
                                     bytes1(0xff),
                                     factory,
    -                                keccak256(abi.encode(features)),
    +                                keccak256(abi.encodePacked(features)),
                                     Constants.V3POOL_INIT_CODE_HASH
                                 )
                             )

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/CallbackLib.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }
    contract Contract0 {
        string a = "Code4rena";
        function not_optimized() public returns(bytes32){
            return keccak256(abi.encode(a));
        }
    }
    contract Contract1 {
        string a = "Code4rena";
        function optimized() public returns(bytes32){
            return keccak256(abi.encodePacked(a));
        }
    }

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 101871                                    | 683             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| not_optimized                             | 2661            | 2661 | 2661   | 2661 | 1       |

| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 99465                                     | 671             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| optimized                                 | 2608            | 2608 | 2608   | 2608 | 1       |

## [G-02] Using XOR (^) and AND (&) bitwise equivalents

Given 4 variables a, b, c and d represented as such:

    0 0 0 0 0 1 1 0 <- a
    0 1 1 0 0 1 1 0 <- b
    0 0 0 0 0 0 0 0 <- c
    1 1 1 1 1 1 1 1 <- d

To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities.Now, if a != b, this means that there’s at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas.However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values.These are logically equivalent too, as the OR bitwise operator (|) would result in a 1 somewhere if any value is not 0 between the XOR (^) statements, meaning if any XOR (^) statement verifies that its arguments are different.

There are 6 instances of this issue in 4 files:

```
File: contracts/SemiFungiblePositionManager.sol	

615: (s_accountLiquidity[positionKey_to] != 0) ||
616: (s_accountFeesBase[positionKey_to] != 0)
```

    diff --git a/contracts/SemiFungiblePositionManager.sol b/contracts/SemiFungiblePositionManager.sol
    index 0ea05b4..16d4db8 100644
    --- a/contracts/SemiFungiblePositionManager.sol
    +++ b/contracts/SemiFungiblePositionManager.sol
    @@ -612,8 +612,8 @@ contract SemiFungiblePositionManager is ERC1155, Multicall {

                 // Revert if recipient already has that position
                 if (
    -                (s_accountLiquidity[positionKey_to] != 0) ||
    -                (s_accountFeesBase[positionKey_to] != 0)
    +                ((s_accountLiquidity[positionKey_to] ^ 0) |
    +                (s_accountFeesBase[positionKey_to] ^ 0)) != 0
                 ) revert Errors.TransferFailed();

                 // Revert if not all balance is transferred

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol

```
File: contracts/tokens/ERC1155Minimal.sol	

202: interfaceId == 0x01ffc9a7 || // ERC165 Interface ID for ERC165
203: interfaceId == 0xd9b67a26; // ERC165 Interface ID for ERC1155
```

    diff --git a/contracts/tokens/ERC1155Minimal.sol b/contracts/tokens/ERC1155Minimal.sol
    index f6ece50..0c29f95 100644
    --- a/contracts/tokens/ERC1155Minimal.sol
    +++ b/contracts/tokens/ERC1155Minimal.sol
    @@ -199,8 +199,8 @@ abstract contract ERC1155 {
         /// @return supported true if the interface is supported
         function supportsInterface(bytes4 interfaceId) public pure returns (bool) {
             return
    -            interfaceId == 0x01ffc9a7 || // ERC165 Interface ID for ERC165
    -            interfaceId == 0xd9b67a26; // ERC165 Interface ID for ERC1155
    +            ((interfaceId ^ 0x01ffc9a7) & // ERC165 Interface ID for ERC165
    +            (interfaceId ^ 0xd9b67a26)) == 0; // ERC165 Interface ID for ERC1155
         }

         /*//////////////////////////////////////////////////////////////

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol

```
File: contracts/types/LeftRight.sol	

167: if (left128 != left256 || right128 != right256) revert Errors.UnderOverFlow();

185: if (left128 != left256 || right128 != right256) revert Errors.UnderOverFlow();
```

    diff --git a/contracts/types/LeftRight.sol b/contracts/types/LeftRight.sol
    index c8bd69c..b0ec80c 100644
    --- a/contracts/types/LeftRight.sol
    +++ b/contracts/types/LeftRight.sol
    @@ -164,7 +164,7 @@ library LeftRight {
                 int256 right256 = int256(x.rightSlot()) + y.rightSlot();
                 int128 right128 = int128(right256);

    -            if (left128 != left256 || right128 != right256) revert Errors.UnderOverFlow();
    +            if (((left128 ^ left256) | (right128 ^ right256)) != 0) revert Errors.UnderOverFlow();

                 return z.toRightSlot(right128).toLeftSlot(left128);
             }
    @@ -182,7 +182,7 @@ library LeftRight {
                 int256 right256 = int256(x.rightSlot()) - y.rightSlot();
                 int128 right128 = int128(right256);

    -            if (left128 != left256 || right128 != right256) revert Errors.UnderOverFlow();
    +            if (((left128 ^ left256) | (right128 ^ right256)) != 0) revert Errors.UnderOverFlow();

                 return z.toRightSlot(right128).toLeftSlot(left128);
             }

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol

```
File: contracts/types/TokenId.sol	

483: (self.strike(i) == Constants.MIN_V3POOL_TICK) ||
484: (self.strike(i) == Constants.MAX_V3POOL_TICK)

498: (self.asset(riskPartnerIndex) != self.asset(i)) ||
499: (self.optionRatio(riskPartnerIndex) != self.optionRatio(i))
```

    diff --git a/contracts/types/TokenId.sol b/contracts/types/TokenId.sol
    index 6f3517c..890471a 100644
    --- a/contracts/types/TokenId.sol
    +++ b/contracts/types/TokenId.sol
    @@ -480,8 +480,8 @@ library TokenId {
                     if ((self.width(i) == 0)) revert Errors.InvalidTokenIdParameter(5);
                     // Strike cannot be MIN_TICK or MAX_TICK
                     if (
    -                    (self.strike(i) == Constants.MIN_V3POOL_TICK) ||
    -                    (self.strike(i) == Constants.MAX_V3POOL_TICK)
    +                    ((self.strike(i) ^ Constants.MIN_V3POOL_TICK) &
    +                    (self.strike(i) ^ Constants.MAX_V3POOL_TICK)) == 0
                     ) revert Errors.InvalidTokenIdParameter(4);

                     // In the following, we check whether the risk partner of this leg is itself
    @@ -495,8 +495,8 @@ library TokenId {

                         // Ensures that risk partners have 1) the same asset, and 2) the same ratio
                         if (
    -                        (self.asset(riskPartnerIndex) != self.asset(i)) ||
    -                        (self.optionRatio(riskPartnerIndex) != self.optionRatio(i))
    +                        ((self.asset(riskPartnerIndex) ^ self.asset(i)) |
    +                        (self.optionRatio(riskPartnerIndex) ^ self.optionRatio(i))) != 0
                         ) revert Errors.InvalidTokenIdParameter(3);

                         // long/short status of associated legs

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized(1,2);
            c1.optimized(1,2);
        }
    }
    
    contract Contract0 {
        function not_optimized(uint8 a,uint8 b) public returns(bool){
            return ((a==1) || (b==1));
        }
    }
    
    contract Contract1 {
        function optimized(uint8 a,uint8 b) public returns(bool){
            return ((a ^ 1) & (b ^ 1)) == 0;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 46099                                     | 261             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 456             | 456 | 456    | 456 | 1       |

| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 42493                                     | 243             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 430             | 430 | 430    | 430 | 1       |

## [G-03] <x> += <y> costs more gas than <x> = <x> + <y> for state variables

Using the addition operator instead of plus-equals saves [113 gas](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8). Subtractions act the same way.

There are 10 instances of this issue in 2 files:

```
File: contracts/SemiFungiblePositionManager.sol	

899: amount0 += Math.getAmount0ForLiquidity(liquidityChunk);
 
901: amount1 += Math.getAmount1ForLiquidity(liquidityChunk);

979: removedLiquidity -= chunkLiquidity;

999: removedLiquidity += chunkLiquidity;
```

    diff --git a/contracts/SemiFungiblePositionManager.sol b/contracts/SemiFungiblePositionManager.sol
    index 0ea05b4..28c1c3b 100644
    --- a/contracts/SemiFungiblePositionManager.sol
    +++ b/contracts/SemiFungiblePositionManager.sol
    @@ -896,9 +896,9 @@ contract SemiFungiblePositionManager is ERC1155, Multicall {

                     unchecked {
                         // increment accumulators of the upper bound on tokens contained across all legs of the position at any given tick
    -                    amount0 += Math.getAmount0ForLiquidity(liquidityChunk);
    +                    amount0 = amount0 + Math.getAmount0ForLiquidity(liquidityChunk);

    -                    amount1 += Math.getAmount1ForLiquidity(liquidityChunk);
    +                    amount1 = amount1 + Math.getAmount1ForLiquidity(liquidityChunk);
                     }
                 }

    @@ -976,7 +976,7 @@ contract SemiFungiblePositionManager is ERC1155, Multicall {
                     /// @dev If the isLong flag is 0=short but the position was burnt, then this is closing a long position
                     /// @dev so the amount of short liquidity should decrease.
                     if (_isBurn) {
    -                    removedLiquidity -= chunkLiquidity;
    +                    removedLiquidity = removedLiquidity - chunkLiquidity;
                     }
                 } else {
                     // the _leg is long (buying: moving *from* uniswap to msg.sender)
    @@ -996,7 +996,7 @@ contract SemiFungiblePositionManager is ERC1155, Multicall {
                     /// @dev If the isLong flag is 1=long and the position is minted, then this is opening a long position
                     /// @dev so the amount of short liquidity should increase.
                     if (!_isBurn) {
    -                    removedLiquidity += chunkLiquidity;
    +                    removedLiquidity = removedLiquidity + chunkLiquidity;
                     }
                 }

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol

```
File: contracts/tokens/ERC1155Minimal.sol	

99: balanceOf[from][id] -= amount;

103: balanceOf[to][id] += amount;

145: balanceOf[from][id] -= amount;

149: balanceOf[to][id] += amount;

217: balanceOf[to][id] += amount;

237: balanceOf[from][id] -= amount;
```

    diff --git a/contracts/tokens/ERC1155Minimal.sol b/contracts/tokens/ERC1155Minimal.sol
    index f6ece50..6a766a4 100644
    --- a/contracts/tokens/ERC1155Minimal.sol
    +++ b/contracts/tokens/ERC1155Minimal.sol
    @@ -96,11 +96,11 @@ abstract contract ERC1155 {
         ) public {
             if (!(msg.sender == from || isApprovedForAll[from][msg.sender])) revert NotAuthorized();

    -        balanceOf[from][id] -= amount;
    +        balanceOf[from][id] = balanceOf[from][id] - amount;

             // balance will never overflow
             unchecked {
    -            balanceOf[to][id] += amount;
    +            balanceOf[to][id] = balanceOf[to][id] + amount;
             }

             afterTokenTransfer(from, to, id, amount);
    @@ -142,11 +142,11 @@ abstract contract ERC1155 {
                 id = ids[i];
                 amount = amounts[i];

    -            balanceOf[from][id] -= amount;
    +            balanceOf[from][id] = balanceOf[from][id] - amount;

                 // balance will never overflow
                 unchecked {
    -                balanceOf[to][id] += amount;
    +                balanceOf[to][id] = balanceOf[to][id] + amount;
                 }

                 // An array can't have a total length
    @@ -214,7 +214,7 @@ abstract contract ERC1155 {
         function _mint(address to, uint256 id, uint256 amount) internal {
             // balance will never overflow
             unchecked {
    -            balanceOf[to][id] += amount;
    +            balanceOf[to][id] = balanceOf[to][id] + amount;
             }

             emit TransferSingle(msg.sender, address(0), to, id, amount);
    @@ -234,7 +234,7 @@ abstract contract ERC1155 {
         /// @param id the ERC1155 token id to mint
         /// @param amount the amount of tokens to burn
         function _burn(address from, uint256 id, uint256 amount) internal {
    -        balanceOf[from][id] -= amount;
    +        balanceOf[from][id] = balanceOf[from][id] - amount;

             emit TransferSingle(msg.sender, from, address(0), id, amount);
         }

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;

        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }

        function testGas() public {
            c0.add();
            c1.add1();
        }
    }

    contract Contract0 {

        uint8 num1 = 1;

        function add() public{
            num1 += 1;
        }

    }

    contract Contract1 {

        uint8 num1 = 1;

        function add1() public{
            num1 = num1 + 1;
        }

    }

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 67017                                     | 268             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| add                                       | 5405            | 5405 | 5405   | 5405 | 1       |

| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 70623                                     | 286             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| add1                                      | 5363            | 5363 | 5363   | 5363 | 1       |

## [G-04] Instead of counting down in *for* statements, count up

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

Using the addition operator instead of plus-equals saves [113 gas](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8). Subtructions act the same way.

There are 7 instances of this issue in 4 files:

```
File: contracts/SemiFungiblePositionManager.sol	

550: for (uint256 i = 0; i < ids.length; ) {
552:     unchecked {
553:         ++i;
554:     }

583: for (uint256 leg = 0; leg < numLegs; ) {
631:     unchecked {
632:         ++leg;
633:     }

860: for (uint256 leg = 0; leg < numLegs; ) {
909:     unchecked {
910:         ++leg;
911:     }
```

    diff --git a/contracts/SemiFungiblePositionManager.sol b/contracts/SemiFungiblePositionManager.sol
    index 0ea05b4..ed5a0a5 100644
    --- a/contracts/SemiFungiblePositionManager.sol
    +++ b/contracts/SemiFungiblePositionManager.sol
    @@ -547,10 +547,10 @@ contract SemiFungiblePositionManager is ERC1155, Multicall {
             uint256[] memory ids,
             uint256[] memory amounts
         ) internal override {
    -        for (uint256 i = 0; i < ids.length; ) {
    +        for (uint256 i = ids.length - 1 ; i >= 0 ; ) {
                 registerTokenTransfer(from, to, ids[i], amounts[i]);
                 unchecked {
    -                ++i;
    +                --i;
                 }
             }
         }
    @@ -580,7 +580,7 @@ contract SemiFungiblePositionManager is ERC1155, Multicall {
             IUniswapV3Pool univ3pool = s_poolContext[id.validate()].pool;

             uint256 numLegs = id.countLegs();
    -        for (uint256 leg = 0; leg < numLegs; ) {
    +        for (uint256 leg = numLegs - 1; leg >= 0 ; ) {
                 // for this leg index: extract the liquidity chunk: a 256bit word containing the liquidity amount and upper/lower tick
                 // @dev see `contracts/types/LiquidityChunk.sol`
                 uint256 liquidityChunk = PanopticMath.getLiquidityChunk(
    @@ -629,7 +629,7 @@ contract SemiFungiblePositionManager is ERC1155, Multicall {
                 s_accountFeesBase[positionKey_to] = fromBase;
                 s_accountFeesBase[positionKey_from] = 0;
                 unchecked {
    -                ++leg;
    +                --leg;
                 }
             }
         }
    @@ -857,7 +857,7 @@ contract SemiFungiblePositionManager is ERC1155, Multicall {

             uint256 numLegs = tokenId.countLegs();
             // loop through up to the 4 potential legs in the tokenId
    -        for (uint256 leg = 0; leg < numLegs; ) {
    +        for (uint256 leg = numLegs - 1; leg >= 0 ; ) {
                 int256 _moved;
                 int256 _itmAmounts;
                 int256 _totalCollected;
    @@ -907,7 +907,7 @@ contract SemiFungiblePositionManager is ERC1155, Multicall {
                 totalCollected = totalCollected.add(_totalCollected);

                 unchecked {
    -                ++leg;
    +                --leg;
                 }
             }

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol

```
File: contracts/tokens/ERC1155Minimal.sol	

141: for (uint256 i = 0; i < ids.length; ) {
154:     unchecked {
155:         ++i;
156:     }

187: for (uint256 i = 0; i < owners.length; ++i) {
```

    diff --git a/contracts/tokens/ERC1155Minimal.sol b/contracts/tokens/ERC1155Minimal.sol
    index f6ece50..8a4d22d 100644
    --- a/contracts/tokens/ERC1155Minimal.sol
    +++ b/contracts/tokens/ERC1155Minimal.sol
    @@ -138,7 +138,7 @@ abstract contract ERC1155 {
             uint256 id;
             uint256 amount;

    -        for (uint256 i = 0; i < ids.length; ) {
    +        for (uint256 i = ids.length - 1; i >= 0 ; ) {
                 id = ids[i];
                 amount = amounts[i];

    @@ -152,7 +152,7 @@ abstract contract ERC1155 {
                 // An array can't have a total length
                 // larger than the max uint256 value.
                 unchecked {
    -                ++i;
    +                --i;
                 }
             }

    @@ -184,7 +184,7 @@ abstract contract ERC1155 {
             // Unchecked because the only math done is incrementing
             // the array index counter which cannot possibly overflow.
             unchecked {
    -            for (uint256 i = 0; i < owners.length; ++i) {
    +            for (uint256 i = owners.length - 1; i >= 0 ; --i) {
                     balances[i] = balanceOf[owners[i]][ids[i]];
                 }
             }

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol

```
File: contracts/types/TokenId.sol	

468: for (uint256 i = 0; i < 4; ++i) {
```

    diff --git a/contracts/types/TokenId.sol b/contracts/types/TokenId.sol
    index 6f3517c..f114f33 100644
    --- a/contracts/types/TokenId.sol
    +++ b/contracts/types/TokenId.sol
    @@ -465,7 +465,7 @@ library TokenId {

             // loop through the 4 (possible) legs in the tokenId `self`
             unchecked {
    -            for (uint256 i = 0; i < 4; ++i) {
    +            for (uint256 i = 4; i >= 0 ; --i) {
                     if (self.optionRatio(i) == 0) {
                         // final leg in this position identified;
                         // make sure any leg above this are zero as well

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol

```
File: contracts/multicall/Multicall.sol	

14: for (uint256 i = 0; i < data.length; ) {
```

    diff --git a/contracts/multicall/Multicall.sol b/contracts/multicall/Multicall.sol
    index e289a34..a734ff2 100644
    --- a/contracts/multicall/Multicall.sol
    +++ b/contracts/multicall/Multicall.sol
    @@ -11,7 +11,7 @@ abstract contract Multicall {
         /// @return results The data returned by each call
         function multicall(bytes[] calldata data) public payable returns (bytes[] memory results) {
             results = new bytes[](data.length);
    -        for (uint256 i = 0; i < data.length; ) {
    +        for (uint256 i = data.length - 1; i >= 0 ; ) {
                 (bool success, bytes memory result) = address(this).delegatecall(data[i]);

                 if (!success) {
    @@ -30,7 +30,7 @@ abstract contract Multicall {
                 results[i] = result;

                 unchecked {
    -                ++i;
    +                --i;
                 }
             }
         }

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/multicall/Multicall.sol

#### Test Code

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

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 77011                                     | 311             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 7040            | 7040 | 7040   | 7040 | 1       |

| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 73811                                     | 295             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 3819            | 3819 | 3819   | 3819 | 1       |