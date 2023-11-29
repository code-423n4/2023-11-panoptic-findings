## Note to the Judge:

All gas optimization estimations presented in this report have been determined using the `forge test --gas-report` feature and compared against the `Deployment Cost` and `Deployment Size` of the audit repo commit [f75d07c](https://github.com/code-423n4/2023-11-panoptic).
This tool allows for a precise analysis of gas consumption before and after the proposed changes, ensuring the accuracy of the savings mentioned.

| Contract Name                             | Deployment Cost (Before) | Deployment Cost (After) | Change in Deployment Cost (%)       | Deployment Size (Before) | Deployment Size (After) | Change in Deployment Size (%)     |
|-------------------------------------------|--------------------------|-------------------------|-------------------------------------|--------------------------|-------------------------|-----------------------------------|
| FeesCalc                                  | 294759                   | 294759                  | 0 (0%)                              | 1530                     | 1530                    | 0 (0%)                            |
| MockERC20                                 | 759913                   | 759913                  | 0 (0%)                              | 4914                     | 4914                    | 0 (0%)                            |
| MissingReturnToken                        | 345823                   | 345823                  | 0 (0%)                              | 1559                     | 1559                    | 0 (0%)                            |
| ReturnsFalseToken                         | 221500                   | 221500                  | 0 (0%)                              | 938                      | 938                     | 0 (0%)                            |
| ReturnsGarbageToken                       | 524591                   | 524591                  | 0 (0%)                              | 2452                     | 2452                    | 0 (0%)                            |
| ReturnsTooLittleToken                     | 224706                   | 224706                  | 0 (0%)                              | 954                      | 954                     | 0 (0%)                            |
| ReturnsTooMuchToken                       | 344617                   | 344617                  | 0 (0%)                              | 1553                     | 1553                    | 0 (0%)                            |
| ReturnsTwoToken                           | 218300                   | 218300                  | 0 (0%)                              | 922                      | 922                     | 0 (0%)                            |
| RevertingToken                            | 215294                   | 215294                  | 0 (0%)                              | 907                      | 907                     | 0 (0%)                            |
| SemiFungiblePositionManagerHarness        | 4330251                  | 4312225                 | -18026 (-0.42%)                     | 21819                    | 21729                   | -90 (-0.41%)                      |
| UniswapV3FactoryMock                      | 104553                   | 104553                  | 0 (0%)                              | 554                      | 554                     | 0 (0%)                            |
| FeesCalcHarness                           | 346186                   | 346186                  | 0 (0%)                              | 1761                     | 1761                    | 0 (0%)                            |
| MathHarness                               | 619051                   | 611445                  | -7606 (-1.23%)                      | 3124                     | 3086                    | -38 (-1.22%)                      |
| PanopticMathHarness                       | 1448328                  | 1442721                 | -5607 (-0.39%)                      | 7074                     | 7046                    | -28 (-0.40%)                      |
| MiniPositionManager                       | 715947                   | 715947                  | 0 (0%)                              | 3608                     | 3608                    | 0 (0%)                            |
| ERC1155MinimalHarness                     | 916555                   | 916555                  | 0 (0%)                              | 4610                     | 4610                    | 0 (0%)                            |
| LeftRightHarness                          | 366005                   | 366005                  | 0 (0%)                              | 1860                     | 1860                    | 0 (0%)                            |
| LiquidityChunkHarness                     | 184426                   | 184426                  | 0 (0%)                              | 953                      | 953                     | 0 (0%)                            |
| TokenIdHarness                            | 834470                   | 834470                  | 0 (0%)                              | 4200                     | 4200                    | 0 (0%)                            |
| **Total**                                 |                          |                         | **-31239**                          |                          |                         | **-156**                          |



## G-1: Remove the  `return`  statement when the function defines named return variables to save gas

Once the return variable has been assigned (or has its default value), there is no need to explicitly return it at the end of the function, since it's returned automatically.

```solidity
File: contracts/SemiFungiblePositionManager.sol
715		return (totalCollectedFromAMM, totalMoved, newTick);
```

```diff
-		return (totalCollectedFromAMM, totalMoved, newTick);
```

- Deployment Cost: 4323042 (-7209 gas, -0.1665%)
- Deployment Size: 21783 (-36 gas, -0.0165%)

```solidity
File: contracts/libraries/Math.sol
211     return result;

278     return result;

307     return result;

369     return result;

431     return result;

493     return result;
```

```diff
File: contracts/libraries/Math.sol
-       return result;

-       return result;

-       return result;

-       return result;

-       return result;

-       return result;
```

- Deployment Cost: 611445 (-7606 gas, -1.23%)
- Deployment Size: 3086 (-38 gas, -1.22%)

## G-2: Usage of  `uints`/`ints`  smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

[Each](https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html) operation involving a `uint8` costs an extra [**22-28 gas**](https://gist.github.com/IllIllI000/9388d20c70f9a4632eb3ca7836f54977) (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving `uint256`, due to the compiler having to clear the higher bits of the memory word before operating on the `uint8`, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.

```solidity
File: contracts/SemiFungiblePositionManager.sol
1272		uint128 premium0X64_base;
1273		uint128 premium1X64_base;
```

```diff
-		uint128 premium0X64_base;
+		uint256 premium0X64_base;
-		uint128 premium1X64_base;
+		uint256 premium1X64_base;
```

- Deployment Cost: 4323042 (-7209 gas, -0.1665%)
- Deployment Size: 21783 (-36 gas, -0.0165%)

## G-3: Use assembly to check for `address(0)`

```solidity
File: contracts/SemiFungiblePositionManager.sol
356		if (address(univ3pool) == address(0)) revert Errors.UniswapPoolNotInitialized();

683		if (univ3pool == IUniswapV3Pool(address(0))) revert Errors.UniswapPoolNotInitialized();
```
    
```diff
-		if (address(univ3pool) == address(0)) revert Errors.UniswapPoolNotInitialized();

+       bytes4 errorSelector = Errors.UniswapPoolNotInitialized.selector;
+       assembly {
+           let poolAddress := univ3pool

+           if iszero(poolAddress) {
+               mstore(0, errorSelector)
+               revert(0, 4)
+           }
+       }
```

- Deployment Cost: 4325242 (-5009 gas, -0.1157%)
- Deployment Size: 21794 (-25 gas, -0.0115%)

```diff
-		if (univ3pool == IUniswapV3Pool(address(0))) revert Errors.UniswapPoolNotInitialized();

+       bytes4 errorSelector = Errors.UniswapPoolNotInitialized.selector;
+       assembly {
+           let poolAddress := univ3pool

+           if iszero(poolAddress) {
+               mstore(0, errorSelector)
+               revert(0, 4)
+           }
+       }
```

- Deployment Cost: 4329451 (-800 gas, -0.0185%)
- Deployment Size: 21815 (-4 gas, -0.0018%)

## G-4: Use assembly to emit events

We can use assembly to emit events efficiently by utilizing `scratch space` and the `free memory pointer`. This will allow us to potentially avoid memory expansion costs. Note: In order to do this optimization safely, we will need to cache and restore the free memory pointer.

```solidity
File: contracts/SemiFungiblePositionManager.sol
386		emit PoolInitialized(univ3pool);

490		emit TokenizedPositionBurnt(msg.sender, tokenId, positionSize);

523		emit TokenizedPositionMinted(msg.sender, tokenId, positionSize);
```

```solidity
File: contracts/tokens/ERC1155Minimal.sol
80              emit ApprovalForAll(msg.sender, operator, approved);

161             emit TransferBatch(msg.sender, from, to, ids, amounts);
```

## G-5: Nesting if-statements is cheaper than using `&&`

Nesting if-statements avoids the stack operations of setting up and using an extra `jumpdest`.

```solidity
File: contracts/SemiFungiblePositionManager.sol
774              if ((itm0 != 0) && (itm1 != 0)) {
775                  (uint160 sqrtPriceX96, , , , , , ) = _univ3pool.slot0();
776  
777                  // implement a single "netting" swap. Thank you @danrobinson for this puzzle/idea
778                  // note: negative ITM amounts denote a surplus of tokens (burning liquidity), while positive amounts denote a shortage of tokens (minting liquidity)
779                  // compute the approximate delta of token0 that should be resolved in the swap at the current tick
780                  // we do this by flipping the signs on the token1 ITM amount converting+deducting it against the token0 ITM amount
781                  // couple examples (price = 2 1/0):
782                  //  - 100 surplus 0, 100 surplus 1 (itm0 = -100, itm1 = -100)
783                  //    normal swap 0: 100 0 => 200 1
784                  //    normal swap 1: 100 1 => 50 0
785                  //    final swap amounts: 50 0 => 100 1
786                  //    netting swap: net0 = -100 - (-100/2) = -50, ZF1 = true, 50 0 => 100 1
787                  // - 100 surplus 0, 100 shortage 1 (itm0 = -100, itm1 = 100)
788                  //    normal swap 0: 100 0 => 200 1
789                  //    normal swap 1: 50 0 => 100 1
790                  //    final swap amounts: 150 0 => 300 1
791                  //    netting swap: net0 = -100 - (100/2) = -150, ZF1 = true, 150 0 => 300 1
792                  // - 100 shortage 0, 100 surplus 1 (itm0 = 100, itm1 = -100)
793                  //    normal swap 0: 200 1 => 100 0
794                  //    normal swap 1: 100 1 => 50 0
795                  //    final swap amounts: 300 1 => 150 0
796                  //    netting swap: net0 = 100 - (-100/2) = 150, ZF1 = false, 300 1 => 150 0
797                  // - 100 shortage 0, 100 shortage 1 (itm0 = 100, itm1 = 100)
798                  //    normal swap 0: 200 1 => 100 0
799                  //    normal swap 1: 50 0 => 100 1
800                  //    final swap amounts: 100 1 => 50 0
801                  //    netting swap: net0 = 100 - (100/2) = 50, ZF1 = false, 100 1 => 50 0
802                  // - = Net surplus of token0
803                  // + = Net shortage of token0
804                  int256 net0 = itm0 - PanopticMath.convert1to0(itm1, sqrtPriceX96);
805  
806                  zeroForOne = net0 < 0;
807  
808                  //compute the swap amount, set as positive (exact input)
809                  swapAmount = -net0;
810              } else if (itm0 != 0) {
811                  zeroForOne = itm0 < 0;
812                  swapAmount = -itm0;
813              } else {
814                  zeroForOne = itm1 > 0;
815                  swapAmount = -itm1;
816              }
```

```diff
+		if (itm0 != 0) {
+			if (itm1 != 0) {
+				(uint160 sqrtPriceX96, , , , , , ) = _univ3pool.slot0();
+				int256 net0 = itm0 - PanopticMath.convert1to0(itm1, sqrtPriceX96);
+				zeroForOne = net0 < 0;
+				swapAmount = -net0;
+			} else {
+				zeroForOne = itm0 < 0;
+				swapAmount = -itm0;
+			}
+		} else {
+			if (itm1 != 0) {
+			zeroForOne = itm1 > 0;
+			swapAmount = -itm1;
+     			}
+		}
```

- Deployment Cost: 4328851 (-1400 gas, -0.0323%)
- Deployment Size: 21812 (-7 gas, -0.0032%)