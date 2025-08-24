[← Back to Outline](../outline.md)

---

# Using the Custom Indicator in a Strategy

## Integration Principles

Once you've developed a custom indicator class (such as our `RateOfChange` example), incorporating it into a backtrader strategy follows the exact same patterns used for built-in indicators. This seamless integration is one of backtrader's key strengths—custom analytical tools behave identically to the framework's native indicators.

## Strategy Implementation Steps

### 1. Indicator Instantiation in `__init__`

Create instances of your custom indicators within the strategy's `__init__` method, passing the appropriate data feed and configuration parameters:

```python
import backtrader as bt
# Assume RateOfChange class is defined or imported

class MomentumStrategy(bt.Strategy):
    """
    Strategy utilizing custom ROC indicator for momentum-based trading
    """
    
    params = (
        ('roc_period', 12),           # ROC calculation period
        ('momentum_threshold', 5.0),   # Entry signal threshold
        ('exit_threshold', -2.0),      # Exit signal threshold
        ('position_size', 0.95),       # Position sizing (95% of available cash)
    )
    
    def __init__(self):
        # Initialize order tracking
        self.order = None
        
        # Instantiate custom ROC indicator
        self.roc_indicator = RateOfChange(
            self.datas[0],                    # Apply to primary data feed
            period=self.p.roc_period         # Pass period parameter
        )
        
        # Optional: Create additional indicators for confirmation
        self.sma_short = bt.indicators.SimpleMovingAverage(
            self.data.close, period=20
        )
        self.sma_long = bt.indicators.SimpleMovingAverage(
            self.data.close, period=50
        )
        
        # Initialize logging variables
        self.last_roc_value = None
```

### 2. Accessing Indicator Values in `next()`

Within the strategy's `next()` method, access your custom indicator's output lines using the names defined in its `lines` tuple:

```python
def next(self):
    """Execute strategy logic for each new data bar"""
    
    # Skip if pending orders exist
    if self.order:
        return
    
    # Access current ROC value
    current_roc = self.roc_indicator.roc[0]
    
    # Alternative access method for single-line indicators
    # current_roc = self.roc_indicator[0]  # Often works for single-output indicators
    
    # Handle potential NaN values during initial period
    if current_roc != current_roc:  # NaN check (NaN != NaN is True)
        return  # Wait for valid ROC calculation
    
    # Store current value for logging
    self.last_roc_value = current_roc
    
    # Strategy logic implementation
    self._execute_trading_logic(current_roc)

def _execute_trading_logic(self, roc_value):
    """Encapsulated trading logic based on ROC signals"""
    
    if not self.position:  # No current position
        self._check_entry_signals(roc_value)
    else:  # Currently in position
        self._check_exit_signals(roc_value)

def _check_entry_signals(self, roc_value):
    """Check for entry conditions"""
    
    # Entry condition: Strong positive momentum
    if roc_value > self.p.momentum_threshold:
        # Additional confirmation: Price above short-term average
        if self.data.close[0] > self.sma_short[0]:
            self.log(f'BUY SIGNAL: ROC={roc_value:.2f} above threshold {self.p.momentum_threshold}')
            
            # Calculate position size
            size = int((self.broker.getvalue() * self.p.position_size) / self.data.close[0])
            self.order = self.buy(size=size)

def _check_exit_signals(self, roc_value):
    """Check for exit conditions"""
    
    # Exit condition 1: Momentum reversal
    if roc_value < self.p.exit_threshold:
        self.log(f'EXIT SIGNAL: ROC={roc_value:.2f} below exit threshold {self.p.exit_threshold}')
        self.order = self.close()
    
    # Exit condition 2: Trend reversal (price below long-term average)
    elif self.data.close[0] < self.sma_long[0]:
        self.log(f'TREND EXIT: Price below long-term SMA')
        self.order = self.close()
```

### 3. Enhanced Strategy with Multiple Custom Indicators

Here's an example demonstrating the use of multiple custom indicators:

