### Summary

* \[L-01\] Users should be able to choose the recipient for long positions in case of being blocked
    
* \[L-02\] Actual collected amounts and the requested amounts should be checked in `SemiFungiblePositionManager::_collectAndWritePositionData`
    
* \[L-03\] Input array lengths in `ERC1155Minimal::balanceOfBatch` is not checked
    
* \[L-04\] Possible missing function in `LeftRight.sol` library
    
* \[N-01\] Misleading comments related to `positionKey`
    
* \[N-02\] Slippage tick limits for swaps can be inclusive
    
* \[N-03\] NatSpec `@return` explanation is incorrect in `_createPositionInAMM()` function
    
* \[N-04\] Misleading comment in `_mintLiquidity()` function
    
* \[N-05\] No need to wrap `uniV3pool` parameter in `getAccountPremium` function
    
* \[N-06\] NatSpec `@return` comment is incorrect in `_getPremiaDeltas()` function
    

### \[L-01\] Users should be able to choose the recipient for long positions

[https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1223](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1223)

Users deposit tokens by opening short positions and withdraw tokens by opening long positions or closing short positions (*closing a short position is also considered as long in the codebase*).

If a position is long, the liquidity in the Uniswap is [burned first](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1033) and then [collected](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1051).

In the current implementation, the collected tokens are only transferred to the `msg.sender` and the user has no way to provide a different recipient.  
[https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1222C2-L1228C15](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1222C2-L1228C15)

```solidity
File: SemiFungiblePositionManager.sol
// function _collectAndWritePositionData
       ...
            (uint128 receivedAmount0, uint128 receivedAmount1) = univ3pool.collect(
 -->            msg.sender, //@audit the recipient is always msg.sender
                liquidityChunk.tickLower(),
                liquidityChunk.tickUpper(),
                uint128(amountToCollect.rightSlot()),
                uint128(amountToCollect.leftSlot())
            );
```

This `SemiFungiblePositionManager` contract is the ERC1155 version of Uniswap's `NonfungiblePositionManager`. Users can provide different recipient addresses in both `NonfungiblePositionManager` and `UniswapV3Pool` contracts in the Uniswap protocol.

Users should be able to provide recipients in case of being blocked by some token contracts (e.g. USDC or USDT).

* Alice opens a short position and deposits USDC to Uniswap through Panoptic
    
* Alice's account is blocked by the USDC contract.
    
* Alice tries to close her position.
    
* `_collectAndWritePositionData` reverts while transferring USDC back to Alice due to Alice being blocked.
    
* She can not close her position.
    

If Alice minted a position directly in Uniswap instead of doing this through Panoptic, she could've provided a different address, closed the position and gotten her tokens back.

**Recommendation**: Allow users to provide recipient addresses to collect fees and burnt liquidity instead of transferring only to the `msg.sender`

---

### \[L-02\] Actual collected amounts and the requested amounts should be checked in `SemiFungiblePositionManager::_collectAndWritePositionData`

[https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1222C2-L1228C15](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1222C2-L1228C15)

