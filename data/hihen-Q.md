# QA Report

## Summary

### Low Issues

Total **3 instances** over **2 issues**:

|ID|Issue|Instances|
|:--:|:---|:--:|
| [[L-01]](#l-01-missing-zero-address-check-in-constructor) | Missing zero address check in constructor | 1 |
| [[L-02]](#l-02-vulnerable-versions-of-packages-are-being-used) | Vulnerable versions of packages are being used | 2 |

### Non Critical Issues

Total **62 instances** over **10 issues**:

|ID|Issue|Instances|
|:--:|:---|:--:|
| [[N-01]](#n-01-contract-name-does-not-match-its-filename) | Contract name does not match its filename | 1 |
| [[N-02]](#n-02-events-should-be-emitted-before-external-calls) | Events should be emitted before external calls | 1 |
| [[N-03]](#n-03-contract-implements-interface-without-extending-the-interface) | Contract implements interface without extending the interface | 1 |
| [[N-04]](#n-04-openzeppelincontracts-should-be-upgraded-to-a-newer-version) | @openzeppelin/contracts should be upgraded to a newer version | 2 |
| [[N-05]](#n-05-natspec-documentation-for-function-is-missing) | NatSpec documentation for function is missing | 1 |
| [[N-06]](#n-06-variables-should-be-named-in-mixedcase-style) | Variables should be named in mixedCase style | 19 |
| [[N-07]](#n-07-missing-zero-address-check-in-functions-with-address-parameters) | Missing zero address check in functions with address parameters | 3 |
| [[N-08]](#n-08-named-mappings-are-recommended) | Named mappings are recommended | 2 |
| [[N-09]](#n-09-whitespace-in-expressions) | Whitespace in Expressions | 28 |
| [[N-10]](#n-10-empty-bytes-check-is-missing) | Empty bytes check is missing | 4 |

## Low Issues

### [L-01] Missing zero address check in constructor

Constructors often take address parameters to initialize important components of a contract, such as owner or linked contracts. However, without a checking, there's a risk that an address parameter could be mistakenly set to the zero address (0x0). This could be due to an error or oversight during contract deployment. A zero address in a crucial role can cause serious issues, as it cannot perform actions like a normal address, and any funds sent to it will be irretrievable. It's therefore crucial to include a zero address check in constructors to prevent such potential problems. If a zero address is detected, the constructor should revert the transaction.

There is 1 instance:

- *SemiFungiblePositionManager.sol* ( [342](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L342) ):

```solidity
/// @audit `_factory not checked`
342:     constructor(IUniswapV3Factory _factory) {
```

### [L-02] Vulnerable versions of packages are being used

This project is using specific package versions which are vulnerable to the specific CVEs listed below. Consider switching to more recent versions of these packages that don't have these vulnerabilities.
- [CVE-2023-40014](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2023-40014) - **MEDIUM** - (`openzeppelin-solidity >=4.0.0 <4.9.3`): OpenZeppelin Contracts is a library for secure smart contract development. Starting in version 4.0.0 and prior to version 4.9.3, contracts using `ERC2771Context` along with a custom trusted forwarder may see `_msgSender` return `address(0)` in calls that originate from the forwarder with calldata shorter than 20 bytes. This combination of circumstances does not appear to be common, in particular it is not the case for `MinimalForwarder` from OpenZeppelin Contracts, or any deployed forwarder the team is aware of, given that the signer address is appended to all calls that originate from these forwarders. The problem has been patched in v4.9.3.

- [CVE-2023-34459](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2023-34459) - **MEDIUM** - (`openzeppelin-solidity >=4.7.0 <4.9.2`): OpenZeppelin Contracts is a library for smart contract development. Starting in version 4.7.0 and prior to version 4.9.2, when the `verifyMultiProof`, `verifyMultiProofCalldata`, `procesprocessMultiProof`, or `processMultiProofCalldat` functions are in use, it is possible to construct merkle trees that allow forging a valid multiproof for an arbitrary set of leaves. A contract may be vulnerable if it uses multiproofs for verification and the merkle tree that is processed includes a node with value 0 at depth 1 (just under the root). This could happen inadvertedly for balanced trees with 3 leaves or less, if the leaves are not hashed. This could happen deliberately if a malicious tree builder includes such a node in the tree. A contract is not vulnerable if it uses single-leaf proving (`verify`, `verifyCalldata`, `processProof`, or `processProofCalldata`), or if it uses multiproofs with a known tree that has hashed leaves. Standard merkle trees produced or validated with the @openzeppelin/merkle-tree library are safe. The problem has been patched in version 4.9.2. Some workarounds are available. For those using multiproofs: When constructing merkle trees hash the leaves and do not insert empty nodes in your trees. Using the @openzeppelin/merkle-tree package eliminates this issue. Do not accept user-provided merkle roots without reconstructing at least the first level of the tree. Verify the merkle tree structure by reconstructing it from the leaves.

There are 2 instances:

- Global finding

## Non Critical Issues

### [N-01] Contract name does not match its filename

According to the [Solidity Style Guide](https://docs.soliditylang.org/en/latest/style-guide.html#contract-and-library-names), contract and library names should also match their filenames.

There is 1 instance:

- *ERC1155Minimal.sol* ( [10](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/tokens/ERC1155Minimal.sol#L10) ):

```solidity
/// @audit Not match filename `ERC1155Minimal.sol`
10: abstract contract ERC1155 {
```

### [N-02] Events should be emitted before external calls

Ensure that events follow the best practice of check-effects-interaction, and are emitted before external calls.

There is 1 instance:

- *SemiFungiblePositionManager.sol* ( [523](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L523) ):

```solidity
/// @audit `_mint()` is called on line 521, triggering an external call - `onERC1155Received()`
523:         emit TokenizedPositionMinted(msg.sender, tokenId, positionSize);
```

### [N-03] Contract implements interface without extending the interface

Not extending the interface may lead to the wrong function signature being used, leading to unexpected behavior. If the interface is in fact being implemented, use the `override` keyword to indicate that fact.

There is 1 instance:

- *ERC1155Minimal.sol* ( [10-20](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/tokens/ERC1155Minimal.sol#L10-L20) ):

```solidity
/// @audit IERC165
10: abstract contract ERC1155 {
11:     /*//////////////////////////////////////////////////////////////
12:                                  EVENTS
13:     //////////////////////////////////////////////////////////////*/
14: 
15:     /// @notice Emitted when only a single token is transferred
16:     /// @param operator the user who initiated the transfer
17:     /// @param from the user who sent the tokens
18:     /// @param to the user who received the tokens
19:     /// @param id the ERC1155 token id
20:     /// @param amount the amount of tokens transferred
```

### [N-04] @openzeppelin/contracts should be upgraded to a newer version

These contracts import contracts from `@openzeppelin/contracts`, but they are not using [the latest version](https://github.com/OpenZeppelin/openzeppelin-contracts/releases).

The imported version is `4.8.3`.

There are 2 instances:

- *ERC1155Minimal.sol* ( [5](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/tokens/ERC1155Minimal.sol#L5), [5](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/tokens/ERC1155Minimal.sol#L5) ):

```solidity
5: import {ERC1155Holder} from "@openzeppelin/contracts/token/ERC1155/utils/ERC1155Holder.sol";

5: import {ERC1155Holder} from "@openzeppelin/contracts/token/ERC1155/utils/ERC1155Holder.sol";
```

### [N-05] NatSpec documentation for function is missing

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation.
In complex projects such as DeFi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.

There is 1 instance:

- *SafeTransferLib.sol* ( [16](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/SafeTransferLib.sol#L16) ):

```solidity
16:     function safeTransferFrom(address token, address from, address to, uint256 amount) internal {
```

### [N-06] Variables should be named in mixedCase style

As the [Solidity Style Guide](https://docs.soliditylang.org/en/latest/style-guide.html#naming-styles) suggests: arguments, local variables and mutable state variables should be named in mixedCase style.

<details>
<summary>There are 19 instances (click to show):</summary>

- *SemiFungiblePositionManager.sol* ( [147](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L147), [152](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L152), [179](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L179), [288](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L288), [290](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L290), [296](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L296), [594](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L594), [603](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L603), [669](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L669), [1272](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L1272), [1273](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L1273), [1289](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L1289), [1290](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L1290), [1309](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L1309), [1310](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L1310), [1459](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L1459) ):

```solidity
/// @audit s_AddrToPoolIdData
147:     mapping(address univ3pool => uint256 poolIdData) internal s_AddrToPoolIdData;

/// @audit s_poolContext
152:     mapping(uint64 poolId => PoolAddressAndLock contextData) internal s_poolContext;

/// @audit s_accountLiquidity
179:     mapping(bytes32 positionKey => uint256 removedAndNetLiquidity) internal s_accountLiquidity;

/// @audit s_accountPremiumOwed
288:     mapping(bytes32 positionKey => uint256 accountPremium) private s_accountPremiumOwed;

/// @audit s_accountPremiumGross
290:     mapping(bytes32 positionKey => uint256 accountPremium) private s_accountPremiumGross;

/// @audit s_accountFeesBase
296:     mapping(bytes32 positionKey => int256 baseFees0And1) internal s_accountFeesBase;

/// @audit positionKey_from
594:             bytes32 positionKey_from = keccak256(

/// @audit positionKey_to
603:             bytes32 positionKey_to = keccak256(

/// @audit totalCollectedFromAMM
669:     ) internal returns (int256 totalCollectedFromAMM, int256 totalMoved, int24 newTick) {

/// @audit premium0X64_base
1272:             uint128 premium0X64_base;

/// @audit premium1X64_base
1273:             uint128 premium1X64_base;

/// @audit premium0X64_owed
1289:                 uint128 premium0X64_owed;

/// @audit premium1X64_owed
1290:                 uint128 premium1X64_owed;

/// @audit premium0X64_gross
1309:                 uint128 premium0X64_gross;

/// @audit premium1X64_gross
1310:                 uint128 premium1X64_gross;

/// @audit UniswapV3Pool
1459:     ) external view returns (IUniswapV3Pool UniswapV3Pool) {
```

- *Math.sol* ( [43](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/Math.sol#L43) ):

```solidity
/// @audit sqrtR
43:             uint256 sqrtR = absTick & 0x1 != 0
```

- *TokenId.sol* ( [504](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/TokenId.sol#L504), [508](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/types/TokenId.sol#L508) ):

```solidity
/// @audit isLongP
504:                     uint256 isLongP = self.isLong(riskPartnerIndex);

/// @audit tokenTypeP
508:                     uint256 tokenTypeP = self.tokenType(riskPartnerIndex);
```

</details>

### [N-07] Missing zero address check in functions with address parameters

Adding a zero address check for each address type parameter can prevent errors.

There are 3 instances:

- *SemiFungiblePositionManager.sol* ( [351](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L351) ):

```solidity
/// @audit `token0 not checked`
/// @audit `token1 not checked`
351:     function initializeAMMPool(address token0, address token1, uint24 fee) external {
```

- *ERC1155Minimal.sol* ( [77](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/tokens/ERC1155Minimal.sol#L77) ):

```solidity
/// @audit `operator not checked`
77:     function setApprovalForAll(address operator, bool approved) public {
```

### [N-08] Named mappings are recommended

[Named mappings](https://docs.soliditylang.org/en/v0.8.18/types.html#mapping-types) (with syntax `mapping(KeyType KeyName? => ValueType ValueName?)`) are recommended.It can make the mapping variables clearer, more readable and easier to maintain.

There are 2 instances:

- *ERC1155Minimal.sol* ( [62](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/tokens/ERC1155Minimal.sol#L62), [67](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/tokens/ERC1155Minimal.sol#L67) ):

```solidity
62:     mapping(address account => mapping(uint256 tokenId => uint256 balance)) public balanceOf;

67:     mapping(address owner => mapping(address operator => bool approvedForAll))
```

### [N-09] Whitespace in Expressions

See the [Whitespace in Expressions](https://docs.soliditylang.org/en/latest/style-guide.html#whitespace-in-expressions) section of the Solidity Style Guide.

<details>
<summary>There are 28 instances (click to show):</summary>

- *SemiFungiblePositionManager.sol* ( [550](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L550), [583](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L583), [711](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L711), [711](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L711), [711](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L711), [711](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L711), [711](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L711), [775](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L775), [775](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L775), [775](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L775), [775](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L775), [775](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L775), [775](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L775), [860](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L860), [1102](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L1102), [1102](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L1102) ):

```solidity
/// @audit Whitespace inside parenthesis
550:         for (uint256 i = 0; i < ids.length; ) {

/// @audit Whitespace inside parenthesis
583:         for (uint256 leg = 0; leg < numLegs; ) {

/// @audit Whitespace inside parenthesis
711:         (, newTick, , , , , ) = univ3pool.slot0();

/// @audit Whitespace before a comma
711:         (, newTick, , , , , ) = univ3pool.slot0();

/// @audit Whitespace before a comma
711:         (, newTick, , , , , ) = univ3pool.slot0();

/// @audit Whitespace before a comma
711:         (, newTick, , , , , ) = univ3pool.slot0();

/// @audit Whitespace before a comma
711:         (, newTick, , , , , ) = univ3pool.slot0();

/// @audit Whitespace inside parenthesis
775:                 (uint160 sqrtPriceX96, , , , , , ) = _univ3pool.slot0();

/// @audit Whitespace before a comma
775:                 (uint160 sqrtPriceX96, , , , , , ) = _univ3pool.slot0();

/// @audit Whitespace before a comma
775:                 (uint160 sqrtPriceX96, , , , , , ) = _univ3pool.slot0();

/// @audit Whitespace before a comma
775:                 (uint160 sqrtPriceX96, , , , , , ) = _univ3pool.slot0();

/// @audit Whitespace before a comma
775:                 (uint160 sqrtPriceX96, , , , , , ) = _univ3pool.slot0();

/// @audit Whitespace before a comma
775:                 (uint160 sqrtPriceX96, , , , , , ) = _univ3pool.slot0();

/// @audit Whitespace inside parenthesis
860:         for (uint256 leg = 0; leg < numLegs; ) {

/// @audit Whitespace inside parenthesis
1102:         (, uint256 feeGrowthInside0LastX128, uint256 feeGrowthInside1LastX128, , ) = univ3pool

/// @audit Whitespace before a comma
1102:         (, uint256 feeGrowthInside0LastX128, uint256 feeGrowthInside1LastX128, , ) = univ3pool
```

- *FeesCalc.sol* ( [101](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/FeesCalc.sol#L101), [101](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/FeesCalc.sol#L101), [101](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/FeesCalc.sol#L101), [101](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/FeesCalc.sol#L101), [101](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/FeesCalc.sol#L101), [102](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/FeesCalc.sol#L102), [102](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/FeesCalc.sol#L102), [102](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/FeesCalc.sol#L102), [102](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/FeesCalc.sol#L102), [102](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/libraries/FeesCalc.sol#L102) ):

```solidity
/// @audit Whitespace inside parenthesis
101:         (, , uint256 lowerOut0, uint256 lowerOut1, , , , ) = univ3pool.ticks(tickLower);

/// @audit Whitespace before a comma
101:         (, , uint256 lowerOut0, uint256 lowerOut1, , , , ) = univ3pool.ticks(tickLower);

/// @audit Whitespace before a comma
101:         (, , uint256 lowerOut0, uint256 lowerOut1, , , , ) = univ3pool.ticks(tickLower);

/// @audit Whitespace before a comma
101:         (, , uint256 lowerOut0, uint256 lowerOut1, , , , ) = univ3pool.ticks(tickLower);

/// @audit Whitespace before a comma
101:         (, , uint256 lowerOut0, uint256 lowerOut1, , , , ) = univ3pool.ticks(tickLower);

/// @audit Whitespace inside parenthesis
102:         (, , uint256 upperOut0, uint256 upperOut1, , , , ) = univ3pool.ticks(tickUpper);

/// @audit Whitespace before a comma
102:         (, , uint256 upperOut0, uint256 upperOut1, , , , ) = univ3pool.ticks(tickUpper);

/// @audit Whitespace before a comma
102:         (, , uint256 upperOut0, uint256 upperOut1, , , , ) = univ3pool.ticks(tickUpper);

/// @audit Whitespace before a comma
102:         (, , uint256 upperOut0, uint256 upperOut1, , , , ) = univ3pool.ticks(tickUpper);

/// @audit Whitespace before a comma
102:         (, , uint256 upperOut0, uint256 upperOut1, , , , ) = univ3pool.ticks(tickUpper);
```

- *Multicall.sol* ( [14](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/multicall/Multicall.sol#L14) ):

```solidity
/// @audit Whitespace inside parenthesis
14:         for (uint256 i = 0; i < data.length; ) {
```

- *ERC1155Minimal.sol* ( [141](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/tokens/ERC1155Minimal.sol#L141) ):

```solidity
/// @audit Whitespace inside parenthesis
141:         for (uint256 i = 0; i < ids.length; ) {
```

</details>

### [N-10] Empty bytes check is missing

Passing empty bytes to a function can cause unexpected behavior, such as certain operations failing, producing incorrect results, or wasting gas. It is recommended to check that all byte parameters are not empty.

<details>
<summary>There are 4 instances (click to show):</summary>

- *SemiFungiblePositionManager.sol* ( [407-411](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L407-L411), [441-445](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/SemiFungiblePositionManager.sol#L441-L445) ):

```solidity
/// @audit data
407:     function uniswapV3MintCallback(
408:         uint256 amount0Owed,
409:         uint256 amount1Owed,
410:         bytes calldata data
411:     ) external {

/// @audit data
441:     function uniswapV3SwapCallback(
442:         int256 amount0Delta,
443:         int256 amount1Delta,
444:         bytes calldata data
445:     ) external {
```

- *ERC1155Minimal.sol* ( [90-96](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/tokens/ERC1155Minimal.sol#L90-L96), [128-134](https://github.com/code-423n4/2023-11-panoptic/blob/f75d07c345fd795f907385868c39bafcd6a56624/contracts/tokens/ERC1155Minimal.sol#L128-L134) ):

```solidity
/// @audit data
90:     function safeTransferFrom(
91:         address from,
92:         address to,
93:         uint256 id,
94:         uint256 amount,
95:         bytes calldata data
96:     ) public {

/// @audit data
128:     function safeBatchTransferFrom(
129:         address from,
130:         address to,
131:         uint256[] calldata ids,
132:         uint256[] calldata amounts,
133:         bytes calldata data
134:     ) public virtual {
```

</details>