```python
class AdvancedMomentumStrategy(bt.Strategy):
    """Advanced strategy combining multiple custom indicators"""
    
    params = (
        ('roc_short_period', 10),
        ('roc_long_period', 20),
        ('momentum_divergence_threshold', 3.0),
    )
    
    def __init__(self):
        self.order = None
        
        # Multiple ROC indicators with different periods
        self.roc_short = RateOfChange(self.data, period=self.p.roc_short_period)
        self.roc_long = RateOfChange(self.data, period=self.p.roc_long_period)
        
        # Create momentum divergence signal
        self.momentum_divergence = self.roc_short.roc - self.roc_long.roc
        
        # Volume-based confirmation (assuming volume data available)
        self.volume_sma = bt.indicators.SimpleMovingAverage(
            self.data.volume, period=20
        )
    
    def next(self):
        if self.order:
            return
        
        # Access multiple indicator values
        short_roc = self.roc_short.roc[0]
        long_roc = self.roc_long.roc[0]
        divergence = self.momentum_divergence[0]
        current_volume = self.data.volume[0]
        avg_volume = self.volume_sma[0]
        
        # Skip if any values are NaN
        if any(val != val for val in [short_roc, long_roc, divergence]):
            return
        
        # Strategy logic using multiple indicators
        if not self.position:
            # Entry: Positive momentum divergence with volume confirmation
            if (divergence > self.p.momentum_divergence_threshold and 
                short_roc > 0 and 
                current_volume > avg_volume * 1.2):
                
                self.log(f'MULTI-SIGNAL BUY: Divergence={divergence:.2f}, '
                        f'ROC_short={short_roc:.2f}, Volume={current_volume/avg_volume:.2f}x')
                self.order = self.buy()
        
        else:
            # Exit: Momentum divergence reversal
            if divergence < -self.p.momentum_divergence_threshold:
                self.log(f'DIVERGENCE EXIT: Negative divergence={divergence:.2f}')
                self.order = self.close()
```

## Complete Integration Example

### Full Script with Data Loading and Execution

```python
import backtrader as bt
import yfinance as yf
import pandas as pd

# Custom ROC Indicator (previously defined)
class RateOfChange(bt.Indicator):
    lines = ('roc',)
    params = (('period', 10),)
    plotinfo = dict(subplot=True, plotname='Rate of Change (%)', plothlines=[0.0])
    
    def __init__(self):
        self.lines.roc = ((self.data.close - self.data.close(-self.p.period)) 
                         / self.data.close(-self.p.period)) * 100.0
        super(RateOfChange, self).__init__()

# Strategy using custom indicator
class ROCMomentumStrategy(bt.Strategy):
    params = (
        ('roc_period', 14),
        ('entry_threshold', 3.0),
        ('exit_threshold', -1.0),
    )
    
    def __init__(self):
        self.order = None
        self.roc = RateOfChange(self.data, period=self.p.roc_period)
    
    def log(self, txt, dt=None):
        dt = dt or self.datas[0].datetime.date(0)
        print(f'{dt.isoformat()}: {txt}')
    
    def notify_order(self, order):
        if order.status in [order.Completed]:
            if order.isbuy():
                self.log(f'BUY EXECUTED: Price={order.executed.price:.2f}')
            else:
                self.log(f'SELL EXECUTED: Price={order.executed.price:.2f}')
        self.order = None
    
    def next(self):
        if self.order:
            return
        
        roc_value = self.roc.roc[0]
        
        if roc_value != roc_value:  # NaN check
            return
        
        if not self.position:
            if roc_value > self.p.entry_threshold:
                self.log(f'BUY SIGNAL: ROC={roc_value:.2f}')
                self.order = self.buy()
        else:
            if roc_value < self.p.exit_threshold:
                self.log(f'SELL SIGNAL: ROC={roc_value:.2f}')
                self.order = self.close()

# Data preparation and execution
def run_roc_strategy():
    # Download data
    ticker = 'AAPL'
    data_df = yf.download(ticker, start='2020-01-01', end='2023-12-31')
    data_df.columns = data_df.columns.droplevel(1)
    
    # Create backtrader data feed
    data = bt.feeds.PandasData(dataname=data_df)
    
    # Initialize Cerebro
    cerebro = bt.Cerebro()
    cerebro.addstrategy(ROCMomentumStrategy, roc_period=14, entry_threshold=4.0)
    cerebro.adddata(data)
    cerebro.broker.setcash(10000.0)
    cerebro.broker.setcommission(commission=0.001)
    
    # Execute backtest
    print(f'Starting Portfolio Value: {cerebro.broker.getvalue():.2f}')
    results = cerebro.run()
    print(f'Final Portfolio Value: {cerebro.broker.getvalue():.2f}')
    
    # Optional: Plot results
    try:
        cerebro.plot(style='candlestick', volume=False)
    except Exception as e:
        print(f"Plotting not available: {e}")

if __name__ == '__main__':
    run_roc_strategy()
```

## Key Integration Points

### Cerebro Setup
No special configuration is required in Cerebro when using custom indicators. The setup process remains identical to strategies using only built-in indicators.

### Plotting Integration
Backtrader's plotting system automatically recognizes custom indicators based on their `plotinfo` configuration:
- **Subplot indicators** (like ROC) appear in dedicated panels below the price chart
- **Overlay indicators** attempt to display on the main price chart
- Line colors, styles, and panel organization are handled automatically

### Performance Considerations
- Custom indicators using Line Arithmetic perform comparably to built-in indicators
- Multiple custom indicators can be used simultaneously without significant performance degradation
- Memory usage scales linearly with the number of indicators and data history length

This integration approach ensures that custom indicators become first-class citizens within your trading strategies, providing the same level of functionality and performance as backtrader's native analytical tools.


---

[← Back to Outline](../outline.md)