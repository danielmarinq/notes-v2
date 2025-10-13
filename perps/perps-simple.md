# Perpetual Futures: A Simple Guide

## What Are Perpetual Futures?

**Perpetual futures** (or "perps") are derivative contracts that let you trade the price of an asset without owning it, and without an expiration date. Unlike traditional futures that expire monthly or quarterly, perpetual futures can be held forever.

**Key Innovation:** The **funding mechanism** keeps the perpetual price close to the spot price through periodic payments between traders.

---

## Core Concepts

### Positions

- **Long Position:** You profit when price goes up (bullish)
- **Short Position:** You profit when price goes down (bearish)
- **Position Size:** How many contracts you hold (positive = long, negative = short)

### Leverage and Margin

- **Leverage:** Control larger positions with less capital (e.g., 10x leverage = $1,000 controls $10,000)
- **Margin:** The collateral you put up to open a position
- **Notional Value:** The total USD value of your position

$$\text{Notional Value} = \text{Position Size} \times \text{Current Price}$$

### Profit and Loss (PnL)

**Unrealized PnL:** Current profit/loss on open positions
$$\text{Unrealized PnL} = \text{Position Size} \times (\text{Current Price} - \text{Entry Price})$$

**Realized PnL:** Profit/loss locked in when you close positions

---

## Price Types and Discovery

### Index Price
The **index price** represents the external reference price, typically calculated as a weighted average of spot prices from major exchanges (e.g., Coinbase, Binance, Kraken). This serves as the "true" underlying asset price.

### Last Trade Price
The **last trade price** is the most recent execution price on the perpetual futures exchange.

### Mark Price  
The **mark price** is the "fair value" price used for:
- Calculating unrealized PnL
- Determining margin requirements
- Triggering liquidations
- Funding rate calculations

**Mark Price Calculation Methods:**

1. **Oracle-Anchored Method:**
$$\text{Mark Price} = (1-\text{Weight}) \times \text{Index Price} + \text{Weight} \times \text{TWAP}[\text{Last Price}]$$

2. **Impact Bid/Ask Method:**
$$\text{Mark Price} = \text{Index Price} + \text{Clamped Premium Adjustment}$$

The mark price is designed to be manipulation-resistant while staying close to the true asset value.

---

## The Funding Mechanism

### Why Funding Exists
Without expiration, there's no natural force to keep the perpetual price close to the spot price. Funding creates economic incentives for price convergence.

### How It Works

**When perp trades above spot (premium):**
- Longs pay funding to shorts
- This encourages selling → price goes down

**When perp trades below spot (discount):**
- Shorts pay funding to longs  
- This encourages buying → price goes up

### Funding Rate Calculation

**Basic Formula:**
$$\text{Funding Rate} = \text{Premium Rate} + \text{Interest Rate}$$

Where:
$$\text{Premium Rate} = \frac{\text{Mark Price} - \text{Index Price}}{\text{Index Price}}$$

**Funding Payment:**
$$\text{Payment} = \text{Position Size} \times \text{Mark Price} \times \text{Funding Rate}$$

- **Positive funding rate:** Longs pay shorts
- **Negative funding rate:** Shorts pay longs
- **Funding frequency:** Usually every 8 hours

---

## Trading Mechanics

### Order Types

**Market Orders:** Execute immediately at the best available price
- Higher execution certainty but subject to slippage
- Pay taker fees

**Limit Orders:** Execute only at specified price or better  
- Better price control but execution not guaranteed
- Pay maker fees (often rebates)

**Post-Only Orders:** Must be maker orders (cannot execute immediately)
- Guaranteed maker fees/rebates
- Rejected if would execute immediately

**Reduce-Only Orders:** Can only decrease existing position size
- Used for closing positions without risk of increasing exposure

### Trading Fees

**Taker Fee:** Applied to orders that remove liquidity (market orders)
**Maker Fee:** Applied to orders that add liquidity (limit orders, often negative)

