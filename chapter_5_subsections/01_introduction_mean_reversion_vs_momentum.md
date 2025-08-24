[← Back to Outline](../outline.md)

---

# Introduction: Mean Reversion vs. Momentum

## Core Trading Philosophies

In algorithmic trading, two primary strategic approaches dominate the landscape: **mean reversion** and **momentum strategies**. Understanding their fundamental differences is crucial for developing effective trading systems.

### Mean Reversion Philosophy

Mean reversion strategies operate on the principle that asset prices tend to return to their statistical average over time. This approach suggests that:

- **Price extremes are temporary** - When an asset moves significantly away from its historical average, it's likely to correct back toward that mean
- **Contrarian positioning** - Traders sell assets that have risen sharply (anticipating pullbacks) and buy assets that have declined significantly (expecting rebounds)
- **Statistical basis** - The strategy assumes that extreme price movements represent temporary market inefficiencies

**Core assumption**: Price deviations from the mean are unsustainable and will eventually revert.

### Momentum/Trend-Following Philosophy

Momentum strategies are built on the premise that existing market trends tend to persist. This philosophy emphasizes:

- **Trend continuation** - Assets that are rising are likely to continue rising, and falling assets are likely to continue falling
- **Following the crowd** - Buy assets showing strength and sell (or short) assets showing weakness
- **Market psychology** - Leverages behavioral patterns where winning trades attract more buyers, creating self-reinforcing cycles

**Core assumption**: "The trend is your friend" - established price movements will continue until a clear reversal signal appears.

## Strategic Contradictions

These approaches often generate opposing signals for the same market conditions:

- A sharp upward price move might trigger a **sell signal** for mean reversion (expecting a pullback) but a **buy signal** for momentum (expecting continuation)
- The effectiveness of each approach depends heavily on current **market regime** and conditions

## Market Environment Considerations

### When Mean Reversion Works Best
- **Range-bound markets** - Sideways trending conditions where prices oscillate within defined boundaries
- **High volatility periods** - When price swings are exaggerated, creating more reversion opportunities
- **Oversold/overbought conditions** - Markets that have moved too far, too fast

### When Momentum Strategies Excel
- **Trending markets** - Clear directional movements with sustained momentum
- **Breakout scenarios** - When prices break through significant support/resistance levels
- **News-driven moves** - Fundamental events that create lasting directional bias

## Technical Indicator Alignment

Different technical indicators naturally align with specific philosophies:

**Mean Reversion Indicators:**
- Bollinger Bands (trading bounces off outer bands)
- RSI (trading exits from overbought/oversold zones)
- Stochastic oscillators

**Momentum Indicators:**
- MACD (signal line crossovers)
- Moving average crossovers
- Price rate of change

## Chapter Focus

In this chapter, we'll examine practical implementations of both approaches using three fundamental indicators:

1. **Bollinger Bands** - Applied for mean reversion from statistical extremes
2. **RSI** - Used for momentum exhaustion and reversal signals
3. **MACD** - Implemented for momentum shift detection

Each strategy will be coded, analyzed, and discussed in terms of strengths, weaknesses, and optimal market conditions for deployment.


---

[← Back to Outline](../outline.md)