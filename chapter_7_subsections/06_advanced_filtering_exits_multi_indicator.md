[← Back to Outline](../outline.md)

---

# Advanced Filtering and Exits: SMA + RSI + ADX + ATR + Bracket Orders

Building upon our previous SmaCrossFiltered strategy, we can create a more sophisticated approach by incorporating additional filtering mechanisms and explicit risk management through bracket orders. This enhanced version demonstrates how multiple indicators can work together to create more selective entry signals and better position management.

## Enhanced Strategy Concept

**Objective**: Create a more selective entry signal and manage risk/profit more explicitly. This version requires not only SMA crossover and non-overbought RSI conditions but also confirmation from trend strength (rising ADX) and volatility indicators (rising ATR) before entering positions.

**Risk Management**: Once entered, trades are managed through predefined percentage-based stop-loss and take-profit levels, while maintaining the original bearish SMA crossover as an alternative exit condition.

## New Strategy Components

### Additional Indicators

**ADX (Average Directional Index)**: Measures trend strength. We filter entries based on whether ADX is increasing from the previous bar, indicating strengthening trend conditions.

**ATR (Average True Range)**: Measures market volatility. We filter entries based on whether ATR is increasing, potentially signaling the beginning of stronger price movements (though this filter requires careful testing for effectiveness).

### Risk Management Parameters

**Percentage-Based Stops**: Defined through strategy parameters (`stop_loss_pct`, `take_profit_pct`) to provide consistent risk/reward ratios.

**Bracket Orders**: Utilizes `self.buy_bracket()` functionality to place three simultaneous orders:
1. **Main BUY order** to enter the position
2. **SELL STOP order** (stop-loss) at `entry_price × (1 - stop_loss_pct)`
3. **SELL LIMIT order** (take-profit) at `entry_price × (1 + take_profit_pct)`

When the main BUY order executes, the stop and limit orders become active. If either the stop or limit order triggers, backtrader automatically cancels the other.

## Strategic Advantages

### Multi-Layer Filtering

The enhanced approach requires alignment across multiple market dimensions:
- **Trend Direction**: SMA crossover
- **Momentum State**: RSI levels
- **Trend Strength**: ADX movement
- **Volatility Context**: ATR changes

This multi-dimensional approach aims to identify high-probability setups where various market forces align favorably.

### Automatic Risk Management

**Immediate Protection**: Stop-loss orders activate immediately upon position entry, providing downside protection without requiring manual monitoring.

**Profit Capture**: Take-profit orders ensure gains are realized when predetermined targets are reached, removing emotion from exit decisions.

**Systematic Approach**: Bracket orders enforce trading discipline by pre-defining both risk and reward parameters.

## Implementation Considerations

### Filter Effectiveness

**ADX Rising Filter**: Generally provides valuable confirmation of trend development, though it may filter out some early trend opportunities.

**ATR Rising Filter**: Requires careful evaluation as increasing volatility can indicate either opportunity or increased risk, depending on market context.

### Parameter Selection

**Stop-Loss Percentage**: Should balance capital protection with normal market fluctuation tolerance. Too tight may result in frequent stop-outs; too wide may allow excessive losses.

**Take-Profit Percentage**: Should reflect realistic profit expectations while allowing for trend continuation. Consider risk-reward ratios (e.g., 2:1 reward-to-risk).

### Order Management Complexity

**Multiple Order Tracking**: Bracket orders introduce additional complexity in order status monitoring and error handling.

**Position Synchronization**: Ensure proper coordination between automatic bracket exits and manual SMA crossover exits.

## Expected Outcomes

### Improved Selectivity

The additional filters should reduce the number of trades but increase the quality of entry signals by requiring multiple confirmations.

### Enhanced Risk Control

Explicit stop-loss and take-profit levels provide better capital preservation and profit capture compared to signal-reversal-only exits.

### Performance Trade-offs

- **Potential Benefits**: Higher win rate, better risk-adjusted returns, reduced maximum drawdown
- **Potential Costs**: Fewer trading opportunities, increased complexity, possible over-filtering of profitable signals

The next section provides the complete implementation code for this enhanced strategy, demonstrating how these concepts translate into practical backtrader code.


---

[← Back to Outline](../outline.md)