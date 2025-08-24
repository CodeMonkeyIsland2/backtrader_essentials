[← Back to Outline](../outline.md)

---

# Bollinger Bands Strategy: Mean Reversion Approach

## Bollinger Bands Overview

Bollinger Bands form a volatility-based technical analysis tool consisting of three components:
- **Middle Band**: Simple Moving Average (typically 20 periods)
- **Upper Band**: Middle band + (Standard Deviation × multiplier, typically 2.0)
- **Lower Band**: Middle band - (Standard Deviation × multiplier, typically 2.0)

The bands dynamically expand during high volatility periods and contract during low volatility phases, providing adaptive support and resistance levels.

## Mean Reversion Strategy Concept

### Core Logic: Fading the Extremes

The mean reversion approach using Bollinger Bands operates on the principle that price touches to the outer bands represent statistical extremes that are likely unsustainable. This creates trading opportunities when prices deviate significantly from their recent average.

**Strategy Framework:**
- **Entry Signal**: Price closes below the lower band (oversold condition)
- **Exit Signal**: Price returns above the middle band (reversion toward mean)
- **Risk Management**: Position sizing based on band width and volatility

### Signal Generation Rules

**Long Entry Conditions:**
- Current bar's close < Lower Bollinger Band
- No existing position
- Optional: Confirm with volume or other filters

**Position Exit Conditions:**
- Price crosses back above middle band (SMA)
- Alternative exits: Stop loss, profit target, or time-based exit

## Implementation in Backtrader

```python
import backtrader as bt

class BollingerMeanReversionStrategy(bt.Strategy):
    params = (
        ('bb_period', 20),           # Moving average period
        ('bb_deviation', 2.0),       # Standard deviation multiplier
        ('debug_logging', True),     # Enable detailed logging
    )
    
    def log(self, txt, dt=None):
        """Enhanced logging method"""
        if self.params.debug_logging:
            dt = dt or self.datas[0].datetime.date(0)
            print(f'{dt.isoformat()} - {txt}')
    
    def __init__(self):
        self.log(f'Initializing {self.__class__.__name__}')
        
        # Data references
        self.dataclose = self.datas[0].close
        
        # Bollinger Bands indicator
        self.bbands = bt.indicators.BollingerBands(
            self.datas[0],
            period=self.params.bb_period,
            devfactor=self.params.bb_deviation
        )
        
        # Order tracking
        self.order = None
        
        self.log(f'Bollinger Bands configured: period={self.params.bb_period}, '
                f'deviation={self.params.bb_deviation}')
    
    def notify_order(self, order):
        """Order status notification handler"""
        if order.status in [order.Submitted, order.Accepted]:
            return  # Order pending
            
        if order.status == order.Completed:
            if order.isbuy():
                self.log(f'BUY EXECUTED: Price={order.executed.price:.2f}, '
                        f'Value={order.executed.value:.2f}, '
                        f'Commission={order.executed.comm:.2f}')
            elif order.issell():
                self.log(f'SELL EXECUTED: Price={order.executed.price:.2f}, '
                        f'Value={order.executed.value:.2f}, '
                        f'Commission={order.executed.comm:.2f}')
        elif order.status in [order.Canceled, order.Margin, order.Rejected]:
            self.log(f'Order {order.getstatusname()}')
        
        self.order = None  # Reset order tracking
    
    def notify_trade(self, trade):
        """Trade completion notification"""
        if trade.isclosed:
            self.log(f'TRADE CLOSED: Gross PnL={trade.pnl:.2f}, '
                    f'Net PnL={trade.pnlcomm:.2f}')
    
    def next(self):
        """Strategy logic executed on each bar"""
        # Skip if pending orders exist
        if self.order:
            return
        
        # Current market data
        current_close = self.dataclose[0]
        lower_band = self.bbands.bot[0]
        middle_band = self.bbands.mid[0]
        upper_band = self.bbands.top[0]
        
        # Position entry logic
        if not self.position:
            # Mean reversion entry: Buy when price drops below lower band
            if current_close < lower_band:
                self.log(f'ENTRY SIGNAL: Close ({current_close:.2f}) below '
                        f'Lower Band ({lower_band:.2f})')
                self.order = self.buy()
        
        else:
            # Position exit logic
            # Exit when price returns to middle band (mean reversion complete)
            if current_close > middle_band:
                self.log(f'EXIT SIGNAL: Close ({current_close:.2f}) above '
                        f'Middle Band ({middle_band:.2f})')
                self.order = self.close()
```

## Strategy Analysis

### Advantages

**Adaptive to Market Conditions:**
- Bands automatically adjust to volatility changes
- Wider bands in volatile markets, narrower in calm periods
- Self-calibrating support/resistance levels

**Clear Signal Definition:**
- Objective entry and exit criteria
- Quantifiable risk parameters
- Backtestable rules

**Effective in Range-Bound Markets:**
- Profits from price oscillations within trading ranges
- Captures mean reversion moves effectively
- Works well when trends are absent

### Limitations and Risks

**Trend Following Weakness:**
- Dangerous during strong trending periods
- "Walking the bands" phenomenon in sustained moves
- Can generate multiple losing trades in trending markets

**Parameter Sensitivity:**
- Performance varies significantly with period and deviation settings
- Standard 20/2.0 parameters may not suit all instruments
- Requires optimization for different timeframes

**Exit Strategy Dependency:**
- Middle band exit may be premature or too late
- Alternative exits (opposite band, time-based) affect performance
- Stop loss placement is crucial but challenging

## Enhancement Considerations

### Filtering Mechanisms
- **ADX Filter**: Only trade when ADX < 25 (indicating ranging market)
- **Volume Confirmation**: Require above-average volume on entry signals
- **Multiple Timeframe**: Confirm signals with higher timeframe trend

### Risk Management
- **Position Sizing**: Scale position size based on band width
- **Stop Losses**: Place stops below recent swing lows
- **Profit Targets**: Set targets at opposite band or fixed R:R ratios

### Advanced Variations
- **Band Squeeze**: Trade breakouts after band contraction
- **Double Touch**: Require multiple touches before entry
- **Divergence**: Combine with momentum oscillator divergence

## Market Application Guidelines

**Optimal Conditions:**
- Sideways trending markets
- High volatility environments
- Assets with strong mean-reverting characteristics

**Avoid During:**
- Strong trending periods
- Low volatility environments
- News-driven markets with fundamental bias

This mean reversion approach using Bollinger Bands provides a systematic method for trading statistical extremes, but requires careful market regime analysis and robust risk management protocols.


---

[← Back to Outline](../outline.md)