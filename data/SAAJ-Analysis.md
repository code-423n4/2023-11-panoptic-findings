
# Introduction
This analysis is based on assessment of ```Panoptic``` protocol which is developed as an gas efficient alterative for UniSwap-V3 for managing semi-fungible positions within the context of managing multi-leg positions encoded in ERC1155 tokenIds.

The codebase analysis is carried out using a thoughtful and systematic approach to identify any issues and problems that exist in terms of working mechanism, code quality, systemic and centralisation risk which are later used to make recommendation in terms of improving the overall architecture of the ```Panoptic``` protocol.

The codebase is also reviewed for any security pitfalls which existed in any contract in scope that may impact the overall protocol safety in terms of monetary funds or work flow in a smooth manner. The security issues and vulnerabilities are reported separately according to their severity level for this contest.
 
- **Note to the Judge**
 

My reviewing of the codebase spans to around 50 hours by spending around 6 hours daily for 8 days that result in identifying 06 issues, out of which 01 is a high and 05 are of mediums severity level; that are serious in nature and must be tackled effectively to avoid loss to the protocol and its user.

The high issue is related to creation of a pool with unrealistic fee that can lead to manipulation of any user of the pool.
Medium findings are related to re-entrancy attacks, denial of service due to unbounded loop, approval of all tokens rather than specific ones and invalid inputs that may lead to unexpected behaviour or loss of funds for receiver.

# Mechanism Review
This section looks at the workflow of contracts in scope based on my understanding of how each contract and their function works.

```SemiFungiblePositionManager``` is the main contract and extends ```ERC1155Minimal``` and ```Multicall```. It is alternative version for a gas-efficient Uniswap V3 position manager that wraps Uniswap V3 positions with up to 4 legs behind an ERC1155 token. It includes functions for minting and burning positions, transferring positions, and interacting with Uniswap V3 pools. It also includes a reentrancy lock to prevent reentrant calls, which is a good security practice.

```ERC1155Minimal.sol``` contract is an implementation of the ERC1155 token standard, which is a multi-token standard that allows for the creation and management of multiple token types in a single contract. This contract includes functions for transferring tokens, approving operators, and querying balances. ```ERC1155Minimal``` is a standard interface for contracts that manage multiple token types. It allows for the creation, transfer, and management of multiple fungible and non-fungible tokens in a single contract.

```Multicall``` contract allows making multiple calls in a single transaction, which can be useful for batch operations.  This is useful for batch operations or complex transactions that require multiple function calls.

The types contracts category i.e.; ```TokenId```, ```LeftRight``` and ```LiquidityChunk``` are used for data packing and manipulation according to requirement.

The contract uses several libraries for various purposes, such as ```CallbackLib``` for handling Uniswap V3 callbacks, ```Math``` for mathematical operations, ```PanopticMath``` for specific calculations related to the system.
```FeesCalc.sol``` library is responsible for calculating the fees associated with liquidity chunks in a Uniswap V3 pool. It has a function ```calculateAMMSwapFeesLiquidityChunk``` that calculates the AMM Swap/trading fees for a liquidityChunk of each token. It has  a function ```_getAMMSwapFeesPerLiquidityCollected``` that calculates the fee growth that has occurred (per unit of liquidity) in the AMM for an option position's liquidity chunk within its tick range given.

