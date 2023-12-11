L-01: Consider removing `payable` in function `multicall`
The contract multicall is currently defined as follow:
```solidity
function multicall(bytes[] calldata data) public payable returns (bytes[] memory results) {
        results = new bytes[](data.length);
        for (uint256 i = 0; i < data.length; ) {
            (bool success, bytes memory result) = address(this).delegatecall(data[i]);

            if (!success) {
                // Bubble up the revert reason
                // The bytes type is ABI encoded as a length-prefixed byte array
                // So we simply need to add 32 to the pointer to get the start of the data
                // And then revert with the size loaded from the first 32 bytes
                // Other solutions will do work to differentiate the revert reasons and provide paranthetical information
                // However, we have chosen to simply replicate the the normal behavior of the call
                // NOTE: memory-safe because it reads from memory already allocated by solidity (the bytes memory result)
                assembly ("memory-safe") {
                    revert(add(result, 32), mload(result))
                }
            }

            results[i] = result;

            unchecked {
                ++i;
            }
        }
    }
```

The function is payable and contract SFPM inherit this. However, SFPM has no payable function, so if someone calls multicall with some values, it will revert with vague reason `EVMRevert`
Below is a POC, save this test to file `SemiFungiblePositionManager.t.sol` and run it using command:
`forge test --match-path test/foundry/core/SemiFungiblePositionManager.t.sol --match-test test_MultiCallETHPayable -vvvv`

```solidity
function test_MultiCallETHPayable() public {
        _initPool(0);
        tickLower = currentTick - 100;
        tickUpper = currentTick + 100;
        tickLower = tickLower - tickLower % tickSpacing;
        tickUpper = tickUpper - tickUpper % tickSpacing;
        int24 strike = (tickUpper + tickLower)/2;
        int24 width = (tickUpper - tickLower)/tickSpacing;


        // Add leg and add positionSize
        uint256 tokenId = uint256(0).addUniv3pool(poolId).addLeg(
            0,  // leg index
            1,  // option rAtio 
            0, //isWETH,
            0, // islong
            0, // tokenType
            0, // riskParter
            strike,
            width
        );

        uint128 positionSize = 1e14;

        // craft calldata

        vm.startPrank(Alice);

        bytes[] memory data = new bytes[](1);
        data[0] = abi.encodeWithSignature(
            "mintTokenizedPosition(uint256,uint128,int24,int24)",
            tokenId,
            uint128(positionSize),
            TickMath.MIN_TICK,
            TickMath.MAX_TICK
        );
        vm.deal(Alice, 1 ether);
        // Alice mint an ITM position using multicall

        vm.expectRevert();
        sfpm.multicall{value: 1}(data);

        
        // Successful
        sfpm.multicall(data);
        vm.stopPrank();
        
    }
```