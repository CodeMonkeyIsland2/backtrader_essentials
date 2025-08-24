[← Back to Outline](../outline.md)

---

# Adding a Minimal Strategy and Running the Test

## Understanding Backtrader Strategies

A Backtrader strategy is a Python class that inherits from `bt.Strategy`. It defines the trading logic, order management, and decision-making processes for your backtesting simulation.

## Strategy Structure Overview

Every strategy contains these key components:
- **Parameters**: Configurable strategy variables
- **Initialization**: Setup indicators and references
- **Execution Logic**: The `next()` method called for each data bar
- **Order Management**: Handling order notifications and trade events

## Creating Your First Strategy

### Basic Strategy Template

```python
class SimpleTestStrategy(bt.Strategy):
    """
    A minimal strategy for demonstration purposes
    Implements basic buy-and-hold logic with exit conditions
    """
    
    # Strategy parameters (configurable)
    params = (
        ('hold_period', 5),    # Days to hold position
        ('verbose', True),     # Enable detailed logging
    )
    
    def log(self, message, timestamp=None):
        """Custom logging function for strategy events"""
        timestamp = timestamp or self.datas[0].datetime.date(0)
        if self.params.verbose:
            print(f'{timestamp.isoformat()} - {message}')
    
    def __init__(self):
        """Initialize strategy variables and references"""
        # Store reference to closing prices
        self.close_price = self.datas[0].close
        
        # Order tracking variables
        self.current_order = None
        self.buy_price = None
        self.buy_commission = None
        self.entry_bar = None
        
        self.log('Strategy initialized successfully')
    
    def notify_order(self, order):
        """Handle order status notifications"""
        if order.status in [order.Submitted, order.Accepted]:
            # Order submitted to broker - no action needed
            return
        
        if order.status == order.Completed:
            if order.isbuy():
                self.log(f'BUY EXECUTED - Price: ${order.executed.price:.2f}, '
                        f'Value: ${order.executed.value:.2f}, '
                        f'Commission: ${order.executed.comm:.2f}')
                self.buy_price = order.executed.price
                self.buy_commission = order.executed.comm
                self.entry_bar = len(self)
                
            elif order.issell():
                self.log(f'SELL EXECUTED - Price: ${order.executed.price:.2f}, '
                        f'Value: ${order.executed.value:.2f}, '
                        f'Commission: ${order.executed.comm:.2f}')
        
        elif order.status in [order.Canceled, order.Margin, order.Rejected]:
            self.log(f'Order {order.getstatusname()}')
        
        # Reset order tracking
        self.current_order = None
    
    def notify_trade(self, trade):
        """Handle completed trade notifications"""
        if trade.isclosed:
            self.log(f'TRADE CLOSED - Gross P&L: ${trade.pnl:.2f}, '
                    f'Net P&L: ${trade.pnlcomm:.2f}')
    
    def next(self):
        """Main strategy logic - called for each data bar"""
        # Skip if we have a pending order
        if self.current_order:
            return
        
        # Entry logic: Buy on first bar if not in position
        if not self.position:
            if len(self) == 1:  # First bar only
                self.log(f'BUY SIGNAL - Price: ${self.close_price[0]:.2f}')
                self.current_order = self.buy()
        
        # Exit logic: Sell after holding for specified period
        else:
            if self.entry_bar and len(self) >= (self.entry_bar + self.params.hold_period):
                self.log(f'SELL SIGNAL - Price: ${self.close_price[0]:.2f}')
                self.current_order = self.sell()
```

## Integrating Strategy with Cerebro

```python
# Add strategy to the backtesting engine
cerebro.addstrategy(SimpleTestStrategy)
print("Strategy successfully added to Cerebro")

# Optional: Add strategy with custom parameters
# cerebro.addstrategy(SimpleTestStrategy, hold_period=10, verbose=False)
```

## Running the Backtest

```python
# Execute the complete backtesting simulation
print("\n" + "="*50)
print("STARTING BACKTEST SIMULATION")
print("="*50)

# Display initial portfolio state
print(f'Starting Portfolio Value: ${cerebro.broker.getvalue():,.2f}')

# Run the backtest
try:
    strategy_results = cerebro.run()
    first_strategy = strategy_results[0]
    
    # Display final results
    final_value = cerebro.broker.getvalue()
    initial_value = starting_capital
    total_return = ((final_value - initial_value) / initial_value) * 100
    
    print("="*50)
    print("BACKTEST RESULTS")
    print("="*50)
    print(f'Final Portfolio Value: ${final_value:,.2f}')
    print(f'Total Return: {total_return:.2f}%')
    print(f'Absolute Profit/Loss: ${final_value - initial_value:,.2f}')
    
except Exception as e:
    print(f"Backtest execution failed: {e}")
```

## Expected Output Example

```
==================================================
STARTING BACKTEST SIMULATION
==================================================
Starting Portfolio Value: $10,000.00
2020-01-02 - Strategy initialized successfully
2020-01-02 - BUY SIGNAL - Price: $72.72
2020-01-03 - BUY EXECUTED - Price: $71.94, Value: $71.94, Commission: $0.07
2020-01-09 - SELL SIGNAL - Price: $74.96
2020-01-10 - SELL EXECUTED - Price: $75.20, Value: $71.94, Commission: $0.08
2020-01-10 - TRADE CLOSED - Gross P&L: $3.26, Net P&L: $3.11
==================================================
BACKTEST RESULTS
==================================================
Final Portfolio Value: $10,003.11
Total Return: 0.03%
Absolute Profit/Loss: $3.11
```

## Strategy Development Tips

1. **Start Simple**: Begin with basic buy/sell logic before adding complexity
2. **Parameter Testing**: Use strategy parameters for easy optimization
3. **Logging**: Implement comprehensive logging for debugging
4. **Order Management**: Always track order states to prevent conflicts
5. **Position Awareness**: Check position status before placing new orders

## Common Strategy Patterns

### Buy and Hold
```python
def next(self):
    if not self.position and len(self) == 1:
        self.buy()
```

### Mean Reversion
```python
def next(self):
    if self.close_price[0] < self.sma[0] * 0.95:  # 5% below moving average
        self.buy()
    elif self.close_price[0] > self.sma[0] * 1.05:  # 5% above moving average
        self.sell()
```

### Momentum Following
```python
def next(self):
    if self.close_price[0] > self.close_price[-1] * 1.02:  # 2% daily gain
        self.buy()
```

Your strategy is now integrated and ready to execute. The next step is visualizing the results through Backtrader's plotting capabilities.


---

[← Back to Outline](../outline.md)