**[Multicall.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/multicall/Multicall.sol)**

- Using [`delegatecall`](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/multicall/Multicall.sol#L14C8-L15C87) inside a loop may cause issues with payable functions
    ```
     for (uint256 i = 0; i < data.length; ) {
              (bool success, bytes memory result) = address(this).delegatecall(data[i]); //@note
    ```
  If one of the `delegatecall` consumes part of the sent msg.value, other calls might fail, if they expect the full `msg.value`. Consider using a different design (possibly removing from loop), or fully document this to inform potential users.

- The `SemiFungiblePositionManager` is meant to be [multicalled](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L72C1-L72C61), but implements no function to recieve or withdraw `ETH`.
    ```
    contract SemiFungiblePositionManager is ERC1155, Multicall {
    ```
    The `Multicall` contract, through which the multicalls are made, is however marked [payable](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/multicall/Multicall.sol#L12C1-L12C96).
    ```
    function multicall(bytes[] calldata data) public payable returns (bytes[] memory results) {
    ```
    And the function is unprotected from any calls. Consequently, any `ETH` sent during a multicall, will be locked in the contract with no way to retrieve it. Consider removing the payable parameter, or introducing a sweep function to collect any unused `ETH`.
    

**[Math.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol)**

- [Incorrect comment](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/Math.sol#L461C1-L463C1) - Note that this is just 2^128 not ~2^160~.

    ```
    // Note that this is just 2**160 since 2**256 over the fixed denominator (2**128) equals 2**128 //@note 
    prod0 |= prod1 * 2 ** 128;
    ```
  
**[ERC1155Minimal.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol)**

-  `safeBatchTransferFrom` should employ checks, to ensure that `ids` and `amounts` arrays have the same length, making it more `EIP1155` compliant.
   According to [ERC1155 Multi token standard](https://eips.ethereum.org/EIPS/eip-1155), 
   > MUST revert if length of `_ids` is not the same as length of `_values`.
    This can help prevent any unexpected behaviour.
   ```
             if (ids.length != amounts.length) revert NotEnoughTokenBalance();
   ```
   This check can also be included in the `balanceOfBatch` function too, to ensure the `owners` and the `ids` are of the same length.
   ```
                     address[] calldata owners,
                     uint256[] calldata ids
   ```
   Also, the amount being transferred should also be checked to make sure it's <= balanceOf, e.g, also from `EIP1155` standard,
   > MUST revert if any of the balance(s) of the holder(s) for token(s) in `_ids` is lower than the respective amount(s) in `_values` sent to the recipient.
  
   ```
             if (amount > balanceOf) revert NotEnoughTokenBalance();
   ```

- **[SemiFungiblePositionManager](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol)**

  - From the provided readme, any erc20 token (except fee-on-transfer) is supported
    > Any compliant ERC20 token that is part of a Uniswap V3 pool and is not a fee-on-transfer token is supported, including ERC-777 tokens.

    The issue is that Uniswap v3 does not properly support [rebasing tokens](https://docs.uniswap.org/concepts/protocol/integration-issues) so using these tokens with it might result funds getting stuck. From the [docs](https://docs.uniswap.org/concepts/protocol/integration-issues)
    > A negative rebasing token, the more common variant, deflates the balances of token owners. Because the rebasing is not triggered by transfers, the router cannot expect when or how a rebasing will happen. Once it does, the pair reserves will be unbalanced, and the next person to transact with the pair will bear the cost of the delta as a result of the rebasing.
    > Positive rebasing tokens arbitrarily increase the balances of token holders. When a positive rebase happens, it creates a surplus that is unaccounted for in the trading pair. Because the extra tokens are unaccounted for in the trading pair, anyone can call skim() on the trading pair and effectively steal the positive difference from the rebalance.

    These tokens types should be made allowances for, or their effects should be properly documented for potential users.
