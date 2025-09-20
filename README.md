# Turing Market

## Introduction

Turing M is an AI collective data-driven event derivative financial ecosystem. Built on Web3 technology, it establishes a decentralized prediction market platform focusing on political/sports/entertainment multi-domain event prediction trading, breaking through traditional financial market limitations.

## Updates 0926: deploy to Kairos testnet

* Kairos Version:
  * Demo Video: https://youtu.be/HXbBNglQJbI
  * Mobile DApp: https://kaia-test-lite.turingmarket.cc
  * LIFF: https://line.me/R/app/2008136886-Nke5LwLP
* Contract deployment: 
  * TuringMarketApp: https://kairos.kaiascan.io/tx/0x94282f4694dcb7a22291dbf5ac0896955cab4bed1ea890b251cd220679775b5c?tabId=internalTx&page=1
* Desktop version:
  * Dapp: https://turingmarket.cc/
  * Demo Video: https://youtu.be/ni6MiigAUMg
* Pitch Video: https://x.com/Turingmarket/status/1930945177381982507
* Pitch Deck: https://github.com/TuringM-Labs/jutopia2025/blob/main/TuringM-v3.4.pdf
* X: https://x.com/turingmarket

## Features:

1. Using kaia-native USDT to pay for conditional tokens used to predict events will significantly increase kaia-native USDT usage.
2. Complete user experience (smart contracts + dApps).
3. A series of hacker growth and viral marketing features will be launched intensively to achieve massive user growth.

## How we address the pain points of current crypto products:

1. Pain Points of Current Crypto Products
* Pure coin issuance, FOMO (Fear of Momentum) – Too many products are actually Ponzi schemes.
* Complex features and incomprehensible terminology: Various technical terms are difficult for users to understand and understand, making Web3 difficult to gain public attention.
2. How TuringMarket Addresses These Challenges

a). TuringMarket utilizes a COB model, with off-chain matching and on-chain transactions. This significantly improves user transaction efficiency while ensuring the security and reliability of user assets.

b). On mobile, it employs a Tinder-like swipe-left-right interface, transforming boring transactions into easily understandable yes/no responses, significantly lowering the barrier to entry for users.

c). By introducing various hacker growth and viral marketing modules, The application of various crazy viral extension mechanisms of Web2 to Web3 will soon attract a huge influx of Web2 users to the platform.

## Links
* Website: https://TuringM.IO/
* White Paper && Docs:
  * English: https://docs-en.turingm.io/
  * Chinese: https://docs-zh.turingm.io/
* Insights: https://insights.turingmarket.cc/
* X: https://twitter.com/turingmarket
* Full Smart Contract Audit: https://github.com/TuringM-Labs/TuringM-EIP2535

![1.png](./assets/1.png)

## Core Features

![2.png](./assets/2.png)

1. **AI Data Driven**: Implements Model Context Protocol (MCP) to integrate collective data for real-time market probability calibration
2. **Hybrid Trading Mechanism**: Supports multiple binary mutual-exclusive markets enabling complex strategies like hedging portfolios
3. **Liquidity Innovation**: Provides 7x24 instant market access with up to 10x leveraged trading
4. **Token Economy Loop**: TUIT token integration across trading/governance/staking scenarios forms demand growth flywheel

![3.png](./assets/3.png)

## Competitive Advantages

![4.png](./assets/4.png)

1. **Prediction Accuracy**: 42% improvement in event analysis efficiency compared to competitors
2. **User Incentives**: Includes token dividends (50% profit sharing) + prediction leaderboard + event proposal rewards
3. **Compliance Assurance**: Smart contract auto-settlement + compensation fund (50% operational cost) + anti-manipulation algorithms
4. **Community Co-creation**: Supports user-proposed event markets (proposers receive 1-3% transaction shares)

![5.png](./assets/5.png)

## Operational Benchmarks

![6.png](./assets/6.png)

1. **Competitor Reference**: Polymarket achieved 292% MAU growth (30K → 314.5K) in 2024 with annual trading volume exceeding $9B
2. **Revenue Structure**: Dual engine of transaction fees (2%) + outcome commissions (2%) targeting $120M first-year volume
3. **Distribution Model**: 70% for market expansion (agent/KOL commissions), 30% for token buyback & burn

![7.png](./assets/7.png)

## Differentiation Innovations

![8.png](./assets/8.png)

1. **Prediction+Derivatives**: Pioneers event option contracts supporting long/short positions
2. **Social Trading**: Built-in opinion sharing/copy-trading system with planned DAO governance integration
3. **Cross-Market Arbitrage**: Enables correlated trading strategies between real-world events and crypto market volatility

![9.png](./assets/9.png)

## Architecture

![flow.jpg](./assets/flow.jpg)

### Overview

The `TuringMarketFacet` contract implements atomic swaps between binary outcome tokens (ERC1155) and collateral assets (ERC20). 

It adopts a hybrid decentralized exchange model where operators provide off-chain matching services while settlement is completed trustlessly on-chain through signed order messages.

Turing Market supports matching orders containing mint/merge operations, allowing cross-matching of complementary outcome token orders. Orders use EIP712 standard structured data signature format.