The earned fees and the burnt liquidities are collected by calling the [`UniswapV3Pool.collect()`](https://github.com/Uniswap/v3-core/blob/d8b1c635c275d2a9450bd6a78f3fa2484fef73eb/contracts/UniswapV3Pool.sol#L490C5-L513C6) function in the `SemiFungiblePositionManager::_collectAndWritePositionData()` function.

The SFPM contract provides the requested amounts while calling the UniswapV3 pool. These requested amounts include fees and burnt liquidity, and they are provided with `amountToCollect.rightSlot()` and `amountToCollect.leftSlot()`.

```solidity
File: SemiFungiblePositionManager.sol
// function _collectAndWritePositionData
       ...
            (uint128 receivedAmount0, uint128 receivedAmount1) = univ3pool.collect( // @audit what if the received amounts are different than the requested amounts?
                msg.sender,
                liquidityChunk.tickLower(),
                liquidityChunk.tickUpper(),
                uint128(amountToCollect.rightSlot()), // amount0requested
                uint128(amountToCollect.leftSlot()) // amount1requested
            );
```

The UniswapV3Pool [checks the requested amounts compared to the owed amounts](https://github.com/Uniswap/v3-core/blob/d8b1c635c275d2a9450bd6a78f3fa2484fef73eb/contracts/UniswapV3Pool.sol#L498C1-L501C101) to the position and transfers tokens.

```solidity
// UniswapV3Pool.sol

->      Position.Info storage position = positions.get(msg.sender, tickLower, tickUpper); //@audit Position owner is the SFPM contract

        amount0 = amount0Requested > position.tokensOwed0 ? position.tokensOwed0 : amount0Requested;
        amount1 = amount1Requested > position.tokensOwed1 ? position.tokensOwed1 : amount1Requeste
```

The owner of the position is the SFPM contract itself. So, it is normal to assume that the owed amounts to SFPM will be greater than the user's requested amounts and these requested amounts will be transferred correctly.

However, it would be better to check if the actual transferred amounts are equal to the requested amounts as an invariant check. It might be an extremely rare case but there is no way to collect the remaining owed tokens in that case since the SFPM doesn't have a specific function to call the UniswapV3 collect function.

**Recommendation:**

```solidity
if (receivedAmount0 != amountToCollect.rightSlot() || receivedAmount1 != amountToCollect.leftSlot()) revert
```

---

### \[L-03\] Input array lengths in `ERC1155Minimal::balanceOfBatch` is not checked

[https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/tokens/ERC1155Minimal.sol#L174](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/tokens/ERC1155Minimal.sol#L174)

```solidity
File: ERC1155Minimal.sol
    /// @notice Query balances for multiple users and tokens at once
--> /// @dev owners and ids must be of equal length //@audit it is not checked in the code below.
    /// @param owners the users to query balances for
    /// @param ids the ERC1155 token ids to query
    /// @return balances the balances for each user-token pair in the same order as the input
    function balanceOfBatch(
        address[] calldata owners,
        uint256[] calldata ids
    ) public view returns (uint256[] memory balances) {
        balances = new uint256[](owners.length);

        // Unchecked because the only math done is incrementing
        // the array index counter which cannot possibly overflow.
        unchecked {
            for (uint256 i = 0; i < owners.length; ++i) {
                balances[i] = balanceOf[owners[i]][ids[i]];
            }
        }
    }
```

As we can see in the developer's comment above, **owners and ids must be of equal length.**

However, the equality of these parameters is not checked at all in the code.

According to the [EIP-1155 standard](https://eips.ethereum.org/EIPS/eip-1155), these parameters are not strictly required to be equal for the `balanceOfBatch` function (*Different than the* `safeBatchTransferFrom` *where all inputs MUST be equal length*).

As you can see [here](https://github.com/transmissions11/solmate/blob/e8f96f25d48fe702117ce76c79228ca4f20206cb/src/tokens/ERC1155.sol#L124), it is checked in Solady implementation.

**Recommendation:** Consider adding a check similar to Solady's implementation.

```diff
+        if(owners.length == ids.length) revert
```

---

### \[L-04\] Possible missing function in `LeftRight.sol` library

[https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol)

LeftRight library is used to pack two separate data (each 128-bit) into a single 256-bit slot. There are multiple functions to add values to the right slot and left slot.

The functions to add value to the **right slot** are:

* `toRightSlot(uint256 self, uint128 right)`
    
* `toRightSlot(uint256 self, int128 right)`
    
* `toRightSlot(int256 self, uint128 right)`
    
* `toRightSlot(int256 self, int128 right)`
    

The functions to add value to the **left slot** are:

* `toLeftSlot(uint256 self, uint128 left)`
    
* `toLeftSlot(int256 self, uint128 left)`
    
* `toLeftSlot(int256 self, int128 left)`
    

When we compare it to right slot functions, it looks like the function with "**uint256** self" and "**int128** left" parameters is missing.

**Recommendation:** Consider adding a function called `toLeftSlot(uint256 self, int128 left)`, if necessary.

---

### \[N-01\] Misleading comments related to `positionKey`

[https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L175](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L175)

[https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L292](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L292)

```solidity
SemiFungiblePositionManager.sol
175.    /// @dev mapping that stores the liquidity data of keccak256(abi.encodePacked(address poolAddress, address owner, int24 tickLower, int24 tickUpper))
292.    /// @dev mapping that stores a LeftRight packing of feesBase of  keccak256(abi.encodePacked(address poolAddress, address owner, int24 tickLower, int24 tickUpper))
```

Developer comments regarding position key encoding in lines 175 and 292 are misleading. These comments include only 4 elements (`poolAddress`, `owner`, `tickLower` and `tickUpper`) for encoding.

However, there is one more element in the [actual implementation](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L594C13-L601C18), which is **tokenType.**

---

### \[N-02\] Slippage tick limits for swaps can be inclusive

[https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L713](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L713)

Users provide upper and lower tick limits for swaps when minting in-the-money positions.

In the current implementation, the transaction reverts if the new tick after the swap touches the slippage limits. However, these limits should be inclusive and the transaction should revert if these limits are passed.

**Recommendation:**

```diff
713. -         if ((newTick >= tickLimitHigh) || (newTick <= tickLimitLow)) revert Errors.PriceBoundFail();
713. +         if ((newTick > tickLimitHigh) || (newTick < tickLimitLow)) revert Errors.PriceBoundFail();
```

---

### \[N-03\] NatSpec `@return` explanation is incorrect in `_createPositionInAMM()` function

[https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L846](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L846)

```solidity
    /// @return totalMoved the total amount of liquidity moved from the msg.sender to Uniswap
--> /// @return totalCollected the total amount of liquidity collected from Uniswap to msg.sender //@audit It is not total liquidity collected, it is total fees collected.
    /// @return itmAmounts the amount of tokens swapped due to legs being in-the-money
    function _createPositionInAMM(
```

The comment regarding `totalCollected` parameter is incorrect. The returned value is not the total amount of liquidity collected from Uniswap. It is only the collected fee amount.

Received amounts from Uniswap include "fees + burnt liquidity". Burnt liquidity amounts are subtracted from the received amounts and the collected amounts are calculated in `_collectAndWritePositionData()` function.

[https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1231C1-L1244C85](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1231C1-L1244C85)

```solidity
file: SemiFungiblePositionManager.sol
// function _collectAndWritePositionData
            uint128 collected0;
            uint128 collected1;
            unchecked {
                collected0 = movedInLeg.rightSlot() < 0
-->                 ? receivedAmount0 - uint128(-movedInLeg.rightSlot())
                    : receivedAmount0;
                collected1 = movedInLeg.leftSlot() < 0
-->                 ? receivedAmount1 - uint128(-movedInLeg.leftSlot())
                    : receivedAmount1;
            }


            // CollectedOut is the amount of fees accumulated+collected (received - burnt)
-->         // That's because receivedAmount contains the burnt tokens and whatever amount of fees collected
            collectedOut = int256(0).toRightSlot(collected0).toLeftSlot(collected1);
```

---

### \[N-04\] Misleading comment in `_mintLiquidity()` function

[https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1158C1-L1158C101](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1158C1-L1158C101)

```solidity
file: SemiFungiblePositionManager.sol
// _mintLiquidity function

-->     // from msg.sender to the uniswap pool, stored as negative value to represent amount debited //@audit It is not negative 
        movedAmounts = int256(0).toRightSlot(int128(int256(amount0))).toLeftSlot(
            int128(int256(amount1))
        );
```

The comment above `movedAmounts` mentioned that the amounts are stored as negative values. However, both `amount0` and `amount1` are returned values of the Uniswap `mint` function, and [they are positive uint256 values](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1147).

---

### \[N-05\] No need to wrap `uniV3pool` parameter in `getAccountPremium` function

[https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1381C30-L1381C48](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1381C30-L1381C48)

```solidity
    function getAccountPremium(
-->     address univ3pool,
        address owner,
        uint256 tokenType,
        int24 tickLower,
        int24 tickUpper,
        int24 atTick,
        uint256 isLong
    ) external view returns (uint128 premiumToken0, uint128 premiumToken1) {
        bytes32 positionKey = keccak256(
-->         abi.encodePacked(address(univ3pool), owner, tokenType, tickLower, tickUpper) //@audit QA - univ3pool parameter is already address not IUniV3Pool. No need to do "address(univ3pool)"
        );
```

`univ3pool` parameter is already an `address`, not `IUniswapV3Pool`. Therefore, there is no need to do "*address(univ3pool)*"

---

### \[N-06\] NatSpec `@return` comment is incorrect in `_getPremiaDeltas()` function

[https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1253C1-L1254C147](https://github.com/code-423n4/2023-11-panoptic/blob/aa86461c9d6e60ef75ed5a1fe36a748b952c8666/contracts/SemiFungiblePositionManager.sol#L1253C1-L1254C147)

```solidity
    /// @return deltaPremiumOwed The extra premium (per liquidity X64) to be added to the owed accumulator for token0 (right) and token1 (right)
    /// @return deltaPremiumGross The extra premium (per liquidity X64) to be added to the gross accumulator for token0 (right) and token1 (right)
```

The explanation is: "*... for token0 (right) and token1 (right)."*

Both `token0` and `token1` is explained as the right slot in the comments. However, `token1` is on the left slot.