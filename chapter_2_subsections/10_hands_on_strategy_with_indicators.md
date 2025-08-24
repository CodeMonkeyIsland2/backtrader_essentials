[← Back to Outline](../outline.md)

---

# Hands-On: The MyStrategyWithIndicators Code

## Comprehensive Strategy Implementation

Now we'll integrate all the concepts covered—SMA implementation, RSI configuration, and indicator chaining—into a complete, practical strategy class that demonstrates proper structure and indicator management.

## Complete Strategy Code

```python
# Complete strategy implementation demonstrating multiple indicators

import backtrader as bt

class MyStrategyWithIndicators(bt.Strategy):
    """
    Comprehensive strategy demonstrating SMA and RSI implementation,
    including optional indicator chaining (SMA applied to RSI).
    """
    
    # Strategy parameter configuration
    params = (
        ('sma_period', 20),         # Simple Moving Average lookback period
        ('rsi_period', 14),         # Relative Strength Index calculation period  
        ('rsi_sma_period', 10),     # Smoothing period for RSI moving average
        ('log_interval', 20),       # Logging frequency control (every N bars)
    )
    
    def log(self, message, dt=None):
        """Standardized logging utility for strategy messages"""
        # Use primary data feed date or provided date
        dt = dt or self.datas[0].datetime.date(0)
        print(f'{dt.isoformat()} - {message}')
    
    def __init__(self):
        """Strategy initialization - executed once before backtesting begins"""
        
        self.log(f'--- Strategy {self.__class__.__name__} Initialization Starting ---')
        
        # Store reference to primary data feed's closing prices
        self.dataclose = self.datas[0].close
        self.log('Primary data feed reference established.')
        
        # 1. Initialize Simple Moving Average indicator
        self.sma = bt.indicators.SimpleMovingAverage(
            self.datas[0],                    # Apply to primary data feed
            period=self.params.sma_period
        )
        self.log(f'SMA indicator configured: {self.params.sma_period}-period lookback')
        
        # 2. Initialize Relative Strength Index indicator  
        self.rsi = bt.indicators.RelativeStrengthIndex(
            period=self.params.rsi_period     # Automatically uses datas[0].close
        )
        self.log(f'RSI indicator configured: {self.params.rsi_period}-period calculation')
        
        # 3. Initialize chained indicator: SMA of RSI (optional demonstration)
        # Initialize as None first, then conditionally create
        self.rsi_sma = None
        
        # OPTIONAL: Uncomment the following lines to activate RSI smoothing
        # self.rsi_sma = bt.indicators.SimpleMovingAverage(
        #     self.rsi,                         # Apply SMA to RSI output line
        #     period=self.params.rsi_sma_period
        # )
        # self.log(f'RSI smoothing SMA configured: {self.params.rsi_sma_period}-period')
        
        if self.rsi_sma is None:
            self.log('RSI smoothing SMA is currently INACTIVE (commented out).')
        
        self.log(f'--- Strategy {self.__class__.__name__} Initialization Complete ---')
    
    def next(self):
        """
        Executed for each data bar after minimum period requirements are satisfied.
        Contains indicator access examples and placeholder for trading logic.
        """
        
        # Implement periodic logging based on log_interval parameter
        if len(self) % self.params.log_interval == 0:
            
            # Extract current indicator values
            current_close = self.dataclose[0]
            current_sma = self.sma[0]
            current_rsi = self.rsi[0]
            
            # Build comprehensive log message
            log_message = (
                f'Bar #{len(self)}: '
                f'Close: {current_close:.2f}, '
                f'SMA({self.params.sma_period}): {current_sma:.2f}, '
                f'RSI({self.params.rsi_period}): {current_rsi:.2f}'
            )
            
            # Include RSI smoothing data if active
            if self.rsi_sma:
                current_rsi_sma = self.rsi_sma[0]
                log_message += f', RSI_SMA({self.params.rsi_sma_period}): {current_rsi_sma:.2f}'
            
            self.log(log_message)
        
        # --- PLACEHOLDER FOR TRADING LOGIC ---
        # This demonstration strategy focuses on indicator setup and access.
        # Actual buy/sell logic will be implemented in subsequent chapters.
        
        # Example signal detection logic (commented for demonstration):
        # 
        # # RSI oversold condition detection
        # if self.rsi[0] < 30 and self.rsi[-1] >= 30:
        #     self.log("RSI oversold signal detected - Potential buy opportunity")
        # 
        # # Price-SMA bullish crossover detection  
        # if self.dataclose[0] > self.sma[0] and self.dataclose[-1] <= self.sma[-1]:
        #     self.log("Bullish price-SMA crossover - Potential buy signal")
        
        pass  # End of next() method
```

## Code Architecture Analysis

### Parameter Management Strategy

**Flexible Configuration**: All key indicator parameters are defined in the `params` tuple, enabling easy modification during backtesting without code changes.

**Logging Control**: The `log_interval` parameter prevents excessive output by controlling logging frequency, maintaining clean console output during extended backtests.

### Initialization Sequence

**Reference Storage**: `self.dataclose` provides convenient access to closing prices, though it's optional for indicator instantiation.

**Progressive Indicator Setup**: Indicators are created in logical order, with more complex chained indicators built upon simpler ones.

**Optional Features**: The RSI smoothing functionality is included but commented out by default, demonstrating how to implement optional indicator features.

### Execution Logic Structure  

**Minimum Period Compliance**: The `next()` method only executes after all indicators have sufficient data, eliminating data availability concerns.

**Systematic Value Access**: Current indicator values are extracted using consistent indexing patterns (`[0]` for current bar).

**Scalable Logging**: The periodic logging system provides valuable debugging information without overwhelming output volume.

**Trading Logic Placeholder**: The commented signal detection examples show where actual trading decisions would be implemented in production strategies.

This comprehensive example serves as a foundation for developing more sophisticated trading strategies while demonstrating best practices for indicator integration and strategy organization.


---

[← Back to Outline](../outline.md)