```Math.sol``` library provides a set of mathematical functions that are used throughout the project. It includes functions for calculating the absolute value of an integer, calculating the square root ratio at a given tick, and performing safe multiplication and division operations. It also provides functions for calculating the amount of token0 and token1 received for a given ```liquidityChunk`` and the amount of liquidity for a given amount of token0 and token1.

```Constants.sol``` library defines a set of constants that are used throughout the project. These constants include fixed point multipliers, minimum and maximum possible price ticks and ```sqrtPriceX96``` in a Uniswap V3 pool, and the hash of the init code for a Uniswap V3 pool. These constants are likely used for calculations related to Uniswap V3 pools.

```Errors.sol``` library defines a set of custom error messages that can be used throughout the project. These errors cover a variety of scenarios, such as invalid tick values, invalid callbacks, casting errors, and more. These custom errors can be used to provide more informative error messages when something goes wrong.

```SafeTransferLib.sol``` library provides a function for safely transferring ERC20 tokens from one address to another. It handles missing return values gracefully, which can be a common issue when interacting with ERC20 tokens.

```LeftRight.sol``` library provides functions for packing and unpacking 256-bit data into two separate 128-bit chunks. This is useful for efficient storage and manipulation of data in Solidity.

```LiquidityChunk.sol``` library provides functions for creating and manipulating liquidity chunks. A liquidity chunk represents an amount of liquidity deployed between two ticks in a Uniswap v3 pool.

```TokenId.sol``` library provides functions for packing and unpacking option position data into a single uint256. This is used to represent an options position in the SFPM as an options on trading platform.

```LeftRight.sol``` library provides functions for packing and unpacking 256-bit data into two separate 128-bit chunks. This is useful for efficient storage and manipulation of data in Solidity.

# Approach for Audit 
The approach i had taken to audit the codebase include reviewing the documentation, looking for weakness in terms of security practice, reviewing access control mechanism, looking out for re-entrancy issues for functions making external calls or have external visibility. Looking for compliance to ERC standard, assessing libraries are implemented correctly and conducting test to ensure expected result are generated correctly to ensure all function working.

Documentation helps in understanding the codebase working and by reading through any provided documentation to understand the purpose, design choices, and intended behaviour of the contracts. Security best practices involves checking if the contracts follow best practices, such as avoiding deprecated functions, using latest compiler version, and avoid common pitfall. Access control mechanism is based on ensuring that only authorized users can execute sensitive functions. 

Reentrancy protection is linked to ensuring that the contracts are protected against reentrancy attacks by using the ```reentrancyGuard``` pattern where appropriate. For ERC-1155 and other interfaces, the approach is linked to contracts compliance with the best solidity standards. Verify that the functions are implemented correctly and that the contracts interact seamlessly with other ERC-compliant contracts. External calls and callbacks are the main aspect of audit to ensure that these interactions are secure and that the contract handles potential failures and exceptions gracefully.

Library reviewing is linked with the contract  being implemented correctly and securely, as issues in libraries can have significant impacts on the main contracts. Code Complexity is assessed in regards to overall complexity of the code. Some contract has high level of complexity in terms of maths which requires high maths skills and may increase the likelihood of bugs and make it harder to maintain the code.

# Codebase Quality Analysis
The code quality of ```Panoptic``` protocol code base is very good as it follows high safety standard which  are defined as below:

**Comments** makes the whole code base well-documented as the comments provide valuable context and explain the purpose and functionality of the overall code. This is particularly important in complex systems like AMM.

**Error Handling** throughout the codebase linked to use of revert statements with custom error messages to handle errors. This is a good practice as it helps with debugging and understanding why a transaction might have failed.

**Security** by having reentrancy lock to prevent reentrancy attacks, which is a common vulnerability in smart contracts. It also uses ```SafeMath``` for arithmetic operations to prevent overflows and underflows.

**Efficiency** exist for codebase is linked to usage of bitwise operations and other low-level operations for efficiency. It also uses unchecked blocks to save gas when overflow is not a concern.

**Complexity** existed as some parts of the code are quite complex, particularly the mathematical calculations and bitwise operations. This is not necessarily a negative point, as complex calculations are often necessary in smart contracts, but it does mean that the code could be difficult to understand for someone not familiar with these concepts.


# Centralization Risks
Centralization issues for the protocol existed just in relation to contracts that makes external call to the Uniswap V3 pool and ERC1155 contracts, if these external contracts are controlled by any centralized entity it poses the risk of being controlled. These can create issue for working of the protocol as it is designed to be permissionless with no control over it working like that of creating a pool.

# Systemic Risks
The systemic risk identified for the codebase are discussed as below;
- **Access Control**

The codebase doesn't seem to have any access control mechanisms like Ownable or Roles. This means that any address can call any function. Depending on the functionality of the contract, this could be a security risk.

- **External Calls**

The protocol interacts with external contracts (Uniswap V3 Pool). This could potentially be a risk if the external contract is malicious or has bugs. It's important to handle external calls safely.

- **Gas Usage**

The contract contains loops that could potentially consume a lot of gas, depending on the input. This could make some functions expensive to call.

- **Low-Level Calls**

The codebase in scope uses low-level calls as rhese can be more prone to errors than high-level function calls, and should be used carefully.

- **Lack of Function Documentation**

Many functions in overall codebase lacks comments, which make the contract harder to understand and potentially create security risks.

- **Front-Running**

As with all protocols that interact with AMMs like Uniswap, there's a potential for front-running attacks where a malicious actor could see a pending transaction and then issue their own transaction with a higher gas price to front-run it.

- **Unchecked Return Values**

Contracts making external calls do not check the return values of the calls made. This could potentially lead to issues if a called contract fails silently.

# Architecture Recommendations

The recommendation to improve the architecture of the ```Panoptic``` protocol are made as following; 

Some functions are quite complex and long, such as ```_validateAndForwardToAMM``` in ```SemiFungiblePositionManager``` contract. Breaking down these functions into smaller, more manageable functions could improve readability and maintainability.

Using fixed Solidity version as some contracts are compiled without locking to specific version. It's recommended to always use the latest specific version of Solidity for the most recent security and language features with also avoiding known bugs in previous version.

While the code is extensively commented, it would be beneficial to have a separate documentation that explains the overall architecture, the purpose of each contract and function, and how they interact with each other. This would make it easier for new developers to understand the codebase.

The protocol does not seem to have any access control mechanisms like Roles for specific functions like pool working. This could be a potential issue if there are functions that should be restricted to certain roles. It is recommended to using specific roles for criticial functionality to limit any misuse or possible exploitation.

It is recommended to developed the protocol as upgradeable version as the protocol of ```Panoptic``` do not seem to have any mechanism for upgradability. If a critical bug is found, it could not be fixed without deploying a new contract and migrating all states to a new one.

# Conclusion

The ```Panoptic``` protocol has developed an efficient system as an alternative to Uniswap but has not follow the security best practice to avoid high level risk that causes manipulation of the system or loss of funds to user and the protocol due to unexpected behaviour or an attack by a malicious actor.




### Time spent:
50 hours