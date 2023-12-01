## Mechanism review

The Panoptic protocol consists of smart contracts on the Ethereum blockchain that handle the minting, trading, and market-making of perpetual put and call options. All smart contracts are available 24/7 and users can interact with the Panoptic protocol without the need for intermediaries like banks, brokerage firms, clearinghouses, market makers, or centralized exchanges.

Panoptic is the first permissionless options protocol that overcomes the technically challenging task of implementing an options protocol on the Ethereum blockchain. We achieve this by embracing the decentralized nature of Automated Market Makers and permissionless liquidity providing in [Uniswap v3](https://uniswap.org/).

#### Liquidity Providers (LPs)

Panoptic empowers LPs with increased yield, impermanent loss (IL) mitigation, advanced strategies, and risk management.

#### Increased Yield

LPs who deposit liquidity into AMMs like Uniswap v3 are unable to earn additional yield on their LP token. This is because Uniswap v3 represents LP tokens as NFTs, which are harder to integrate into the rest of the DeFi ecosystem.

Through Panoptic's innovative [Semi-Fungible Position Manager](https://panoptic.xyz/docs/developers/semifungiblepositionmanager), LPs are able to lend out their LP tokens for additional yield. LPs still earn all of the fees they would have been entitled to on Uniswap, plus an additional yield that comes from options buyers who are borrowing the LP token.

![panoptic](https://i.imgur.com/D7C261u.png)

#### Impermanent Loss (IL) Mitigation

LPs don't need to feel trapped by IL when they understand options. IL is simply gamma risk in options trading: the accelerating risk of assets conversion away from the initial 50:50 ratio.

LPs can use Panoptic to control and decrease the amount of IL suffered. To mitigate IL, LPs can:

Use Panoptic to deploy liquidity onto Uniswap. Deploying through Panoptic enables your LP token to be lent out to other traders, which increases your profitability and decreases your IL.
Buy perpetual options to transform naked positions into spreads.
Keep track of and neutralize your delta and gamma.
Trade wide positions to limit pin risk.

#### Advanced strategies

LPs are selling perpetual put options, whether they know it or not. Panoptic embraces this discovery by utilizing concentrated liquidity pool on Uniswap v3 to create option payoffs for users. Instead of only selling put options, LPs can use Panoptic as a one-stop shop to also buy put options, buy call options, sell call options, trade delta-neutral straddles, and more. Stop being restricted to only trading neutral-to-bullish positions in AMMs, and start exploring the world of bearish, delta-neutral, bullish, long volatility, and short volatility trades!

#### Risk Management

LPs can take advantage of Panoptic's visual interface, that allows them to:

Choose the right LP width based on their desired timescale.
Easily monitor their position's P&L, delta, gamma, and other Greeks.
Decide when to open or close an LP position based off of the risk dashboard's implied volatility (IV) metric
Rebalance their LP position through Panoptic's gas-efficient and low-cost rolling mechanism.

## Perp Traders

Enhance your Perpetual Trading experience with Panoptic's perpetual options. Trade any token permissionlessly with leverage, capped downside, no liquidation risk, and no expiries in the non-custodial, oracle-free options platform.

- Any Token
Users can trade perps and perpetual options on any ERC-20 token, thanks to the permissionless nature of Panoptic and Uniswap.

- Permissionless
Depositing, trading, pricing, settlement, listing, and liquidating are all on chain and permissionless, thanks to the transparent and decentralized platform that blockchain technology enables.

- Leverage
Users can buy perpetual options with up to 10x leverage, making for bigger potential gains (and losses).

- Capped Downside
Perpetual options provide a major advantage over perps for traders to limit their downside risk. For example, buying a perpetual put option on ETH gives you unlimited upside should the price of ETH fall, while capping your downside should the price of ETH rise. Contrast this with ETH perps which have unlimited downside should the price of ETH rise.

- No liquidation Risk
Users can purchase perpetual options without liquidation risk. Some long calls and puts have no liquidation price thanks to the capped downside they offer. Unlike perps, options on Panoptic are protected against liquidations caused by single-wick price movements in the underlying asset.

While long options in Panoptic can still be liquidated, the risk is more manageable due to the gradual accumulation of premia owed and the defined risk of the position.

- No Expiries
Perps traders enjoy the flexibility of not having to roll and manage expiring futures contracts. Similarly, traders on Panoptic enjoy the hassle-free perpetuality of non-expiring options contracts.

- Non-custodial
Panoptic is a completely non-custodial, decentralized exchange. All funds are held and managed automatically on the blockchain through public, audited smart contracts.

- Oracle-free
Panoptic is a groundbreaking protocol enabling perpetual options trading without oracles. Pricing is based on Uniswap liquidity provider (LP) fees and is completely on chain. Because of its oracle-free nature, Panoptic is able to support the immediate listing of options on any ERC-20 token without having to wait for oracle support.

- Synthetic Perps
Users can go long or short by trading leveraged perps on Panoptic. The ability to create synthetic futures contracts by combining puts and calls carries over to crypto - users can create synthetic perps by combining perpetual puts and perpetual calls on Panoptic.

## Option traders

Options Traders can leverage Panoptic's full-featured platform. Users can buy and sell capital-efficient, multi-legged options strategies on any token at any strike price.

- Full Flexibility
Panoptic offers the full flexibility that options traders are familiar with, without sacrificing liquidity. Here is how Panoptic avoids restrictions pervasive in DeFi options:

Ability to support large sizes: Panoptic enables unlimited selling of options. When there are no buyers, sellers are still compensated through Uniswap trading fees. Perpetual options remove expiries to consolidate liquidity. Panoptic renders OTC options on longtail assets obsolete since Panoptic is oracle-free and enables immediate listing of any ERC-20 token pair.

Keeping variety without losing liquidity: Traders can sell options at any strike price without limitation. Traders can choose whether to buy or sell options, which strike to underwrite, what type of option to trade (put, call, or any multi-leg combination), with full capital efficiency.

Liquidity providers (LPs) can define their risk exposure: Liquidity provision in Panoptic (PLP) is separate from options selling. Users can choose between passive provision (no delta risk) and active option selling (has delta risk). Options sellers will be equipped with risk management tools on Panoptic such as P&L visualization, liquidation prices, and Greeks calculations.

A balanced, two-sided market: Panoptic enables a two-sided market where traders can buy or sell perpetual options (i.e. go short or long LP tokens). This enables better price discovery for the LP market to come to equilibrium regarding its implied volatility (IV), pricing, and size.

Capital can be withdrawn: Panoptic implements dynamic collateral requirements, dynamic spreads, and forced exercising in order to discourage over-utilization of liquidity.

- Capital-Efficient
Options in Panoptic are optimized for capital-efficiency. Traders can buy options with up to 10x leverage, and sell options with up to 5x leverage. More advanced strategies like straddles, strangles, and spreads can also be traded on leverage, allowing for lower upfront capital commitment and higher potential gains (and losses).

- Multi-legged
Traders can combine multiple puts and calls ("legs") at various strike prices and timescale to create more advanced strategies. Traditional multi-leg strategies such as straddles, strangles, spreads, iron condors, and jade lizards can all be used in Panoptic.

- Any Token
Users can trade perpetual options on any ERC-20 token, thanks to the permissionless nature of Panoptic and Uniswap.

- Any Strike
Traders can sell perpetual options at any strike price, thanks to the permissionless nature of Panoptic and Uniswap. Traders can buy perpetual options at any strike price so long as there is sufficient liquidity sold at the desired strike.

## Systemic risks

Options trading is not for everyone. Like any form of leveraged trading, trading options is associated with significant risks. Any user that wishes to interact with the Panoptic protocol has to understand the risks involved.

The main risks are:

- Over-exposure: losing due to leveraged exposure.
- Unfavorable pricing: paying a large premium for options.
- Forced exercise: having far-the-money positions closed by external actors/liquidators.
- Liquidations: losing deposited collateral because of a margin call.

## High level overview

![overview](https://i.imgur.com/j5XiPhh.png)

## Codebase strenghs

The core idea behind Perpetual Options is that Uniswap v3 liquidity provider (LP) positions can be seen as tokenized short puts. This core result emerges from the simple observation that providing concentrated liquidity in Uniswap v3 generates a payoff that is mathematically identical to selling a put option.

![relation](https://i.imgur.com/iogaBBS.png)

- Codebase is well written and respects the solidity best practices.
- Codebase is fully decentralized, this is awesome as it erases any potential centralization risk.
- Comments and natspec were really helpfull, they were really detailled and self-explanatory.
- Redondants codes are refactored into modifiers.
- Documentation is well written and self-explanatory.
- Codebase was already audited which makes it even harder to find any issue.

## Codebase weaknesses

- Majority of codebase is usinf a fixed version of solidity that can makes it harder to maintain in the future.
- Audit scope is only a small part of the protocol so it does not reveal the true NSloc a warden need to understand to get going with the audit.

## Audits approach

- Day 1: read the docs and understand the protocol.
- Day 2: Delve deep into the codebase to get a general code architecture understanding.
- Day 3: Delve more deeper into uniswap V3 to understand more the code.
- Day 4: Compile the findings and write the report.

### Time spent:
20 hours