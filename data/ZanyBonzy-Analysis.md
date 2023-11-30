#  **Advanced Analysis Report for  [<img src="https://panoptic.xyz/img/logo-mono-white.svg" alt="Panoptic Logo" width="auto" height="auto"/>](https://code4rena.com/contests/2023-11-panoptic#top)** 


##  **1. Overview**
### **1.1 Protocol Overview**
   <img src="https://github.com/panoptic-labs/panoptic-v1-core/blob/main/assets/banner.png" alt="Panoptic Banner" width="auto" height="auto"/>

- The Panoptic protocol is a decentralized options trading platform on the Ethereum blockchain. It allows users to create, trade, and market-make perpetual options without the need for intermediaries or oracles. This is achieved by leveraging the decentralized architecture of Automated Market Makers and the permissionless liquidity pools of UniswapV3.
- Unlike conventional options, Panoptic options employ UniswapV3 Liquidity Provider (LP) positions as the core mechanism for trading long and short options. These options are perpetual, meaning they have no expiration date, and can be freely created by anyone in any options market for any asset without any permissions required. Furthermore, Panoptic options are not affected by "vega," which ensures that the premiums of open positions remain unaffected by volatility fluctuations.

### **1.2 Architecture and CodeBase Overview**

#### **1.2.1 Codebase**

- In context of the audit, the Panoptic protocol comprises 13 smart contracts with a total codebase of 1676 SLoC. Its primary design strategy is composition, and it incorporates two external imports. The protocol is planned to be deployed on the Ethereum and pther EVM chains that support active UniswapV3 deployments. The `SemiFungiblePositionManager`, the protocol's core contract, adheres to the ERC1155 token standard. It lacks an AMM, oracle, or fork with any well-known projects. The protocol leverages all ERC20 tokens, excluding fee-on-transfer tokens.

- Similar to other codebases built using composition patterns, the Panoptic protocol's code presented was initially challenging to understand. Nonetheless, the code is thoroughly commented, providing detailed explanations for each function's functionality. However, the website documentation seems to be outdated, as it omits certain contracts included in the audit scope. We recommend updating the documentation to enhance user comprehension and streamline future audits.
  
- The protocol demonstrates exemplary test coverage, a testament to the developers' dedication to quality assurance. Unit tests were rigorously executed for all contracts. Considering the intricate nature of certain codebases, we strongly advocate for the incorporation of invariant fuzzing tests. These tests possess the power to uncover rudimentary bugs, strengthen modularity, and elevate overall security measures.

- The protocol also exhibits robust error handling practices. A blend of custom and require errors is implemented, with a preference for custom errors. To promote uniformity across the codebase, we recommend updating the `Math.sol` contract with custom errors, as they offer superior gas efficiency. However, these custom errors currently lack descriptive explanations. Incorporating such descriptions would enhance code readability and simplify the process of pinpointing parameters responsible for errors.
  
#### **1.2.2 Contract Overview**
  <img src="https://github.com/panoptic-labs/panoptic-v1-core/blob/main/assets/Smart%20Contracts_1.png" alt="Panoptic Smart COntracts" width="50%" height="auto"/>

In scope, the contracts are divided into five categories;

**Core contract**
 - **[SemiFungiblePositionManager.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/SemiFungiblePositionManager.sol)** - lies at the heart of the protocol, acting as an alternative to UniswapV3's `NonfungiblePositionManager`. Through this contract, users can effortlessly initialize their chosen UniswapV3 pools, mint or burn their desired tokenized positions, and access vital information regarding a pool's liquidity and fee structure. Compliant with the ERC1155 standard, the contract seamlessly integrates UniswapV3 pool and factory interfaces, features UniswapV3 mint/burn callbacks, and comprises 666 SLoC.
   
 <img src="https://panoptic.xyz/img/SFPM-architecture.svg" alt="SFPM" width="auto" height="auto"/>

**Token contract**
 - **[ERC1155Minimal.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/tokens/ERC1155Minimal.sol)** - is the protocol's implementation of the ERC1155 token standard, specifically tailored for the Panoptic protocol's needs. It is a modified version of Solmate's `ERC1155` contract and integrates the OZ `ERC1155Holder` contract. The contract consists of 129 SLoC.

**Multicall contract**
 - **[Multicall.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/multicall/Multicall.sol)** - enables the efficient execution of multiple methods within a single contract call, making it an ideal tool for batch operations. It consists of 18 SLoC.

**Type contracts**
- **[LeftRight.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LeftRight.sol)** - is responsible for implementing a set of custom data types designed to store two 128-bit numbers. It comprises 91 SLoC.

- **[LiquidityChunk.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/LiquidityChunk.sol)**	- is responsible for implementing a specialized data type tailored to represent a liquidity chunk of a given size within the Uniswap ecosystem. It comprises 45 SLoC.

- **[TokenId.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/types/TokenId.sol)** -  is responsible for implementing the custom data type utilized by the `SemiFungiblePositionManager` to encode position information into 256-bit ERC1155 tokenIds. It encapsulates a pool identifier and supports up to four comprehensive position legs. The contract comprises 241 SLoC.

