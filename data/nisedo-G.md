## Note to the Judge:

All gas optimization estimations presented in this report have been determined using the `forge test --gas-report` feature and compared against the `Deployment Cost` and `Deployment Size` of the audit repo commit [f75d07c](https://github.com/code-423n4/2023-11-panoptic).
This tool allows for a precise analysis of gas consumption before and after the proposed changes, ensuring the accuracy of the savings mentioned.

## G-1: Remove the  `return`  statement when the function defines named return variables to save gas

Once the return variable has been assigned (or has its default value), there is no need to explicitly return it at the end of the function, since it's returned automatically.

```solidity
File: contracts/SemiFungiblePositionManager.sol
715		return (totalCollectedFromAMM, totalMoved, newTick);
```

```diff
-		return (totalCollectedFromAMM, totalMoved, newTick);
```

- Deployment Cost: 4323042 (-7209 gas, -0.16648%)
- Deployment Size: 21783 (-36 gas, -0.01649938%)

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

- Deployment Cost: 4323042 (-7209 gas, -0.16648%)
- Deployment Size: 21783 (-36 gas, -0.01649938%)

## G-3: Remove unused named return variables to save gas
Consider changing the variable to be an unnamed one, since the variable is never assigned, nor is it returned by name.

```solidity
File: contracts/SemiFungiblePositionManager.sol
1459		) external view returns (IUniswapV3Pool UniswapV3Pool) {
```

```diff
-		) external view returns (IUniswapV3Pool UniswapV3Pool) {
+		) external view returns (IUniswapV3Pool) {
```

- Deployment Cost: 4323042 (-7209 gas, -0.16648%)
- Deployment Size: 21783 (-36 gas, -0.01649938%)

```solidity
File: contracts/libraries/Math.sol
103		) internal pure returns (uint256 amount0) {
```

```diff
-		) internal pure returns (uint256 amount0) {
+		) internal pure returns (uint256) {
```

- Deployment Cost: 4323042 (-7209 gas, -0.16648%)
- Deployment Size: 21783 (-36 gas, -0.01649938%)

```solidity
File: contracts/libraries/Math.sol
103		) internal pure returns (uint256 amount1) {
```

```diff
-		) internal pure returns (uint256 amount1) {
+		) internal pure returns (uint256) {
```

- Deployment Cost: 4323042 (-7209 gas, -0.16648%)
- Deployment Size: 21783 (-36 gas, -0.01649938%)

```solidity
File: contracts/libraries/Math.sol
138		) internal pure returns (uint128 liquidity) {
```

```diff
-		) internal pure returns (uint128 liquidity) {
+		) internal pure returns (uint128) {
```

- Deployment Cost: 4323042 (-7209 gas, -0.16648%)
- Deployment Size: 21783 (-36 gas, -0.01649938%)

```solidity
File: contracts/libraries/Math.sol
157		) internal pure returns (uint128 liquidity) {
```

```diff
-		) internal pure returns (uint128 liquidity) {
+		) internal pure returns (uint128) {
```

- Deployment Cost: 4323042 (-7209 gas, -0.16648%)
- Deployment Size: 21783 (-36 gas, -0.01649938%)

## G-4: Use assembly to check for `address(0)`

*Saves 6 gas per instance*

```solidity
File: contracts/SemiFungiblePositionManager.sol
356		if (address(univ3pool) == address(0)) revert Errors.UniswapPoolNotInitialized();

683		if (univ3pool == IUniswapV3Pool(address(0))) revert Errors.UniswapPoolNotInitialized();
```

## G-5: Use assembly to emit events

We can use assembly to emit events efficiently by utilizing `scratch space` and the `free memory pointer`. This will allow us to potentially avoid memory expansion costs. Note: In order to do this optimization safely, we will need to cache and restore the free memory pointer. For example, for a generic `emit` event for `eventSentAmountExample`:

```solidity
// uint256 id, uint256 value, uint256 amount
emit eventSentAmountExample(id, value, amount);
```

We can use the following assembly emit events:

```solidity
            assembly {
                let memptr := mload(0x40)
                mstore(0x00, calldataload(0x44))
                mstore(0x20, calldataload(0xa4))
                mstore(0x40, amount)
                log1(
                    0x00,
                    0x60,
                    // keccak256("eventSentAmountExample(uint256,uint256,uint256)")
                    0xa622cf392588fbf2cd020ff96b2f4ebd9c76d7a4bc7f3e6b2f18012312e76bc3
                )
                mstore(0x40, memptr)
            }
```

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