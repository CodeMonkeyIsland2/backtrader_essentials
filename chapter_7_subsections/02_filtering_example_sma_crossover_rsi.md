[← Back to Outline](../outline.md)

---

# Filtering Example: SMA Crossover + RSI

Let's demonstrate the filtering concept by combining a fundamental trend-following signal (SMA Crossover) with a momentum filter (RSI). This approach illustrates how different indicator types can work together to improve signal quality.

## Strategy Concept

The core idea involves using the crossover of a faster SMA above/below a slower SMA to identify potential trend changes, but only acting on these signals when market momentum, measured by RSI, hasn't reached extreme levels that might indicate an imminent reversal.

## Specific Implementation Logic

**Entry Signal**: Fast SMA crosses above Slow SMA (bullish signal)

**Filter Condition**: We only execute this buy signal when RSI remains below overbought territory (e.g., RSI < 70). The reasoning is to avoid entering positions when short-term momentum reaches potentially unsustainable peaks, even when moving averages generate buy signals.

**Exit Signal**: For this initial example, we'll use the opposite signal - exit long positions when the Fast SMA crosses below the Slow SMA (bearish crossover). *(We'll explore improved exit strategies later in this chapter)*

## Code Implementation

```python
import backtrader as bt

class SmaCrossFiltered(bt.Strategy):
    """
    Enters long positions on bullish SMA crossovers only when RSI is not overbought.
    Exits long positions on bearish SMA crossovers.
    """
    
    params = (
        ('fast_ma', 10),           # Fast SMA period
        ('slow_ma', 30),           # Slow SMA period  
        ('rsi_period', 14),        # RSI calculation period
        ('rsi_overbought', 70.0),  # RSI overbought threshold for filtering
    )
    
    def log(self, txt, dt=None):
        dt = dt or self.datas[0].datetime.date(0)
        print(f'{dt.isoformat()} - {txt}')
    
    def __init__(self):
        self.log(f'--- Strategy {self.__class__.__name__} Initializing ---')
        
        self.dataclose = self.datas[0].close
        self.order = None  # Track pending orders
        
        # Initialize indicators
        self.sma_fast = bt.indicators.SimpleMovingAverage(
            self.datas[0], period=self.params.fast_ma)
        self.sma_slow = bt.indicators.SimpleMovingAverage(
            self.datas[0], period=self.params.slow_ma)
        self.rsi = bt.indicators.RelativeStrengthIndex(
            period=self.params.rsi_period)
        
        self.log(f'Indicators initialized: SMA({self.params.fast_ma}, {self.params.slow_ma}), '
                f'RSI({self.params.rsi_period}, OB={self.params.rsi_overbought})')
        self.log(f'--- Strategy {self.__class__.__name__} Initialized ---')
    
    def notify_order(self, order):
        # Standard order notification handling
        if order.status in [order.Submitted, order.Accepted]: 
            return
        
        if order.status in [order.Completed]:
            if order.isbuy(): 
                self.log(f'BUY EXECUTED, Price:{order.executed.price:.2f}, '
                        f'Cost:{order.executed.value:.2f}, Comm:{order.executed.comm:.2f}')
            elif order.issell(): 
                self.log(f'SELL EXECUTED, Price:{order.executed.price:.2f}, '
                        f'Cost:{order.executed.value:.2f}, Comm:{order.executed.comm:.2f}')
        elif order.status in [order.Canceled, order.Margin, order.Rejected]: 
            self.log(f'Order {order.getstatusname()}')
        
        self.order = None
    
    def notify_trade(self, trade):
        # Standard trade notification handling
        if trade.isclosed: 
            self.log(f'TRADE CLOSED: PNL Gross {trade.pnl:.2f}, PNL Net {trade.pnlcomm:.2f}')
    
    def next(self):
        """Called on each bar after minimum periods are satisfied"""
        
        # 1. Skip if order is pending
        if self.order:
            return
        
        # 2. Determine current position status
        if not self.position:
            # --- Entry Logic ---
            
            # Condition 1: Bullish SMA crossover on current bar
            # Verify fast SMA is above slow SMA now AND was below/equal previously
            is_bullish_cross = (self.sma_fast[0] > self.sma_slow[0] and
                              self.sma_fast[-1] <= self.sma_slow[-1])
            
            # Condition 2: RSI below overbought threshold
            is_rsi_acceptable = (self.rsi[0] < self.params.rsi_overbought)
            
            # Combined entry signal
            if is_bullish_cross and is_rsi_acceptable:
                self.log(f'BUY SIGNAL: SMA Cross & RSI Filter. Price: {self.dataclose[0]:.2f}, '
                        f'FastSMA: {self.sma_fast[0]:.2f}, SlowSMA: {self.sma_slow[0]:.2f}, '
                        f'RSI: {self.rsi[0]:.2f} (<{self.params.rsi_overbought})')
                
                # Execute buy order
                self.order = self.buy()
            
        else:  # Currently in position (long)
            # --- Exit Logic ---
            
            # Condition: Bearish SMA crossover on current bar
            # Verify fast SMA is below slow SMA now AND was above/equal previously
            is_bearish_cross = (self.sma_fast[0] < self.sma_slow[0] and
                              self.sma_fast[-1] >= self.sma_slow[-1])
            
            if is_bearish_cross:
                self.log(f'SELL/CLOSE SIGNAL: Bearish SMA Cross. Price: {self.dataclose[0]:.2f}, '
                        f'FastSMA: {self.sma_fast[0]:.2f}, SlowSMA: {self.sma_slow[0]:.2f}')
                
                # Execute close order
                self.order = self.close()
```

This implementation demonstrates how to effectively combine trend-following signals with momentum filters, creating a more selective entry process that aims to improve signal quality by avoiding entries during unfavorable momentum conditions.


---

[← Back to Outline](../outline.md)