**Libraries**

 - **[CallbackLib.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/CallbackLib.sol)** - is the library for validating and decoding Uniswap callbacks. It comprises 36 SLoC.

 - **[Constants.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Constants.sol)** - holds the constants used in the protocol contracts, including UniswapV3 min and max tick prices, uniswap hash init code, etc. It comprises 13 SLoC.	

- **[Errors.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Errors.sol)** - contains all custom errors used in Panoptic's core contracts. It comprises 18 SLoC.		

- **[FeesCalc.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/FeesCalc.sol)** - is the library for fee calculations. It comprises 52 SLoC.

- **[Math.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/Math.sol)**	- is a concatenation of various UniswapV3 math contracts including TickMath, SafeCast, BitMath etc. It comprises 266 SLoC.

- **[PanopticMath.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/PanopticMath.sol)** - contains advanced Panoptic/Uniswap-specific functions. It comprises 82 SLoC.

- **[SafeTransferLib.sol](https://github.com/code-423n4/2023-11-panoptic/blob/main/contracts/libraries/SafeTransferLib.sol)** - is a modified version of Solmate's SafeTransferLib library. It comprises 19 SLoC.

#### **1.2.3 Contract Interactions graph**

<img src="https://github-production-user-asset-6210df.s3.amazonaws.com/112232336/286749676-38eb1d57-d0fe-4cc4-850c-0dd01bd1d521.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20231129%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20231129T214216Z&X-Amz-Expires=300&X-Amz-Signature=1bdea58dc88043de0abbf75c7cf90f8be5b7d2974f1b20fa73aa974c55e4e637&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=0" alt="Contract Architecture" width="auto" height="auto"/>

## **2. Risks to protocol**

- **Option Trading Risks** - Engaging in options trading carries inherent risks, and the Panoptic protocol along with its users are not exempt from these risks. Potential issues include margin calls, liquidations (or the inability to liquidate due to insufficient liquidity), exceptionally high leverages and premiums, depegging of purportedly stablecoin assets, and so forth. These factors warrant careful consideration by protocol users. 

- **ThirdParty dependencies** - The Panoptic protocol relies on UniswapV3 contracts to operate effectively. As a result, any vulnerabilities or errors in the UniswapV3 protocol could potentially compromise the security and functionality of the Panoptic protocol. Additionally, despite being primarily composed through code composition, the Panoptic protocol incorporates and modifies certain OZ and Solmate contracts. Consequently, vulnerabilities in these external contracts could also introduce risks to the protocol.

- **Smart Contract vulnerabilities** - Smart contract vulnerabilities pose a significant threat to the security and integrity of decentralized protocols. These vulnerabilities can manifest as critical security flaws, enabling malicious actors to exploit the protocol and steal funds or manipulate the system. Logical inconsistencies in the code can also lead to unexpected behavior and unintended consequences, potentially causing financial losses or compromising the protocol's overall functionality.
  
- **Non-standard ERC20 tokens risks** - The protocol supports a wide range of ERC20 tokens, excluding fee-on-transfer tokens, which offers users a greater choice of assets for trading and position management. However, this versatility also introduces potential risks associated with non-standard ERC20 tokens. These non-standard tokens deviate from the expected behavior of the ERC20 standard, potentially leading to unexpected behaviours or vulnerabilities that could be exploited by malicious actors.
  
- **Centralization Risks** - No significant centralization risks were identified in the contracts in scope as the protocol appears to be permissionless.

- **Multichain Compatibility** - Due to the inherent variations in blockchain implementations and their respective properties. Transactions that execute flawlessly on one chain may encounter errors or produce unintended consequences on other chains. This heterogeneity introduces risks for users interacting with the protocol on these susceptible chains.

## **3. Audit approach**
We approached the audit in 3 general steps after which we generated our report.

- **Documentation review** - We reviewed the docs, readme, video and explanations provided by the devs. While this was going on, we ran the contracts through slither and compared the generated reports to the bot report.

- **Manual code review** - Here, we manually reviewed the codebase, ran provided tests, tested out various attack vectors. We looked for ways to DOS the system, ways a user can remove other users liquidity, pay more fees than thy're entiltled to and so on. We tested out the functions' logic to make sure they work as intended, as well as making sure the contracts comply with the needed EIPs.

- **Codebase comparisons** - After the manual review, we used solodit to check for any similar protocol types so as to compare their implementations and tried to find any general vulnerabilities. This was a bit difficult, as the protocol seems to be one of a kind.

### 4. Conclusions

- Options are one of the most traded instruments in finance ant the Panoptic protocol aims to redefine options trading by introducing a groundbreaking approach that leverages the power of decentralized finance. Unlike traditional options contracts that involve third parties, Panoptic enables the creation of option-like payoffs by dynamically redistributing liquidity among participants within the ecosystem. This mechanism eliminates the need for intermediaries and opens up the possibility of trading options on any token pair that has a UniswapV3 pool.
- Overall, the codebase stands out for its well-structured organization and adherence to high-quality standards. The development team has meticulously crafted a permissionless and gas-efficient alternative to UniswapV3's `NonFungiblePositionManager`. Nevertheless, there remains potential for further improvement. We recommend cleaning up the codebase and mititgating any identified vulnerabilities before deployment. This proactive approach will ensure the protocol's continued robustness and security.


### Time spent:
030 hours