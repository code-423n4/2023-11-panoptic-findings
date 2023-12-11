## Overview

Panoptic is the perpetual, oracle-free options protocol built on the Ethereum blockchain. The protocol’s intuitive design ensures a seamless user experience, enabling users to easily navigate through various features and perform operations effortlessly. Users can interact with the Panoptic protocol without the need for intermediaries like banks, brokerage firms, clearinghouses, market makers, or centralized exchanges.

## My thoughts
Pantopic has built a next-level crypto asset options trading protocol without the use of an oracle which may be susceptible to vulnerabilities and by extension exploits on the protocol. Also, the options do not expire.

Pantopic's approach leverages on the concentrated liquidity mechanism.  In Panoptic, market making involves responding to both the demand from buyers and the need to stabilize prices within a Uniswap v3 pool. That may mean having to sell OTM options or providing liquidity in response to the put skew of an underlying asset. Or buy close to the money options when the implied volatility is low compared to the realized volatility. Of course, seasoned market makers will need to do so in a way that maintains delta-neutrality at the portfolio level.


## Mechanism review
In scope for this Audit are 13 smart contracts of which the `SemiFungiblePositionManager.sol` (SFPM) is at the heart of the Panoptic options trading as it handles the core liquidity management.
It works thus:
- users deposit funds into the `Panopticpool` which is not in scope for this audit.
- option buyers and sellers deposit collateral into the Panoptic pool (AKA panpool) a
- traders interact with the SFPM through the functions in the panpool open and close/exercise their option positions.
- There are 10 libraries some of which are unique feature implementations of Panoptic protocol and some others suboptimal implementations of Uniswap v3 features. These libraries provide helper functions in different parts of the code
- Users (options buyers and sellers) can open and close long and short positions In-The-Money (ITM) and Out-of-The-Money (OTM).
- A user opening/selling a short position will move liquidity from the panpool to uniswap V3 pool within a preferred price range.
- A user opening/buying a long position will remove liquidity from the  Uniswap V3 pool and pay a premium accrued which is equivalent to the fees the removed liquidity would have earned in the Uniswap V3 pool



## Audit approach

I followed below steps while analyzing and auditing the code base.
- Read the Audit Readme.md and took the required notes.
- Read the docs in Panoptic's website
- Study of documentation to understand each contract purposes, its functionality, how it is connected with other contracts, etc. The protocol is heavily reliant on uniswap V3.
- Run the tests to checks all test passed. I used foundry to test moonwell protocol.
- Finally, I started with the auditing the code base in depth way I started understanding line by line code and took the necessary notes to ask some questions to sponsors

## Audit Stages
- In the first stage of the audit, I focused on understanding the protocol usage in more detail having read the docs many times over. This involved identifying and analyzing the important contracts and functions within the system. By examining these key components, the audit aimed to gain a comprehensive understanding of the protocol’s functionality and potential risks.

- During the second stage of the audit, my focus was on thoroughly examining and marking any doubtful or vulnerable areas within the protocol noting how parts of the UNISWAP V3 code that was rewritten would introduce vulnerabilities.

- During the third stage of the audit, a comprehensive analysis and test cases (considering that most of the test cases were fuzz tests) of the previously identified doubtful and vulnerable areas were conducted. (Mostly false positives were found)

## Learnings
The Panoptic protocol team is building a novel project in the crypto asset options trading space and as such does not have other Defi protocols to look up that may have faced challenges with building with such projects without an oracle.

By prioritizing security, conducting thorough risk assessments, and actively involving the community in governance decisions, the Pantopic Protocol can proactively address these areas of concern and strengthen its position as a secure and community-driven DeFi lending and borrowing platform


## Codebase quality analysis
Despite being relying heavily on Uniswap V3 functionalities, the codebase is properly documented leaving room for improvement. 

## Possible Systemic Risks
- `Smart Contract Vulnerabilities`: Implementing some some Uniswap V3 features from scratch to use in the protocol  can introduce potential smart contract vulnerabilities. Example of such functions in the codebase are 
`getAmount0ForLiquidity(...)` and 
`getAmount1ForLiquidity(...)`
in the `Math.sol` Libraries and also 
`getLiquidityForAmount0(...)`
`getLiquidityForAmount1(...)`
which are perceived as suboptimal representations that are designed to match Uniswap's implementation

- In the `SFPM`, when a tokenised position is being burnt, the `isLong` bits are flipped changing the tokenId, but one can argue that since the `tokenId`

### Time spent:
90 hours