Additionally, the contract implements a symmetric fee mechanism. When orders match, one party acts as maker and the other as taker, with price improvement benefits going to the taker side.

### Matching Scenarios

#### Asset Definitions

* **`A`** - ERC1155 outcome token
* **`A'`** - ERC1155 complementary outcome token*
* **`C`** - ERC20 collateral token
* All examples assume `A` and `A'` are priced at 0.5 USDC each

*Note: Complementary relationship means 1 outcome token + 1 complementary token can be merged into 1 collateral token, and vice versa (i.e., **`A`** + **`A'`** = **`C`**). Assume outcome tokens and collateral tokens have the same decimal precision. Examples assume **`C`** is USDC.

#### Scenario 1 - Minting Transaction

* user1 buys 100 `A`, transfers 50 `C` to exchange
* user2 buys 100 `A'`, transfers 50 `C` to exchange
* Exchange mints 100 `A` for user1, 100 `A'` for user2

```js
const takerOrder = {
    salt: randomId(),
    maker: user1,
    tokenId: 1, // Token ID for A'
    tokenAmount: 100 * 10**6,
    tokenPriceInPaymentToken: 0.5 * 10**6, // 0.5 USDC (could be 0 for market orders)
    paymentTokenAddress: '0xxxxx', // USDC address
    slippageBps: 100, // Allowed slippage (0 indicates limit order)
    deadline: 1672531200, // Expiration timestamp
    side: 1, // 1: buy, 2: sell
    feeTokenAddress: '', // Fee token address (USDC or company token)
    
    // Signature fields
    sig: 'xxxxx', // Frontend signature of above data
    
    // Off-chain calculated transaction details
    exchangeNftAmount: 100 * 10**6, // NFT amount maker will receive
    paymentTokenAmount: 100 * 10**6, // USDC amount maker must pay
    paymentTokenAddress: '0xxxxx', // Payment token address (USDC)
    
    // Dual fee fields for multi-token fee support
    fee1Amount: 123, // Primary fee amount
    fee1TokenAddress: '0xxxxx', // Primary fee token (USDC/company token)
    fee2Amount: 456, // Secondary fee amount
    fee2TokenAddress: '0xxxxx' // Secondary fee token (USDC/company token)
}

const makerOrders = [{
    salt: randomId(),
    maker: user2,
    tokenId: 2, // Token ID for A'
    tokenAmount: 100 * 10**6,
    tokenPriceInPaymentToken: 0.5 * 10**6,
    paymentTokenAddress: '0xxxxx',
    slippageBps: 100,
    deadline: 1672531200,
    side: 1,
    feeTokenAddress: '',
    sig: 'xxxxx',
    exchangeNftAmount: 100 * 10**6,
    paymentTokenAmount: 100 * 10**6,
    paymentTokenAddress: '0xxxxx',
    fee1Amount: 123,
    fee1TokenAddress: '0xxxxx',
    fee2Amount: 456,
    fee2TokenAddress: '0xxxxx'
}]

matchOrders(takerOrder, makerOrders)
```

#### Scenario 2 - Regular Transaction

* user1 buys 100 `A`, transfers 50 `C` to exchange
* user2 sells 100 `A`, receives 50 `C` from exchange

#### Scenario 3 - Merge Transaction

* user1 sells 100 `A`
* user2 sells 100 `A'`
* Exchange merges 100 `A` + 100 `A'` into 100 `C`
* Exchange transfers 50 `C` to user1
* Exchange transfers 50 `C` to user2

### Fee Mechanism

Fees are collected from output assets (proceeds). For complementary binary tokens (**`A`** + **`A'`** = **`C`**), fees must maintain symmetry to preserve market integrity. Symmetry means selling 100 `A` at $0.99 and buying 100 `A'` at $0.01 should incur equal fee value.

**Rules:**
- Buy operations (receiving **`A`**/**`A'`**) charge fees on output tokens
- Sell operations (receiving **`C`**) charge fees on collateral tokens

Base fee rate (`baseFeeRate`) is written into order structure, corresponding to 2x fee rate when tokens are equally priced ($0.50/$0.50). Price deviations use formula:

`USDC fee = baseFeeRate * min(price, 1-price) * token quantity`

*Example 1:*
- price = 0.2
- feeBps = 100
- fee = 0.01 * min(0.2, 0.8) * 100 = 0.2

*Example 2:*
- Selling 100 `A` at $0.99:  
  0.01 * min(0.99, 0.01) * 100 = 0.01 (deduct 0.01 `C` as fee)
- Buying 100 `A'` at $0.01:  
  0.01 * min(0.01, 0.99) * 100 = 0.01 (deduct 1 `A'` as fee)

![10.png](./assets/10.png)

![11.png](./assets/11.png)


## Team Information

- **Jesse** CEO 10+ years SaaS software market expansion experience
- **RichRed** Investment Director Years of Crypto market investment experience
- **Sam** Chief Talent Officer
- **David** CTO Years of CEX architecture experience, delivered multiple CEX projects
- **George** CPO 10+ years internet financial product leadership experience
- **Adam** Chief Development Engineer 5+ years blockchain development, 10+ years full-stack experience
- **Rushen** Project Director & Agile Coach Years of project management experience