$$\text{Fee Amount} = \text{Executed Notional} \times \text{Fee Rate}$$

---

## Risk Management and Collateral

### Multi-Collateral System

Modern perpetual exchanges support multiple collateral types with risk-adjusted valuations:

$$\text{Collateral Value} = \sum \text{Amount} \times \text{Price} \times (1 - \text{Haircut})$$

Where each collateral type has its own haircut (risk adjustment).

**Typical Haircuts:**
- Stablecoins (USDC, USDT): 0-2%
- Major cryptocurrencies (BTC, ETH): 5-10%
- Altcoins: 10-25%

### Account Equity
$$\text{Account Equity} = \text{Collateral Value} + \text{Total Unrealized PnL} - \text{Total Funding Owed}$$

### Margin Requirements

**Initial Margin Ratio:** Required to open positions (e.g., 10% for 10x leverage)
**Maintenance Margin Ratio:** Minimum to avoid liquidation (e.g., 5%)

For tiered margin systems (larger positions need higher margins):
$$\text{Maintenance Margin Rate} = \text{Base Rate} + \text{Size Adjustment}$$

**Margin Requirements:**
$$\text{Initial Margin} = \text{Total Notional} \times \text{Initial Margin Rate}$$
$$\text{Maintenance Margin} = \text{Total Notional} \times \text{Maintenance Margin Rate}$$

### Margin Modes

**Cross-Margin:** Single equity pool shared across all positions
- More capital efficient
- Profits in one market offset losses in another
- Higher correlation risk

**Isolated Margin:** Separate margin allocation per market  
- Risk contained per market
- Less capital efficient
- Better risk isolation

### Leverage Metrics

$$\text{Margin Ratio} = \frac{\text{Account Equity}}{\text{Total Notional Value}}$$

$$\text{Effective Leverage} = \frac{\text{Total Notional Value}}{\text{Account Equity}}$$

---

## Liquidation and Solvency

### Liquidation Trigger

$$\text{Account Equity} \leq \text{Maintenance Margin Requirement}$$

Equivalently:

$$\text{Margin Ratio} \leq \frac{\text{Maintenance Margin}}{\text{Total Notional}}$$

### Liquidation Process

**Partial Liquidation:** Close minimum position $\phi \in (0,1]$ to restore health
- Objective: Find minimal $\phi$ such that post-liquidation margin ratio exceeds target
- Preferred method to minimize trader impact

**Full Liquidation:** Close entire position if partial liquidation insufficient
- Last resort when account cannot be restored to health

**Liquidation Penalties:**
$$\text{Penalty} = \text{Penalty Rate} \times \text{Liquidated Notional}$$
where penalty rate is typically 0.5-2.5%

### Liquidation Price Calculation

**For Long Positions:**
$$\text{Liquidation Price} = \frac{\text{Entry Price} \times \text{Size} - \text{Available Margin}}{\text{Size} \times (1 - \text{Maintenance Rate})}$$

**For Short Positions:**
$$\text{Liquidation Price} = \frac{\text{Entry Price} \times \text{Size} + \text{Available Margin}}{\text{Size} \times (1 + \text{Maintenance Rate})}$$

### Insurance Fund and Bankruptcy

**Insurance Fund:** Pool of funds that absorbs losses from bankrupt accounts

**Sources:**
- Liquidation penalties
- Excess margin from profitable liquidations
- Exchange contributions

**Bankruptcy Handling:**
When liquidation results in negative equity, the insurance fund covers the loss.

### Auto-Deleveraging (ADL)

When insurance fund insufficient, **Auto-Deleveraging** socializes losses among profitable traders.

**ADL Ranking Score:**
$$\text{ADL Score} = \text{PnL Direction} \times \sqrt{\frac{\text{Profit Amount}}{\text{Account Equity}}}$$

Higher scores are deleveraged first (most profitable + highest leverage).

---

## Comprehensive Formula Reference

