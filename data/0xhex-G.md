# Gas Optimization

# Summary

| Number | Gas Optimization                                                                                  | Context |
| :----: | :------------------------------------------------------------------------------------------------ | :-----: |
| [G-01] | Can Make The Variable Outside The Loop To Save Gas                                                |    8    |
| [G-02] | Use Assembly To Check For address(0)                                                              |    1    |
| [G-03] | Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas) for the extra SLOAD |    3    |
| [G-04] | State variables that are used multiple times in a function should be cached in stack variables    |    9    |
| [G-05] | The result of function calls should be cached rather than re-calling the function                 |    2    |
| [G-06] | Use assembly in place of abi.decode to extract calldata values more efficiently                   |    2    |
| [G-07] | Amounts should be checked for 0 before calling a transfer                                         |    1    |
| [G-08] | Use hardcode address instead address(this)                                                        |    2    |
| [G-09] | Multiple Address/id Mappings Can Be Combined Into A Single Mapping Of An Address/id To A Struct   |    2    |

## [G-01] Can Make The Variable Outside The Loop To Save Gas

When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract. Here's an example:

```solidity
File:  SemiFungiblePositionManager.sol
586  uint256 liquidityChunk = PanopticMath.getLiquidityChunk(

594  bytes32 positionKey_from = keccak256(

603  bytes32 positionKey_to = keccak256(

620  uint256 fromLiq = s_accountLiquidity[positionKey_from];

623  int256 fromBase = s_accountFeesBase[positionKey_from];
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L586

```solidity
File:  SemiFungiblePositionManager.sol
            int256 _moved;
            int256 _itmAmounts;
            int256 _totalCollected;
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L861-L863

```solidity
File:  multicall/Multicall.sol
15  (bool success, bytes memory result) = address(this).delegatecall(data[i]);
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/multicall/Multicall.sol#L15

## [G-02] Use Assembly To Check For address(0)

```solidity
File:  SemiFungiblePositionManager.sol

356  if (address(univ3pool) == address(0)) revert Errors.UniswapPoolNotInitialized();
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L356

## [G-03] Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas) for the extra SLOAD, and to avoid Gsset (20000 gas) when changing from 'false' to 'true', after having been 'true' in the past

```solidity
File:  SemiFungiblePositionManager.sol

226   s_poolContext[poolId].locked = true;

333   s_poolContext[poolId].locked = false;

688    swapAtMint = true;
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L226

## [G‑04] State variables that are used multiple times in a function should be cached in stack variables

```solidity
File:  SemiFungiblePositionManager.sol

615   (s_accountLiquidity[positionKey_to] != 0) ||
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L615

```solidity
File:  SemiFungiblePositionManager.sol

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
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L594-L611

```solidity
File:  SemiFungiblePositionManager.sol

        bytes32 positionKey = keccak256(
            abi.encodePacked(
                address(_univ3pool),
                msg.sender,
                _tokenType,
                _liquidityChunk.tickLower(),
                _liquidityChunk.tickUpper()
            )
        );
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L945-953

## [G‑05] The result of function calls should be cached rather than re-calling the function

```solidity
File: SemiFungiblePositionManager.sol

599           liquidityChunk.tickLower(),
600           liquidityChunk.tickUpper()
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L599-L600

## [G-06] Use assembly in place of abi.decode to extract calldata values more efficiently

Instead of using abi.decode, we can use assembly to decode our desired calldata values directly. This will allow us to avoid decoding calldata values that we will not use.

```solidity
File:  SemiFungiblePositionManager.sol

413   CallbackLib.CallbackData memory decoded = abi.decode(data, (CallbackLib.CallbackData));

447   CallbackLib.CallbackData memory decoded = abi.decode(data, (CallbackLib.CallbackData));
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L413

## [G-07] Amounts should be checked for 0 before calling a transfer

Checking non-zero transfer values can avoid an expensive external call and save gas.

There are 1 instances of this issue:

```solidity
File:  SemiFungiblePositionManager.sol

460   SafeTransferLib.safeTransferFrom(token, decoded.payer, msg.sender, amountToPay);
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L460

becuase if uint256 amountToPay = amount0Delta > 0 ? uint256(amount0Delta) : uint256(amount1Delta); this condition is true work will but if condition is false there is need of > 0 check.

## [G-08] Use hardcode address instead address(this)

it can be more gas-efficient to use a hardcoded address instead of the address(this) expression, especially if you need to use the same address multiple times in your contract.

The reason for this is that using address(this) requires an additional EXTCODESIZE operation to retrieve the contract's address from its bytecode, which can increase the gas cost of your contract. By pre-calculating and using a hardcoded address, you can avoid this additional operation and reduce the overall gas cost of your contract.

Here's an example of how you can use a hardcoded address instead of address(this):

```
contract MyContract {
    address public immutable myAddress = 0x1234567890123456789012345678901234567890;
    #L
    function doSomething() public {
        // Use myAddress instead of address(this)
        require(msg.sender == myAddress, "Caller is not authorized");

        // Do something
    }
}
```

In the above example, we have a contract MyContract with a public address variable myAddress. Instead of using address(this) to retrieve the contract's address, we have pre-calculated and hardcoded the address in the variable. This can help to reduce the gas cost of our contract and make our code more efficient.

[References](https://book.getfoundry.sh/reference/forge-std/compute-create-address)

There are 2 instances of this issue:

```solidity
File:  SemiFungiblePositionManager.sol

1106            address(this),

1148            address(this),
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol#L1106

## [G-09] Multiple Address/id Mappings Can Be Combined Into A Single Mapping Of An Address/id To A Struct, Where Appropriate

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to [not having to recalculate the key’s keccak256 hash](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0) (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.

Note: this insteance is missed form bots

```solidity
File:  tokens/ERC1155Minimal.sol

    mapping(address account => mapping(uint256 tokenId => uint256 balance)) public balanceOf;

    /// @notice Approved addresses for each user
    /// @dev indexed by user, then by operator
    /// @dev operator is approved to transfer all tokens on behalf of user
    mapping(address owner => mapping(address operator => bool approvedForAll))
        public isApprovedForAll;
```

https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol#L62-L68