### Position and PnL Calculations

| Concept | Formula |
|---------|---------|
| **Notional Value** | $\text{Position Size} \times \text{Mark Price}$ |
| **Unrealized PnL** | $\text{Position Size} \times (\text{Mark Price} - \text{Entry Price})$ |
| **Realized PnL** | $\text{Closed Size} \times (\text{Exit Price} - \text{Entry Price})$ |
| **Account Equity** | $\text{Collateral} + \text{Total PnL} - \text{Total Funding}$ |

### Price Discovery

| Concept | Formula |
|---------|---------|
| **Oracle-Anchored Mark** | $(1-\text{Weight}) \times \text{Index} + \text{Weight} \times \text{TWAP}$ |
| **Impact Mark Price** | $\text{Index Price} + \text{Clamped Premium}$ |
| **TWAP** | $\frac{1}{\text{Window}} \times \sum \text{Recent Prices}$ |

### Funding Mechanism

| Concept | Formula |
|---------|---------|
| **Premium Rate** | $\frac{\text{Mark Price} - \text{Index Price}}{\text{Index Price}}$ |
| **Funding Rate** | $\text{Premium Rate} + \text{Interest Rate}$ |
| **Funding Payment** | $\text{Position Direction} \times \text{Notional} \times \text{Funding Rate}$ |

### Risk Management

| Concept | Formula |
|---------|---------|
| **Collateral Value** | $\sum (\text{Amount} \times \text{Price} \times (1-\text{Haircut}))$ |
| **Initial Margin** | $\text{Total Notional} \times \text{Initial Rate}$ |
| **Maintenance Margin** | $\text{Total Notional} \times \text{Maintenance Rate}$ |
| **Margin Ratio** | $\frac{\text{Account Equity}}{\text{Total Notional}}$ |
| **Effective Leverage** | $\frac{\text{Total Notional}}{\text{Account Equity}}$ |

### Liquidation

| Concept | Formula |
|---------|---------|
| **Liquidation Trigger** | $\text{Account Equity} \leq \text{Maintenance Margin}$ |
| **Long Liquidation Price** | $\frac{\text{Entry} \times \text{Size} - \text{Available Margin}}{\text{Size} \times (1 - \text{Maint Rate})}$ |
| **Short Liquidation Price** | $\frac{\text{Entry} \times \text{Size} + \text{Available Margin}}{\text{Size} \times (1 + \text{Maint Rate})}$ |
| **Liquidation Penalty** | $\text{Penalty Rate} \times \text{Liquidated Notional}$ |

### Trading Mechanics

| Concept | Formula |
|---------|---------|
| **Trading Fee** | $\text{Executed Notional} \times \text{Fee Rate}$ |
| **Slippage** | $\frac{\text{Actual Price} - \text{Expected Price}}{\text{Expected Price}}$ |

---

## Practical Example

**Setup:**
- Open 1 BTC long position at $50,000
- Current mark price: $52,000
- Account has $10,000 collateral
- Maintenance margin: 5%

**Calculations:**
- **Notional Value:** 1 × $52,000 = $52,000
- **Unrealized PnL:** 1 × ($52,000 - $50,000) = $2,000
- **Account Equity:** $10,000 + $2,000 = $12,000
- **Margin Ratio:** $12,000 / $52,000 = 23%
- **Leverage:** $52,000 / $12,000 = 4.3x

**Funding Example:**
- Funding rate: 0.01% (longs pay)
- Funding payment: 1 × $52,000 × 0.0001 = $5.20

**Liquidation Price:**
- Maintenance margin needed: $52,000 × 5% = $2,600
- Available margin: $10,000 - $2,600 = $7,400
- Liquidation price ≈ $50,000 - $7,400/1 = $42,600

---

## Risk Factors

### For Traders
- **Leverage amplifies losses** as well as gains
- **Funding costs** can accumulate over time
- **Liquidation risk** if positions move against you
- **Market volatility** can cause rapid PnL changes

### System Risks
- **Oracle failures** can affect mark prices
- **Extreme market conditions** may trigger auto-deleveraging
- **Liquidity crunches** can worsen liquidation outcomes

---

## Best Practices

1. **Understand leverage:** Higher leverage = higher risk
2. **Monitor funding rates:** Long-term costs can be significant  
3. **Set stop losses:** Don't rely only on liquidation protection
4. **Size positions appropriately:** Don't risk more than you can afford to lose
5. **Understand the mechanics:** Know how funding and liquidation work

---

## Comprehensive Glossary

### Core Concepts

| Term | Definition |
|------|------------|
| **Perpetual Futures** | Derivative contracts with price exposure to underlying assets but no expiration date |
| **Position Size** | Signed quantity of contracts held (positive = long, negative = short) |
| **Notional Exposure** | USD value of a position (Size × Mark Price) |
| **Long Position** | Position that profits when asset price increases |
| **Short Position** | Position that profits when asset price decreases |

### Price Types

| Term | Definition |
|------|------------|
| **Index Price** | Oracle-provided external reference price for underlying asset |
| **Mark Price** | Fair value price used for margining, PnL, and liquidations |
| **Last Trade Price** | Most recent execution price on the exchange |
| **Entry Price** | Volume-weighted average price of position |
| **TWAP** | Time-Weighted Average Price over specified window |

### PnL and Accounting

| Term | Definition |
|------|------------|
| **Unrealized PnL** | Current profit/loss on open positions |
| **Realized PnL** | Profit/loss locked in when positions are closed |
| **Account Equity** | Total account value including collateral and PnL |
| **Funding Payments** | Cumulative funding owed by trader |

### Funding Mechanism

| Term | Definition |
|------|------------|
| **Funding Rate** | Periodic payment rate between long and short holders |
| **Premium Rate** | Measure of perpetual price deviation from index |
| **Interest Rate** | Cost of capital component in funding |
| **Funding Payment** | Actual payment amount per funding period |

### Risk Management

| Term | Definition |
|------|------------|
| **Margin** | Collateral required to open and maintain positions |
| **Initial Margin** | Margin required to open new positions |
| **Maintenance Margin** | Minimum margin to avoid liquidation |
| **Margin Ratio** | Account equity divided by total notional exposure |
| **Leverage** | Total notional divided by account equity |
| **Haircut** | Risk adjustment applied to collateral values |

### Trading Mechanics

| Term | Definition |
|------|------------|
| **Market Order** | Order that executes immediately at best available price |
| **Limit Order** | Order that executes only at specified price or better |
| **Post-Only** | Order that must be maker (cannot execute immediately) |
| **Reduce-Only** | Order that can only decrease existing position size |
| **Taker Fee** | Fee for orders that remove liquidity |
| **Maker Fee** | Fee/rebate for orders that add liquidity |
| **Slippage** | Difference between expected and actual execution price |

### Liquidation and Solvency

| Term | Definition |
|------|------------|
| **Liquidation** | Forced closure of positions when margin requirements violated |
| **Liquidation Price** | Price at which position would be liquidated |
| **Liquidation Penalty** | Fee charged during liquidation process |
| **Insurance Fund** | Pool of funds to cover losses from bankrupt accounts |
| **Auto-Deleveraging (ADL)** | Mechanism to socialize losses when insurance insufficient |
| **Bankruptcy** | When account equity becomes negative after liquidation |

### System Components

| Term | Definition |
|------|------------|
| **Oracle** | External data source providing price information |
| **CLOB** | Central Limit Order Book - matching engine for orders |
| **Cross-Margin** | Single equity pool shared across all positions |
| **Isolated Margin** | Separate margin allocation per market |
| **Tiered Margin** | Margin requirements that increase with position size |

---

*This guide covers the essential mechanics of perpetual futures. Always understand the risks before trading with leverage.*
