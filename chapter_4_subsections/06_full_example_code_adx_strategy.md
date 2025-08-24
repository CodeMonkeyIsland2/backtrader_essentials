[← Back to Outline](../outline.md)

---

# Full Example Code: AdxStrategyExample

This section presents a complete, production-ready ADX strategy implementation that integrates all the concepts discussed in previous sections. The strategy combines trend strength filtering with directional signals and includes comprehensive order management.

## Complete Strategy Implementation

```python
import backtrader as bt

class AdxStrategyExample(bt.Strategy):
    """
    Complete ADX System Implementation
    
    Features:
    - ADX trend strength filtering
    - Directional indicator crossover signals
    - Comprehensive order management
    - Position tracking and risk control
    """
    
    params = (
        ('adx_period', 14),         # ADX calculation period
        ('adx_threshold', 25.0),    # Minimum ADX for trend confirmation
        ('position_size', 1.0),     # Position sizing factor
    )
    
    def log(self, txt, dt=None):
        """Enhanced logging with timestamp"""
        dt = dt or self.datas[0].datetime.date(0)
        print(f'{dt.isoformat()} - {txt}')
    
    def __init__(self):
        """Strategy initialization and indicator setup"""
        
        self.log(f'--- Initializing {self.__class__.__name__} ---')
        
        # Data references
        self.dataclose = self.datas[0].close
        self.datahigh = self.datas[0].high
        self.datalow = self.datas[0].low
        
        # Initialize ADX system indicators
        self.adx = bt.indicators.ADX(
            self.datas[0], 
            period=self.params.adx_period
        )
        
        self.plus_di = bt.indicators.PlusDI(
            self.datas[0], 
            period=self.params.adx_period
        )
        
        self.minus_di = bt.indicators.MinusDI(
            self.datas[0], 
            period=self.params.adx_period
        )
        
        # Strategy state tracking
        self.order = None                    # Track pending orders
        self.position_entered = False        # Track position status
        self.entry_price = 0.0              # Record entry price
        
        self.log(f'ADX System configured: Period={self.params.adx_period}, '
                f'Threshold={self.params.adx_threshold}')
        self.log(f'--- Strategy Initialization Complete ---')
    
    def notify_order(self, order):
        """Handle order status notifications"""
        
        if order.status in [order.Submitted, order.Accepted]:
            # Order submitted to broker - no action needed
            return
        
        if order.status == order.Completed:
            if order.isbuy():
                self.log(f'BUY EXECUTED - Ref: {order.ref}, '
                        f'Price: {order.executed.price:.2f}, '
                        f'Value: {order.executed.value:.2f}, '
                        f'Commission: {order.executed.comm:.2f}')
                self.position_entered = True
                self.entry_price = order.executed.price
                
            elif order.issell():
                self.log(f'SELL EXECUTED - Ref: {order.ref}, '
                        f'Price: {order.executed.price:.2f}, '
                        f'Value: {order.executed.value:.2f}, '
                        f'Commission: {order.executed.comm:.2f}')
                self.position_entered = False
                self.entry_price = 0.0
        
        elif order.status in [order.Canceled, order.Margin, order.Rejected]:
            self.log(f'ORDER {order.getstatusname()} - Ref: {order.ref}')
        
        # Reset order tracker - allows new orders
        self.order = None
    
    def notify_trade(self, trade):
        """Handle trade completion notifications"""
        
        if not trade.isclosed:
            return
        
        # Calculate trade performance
        gross_pnl = trade.pnl
        net_pnl = trade.pnlcomm
        trade_return = (net_pnl / trade.value) * 100 if trade.value else 0
        
        self.log(f'TRADE CLOSED - Ref: {trade.ref}, '
                f'Gross P&L: {gross_pnl:.2f}, '
                f'Net P&L: {net_pnl:.2f}, '
                f'Return: {trade_return:.2f}%')
    
    def next(self):
        """Main strategy logic executed on each bar"""
        
        # Skip if order is pending
        if self.order:
            return
        
        # Gather current indicator values
        current_adx = self.adx[0]
        current_plus_di = self.plus_di[0]
        current_minus_di = self.minus_di[0]
        
        # Gather previous values for crossover detection
        previous_plus_di = self.plus_di[-1]
        previous_minus_di = self.minus_di[-1]
        
        # Current price for reference
        current_price = self.dataclose[0]
        
        # === TREND STRENGTH ANALYSIS ===
        
        if current_adx > self.params.adx_threshold:
            # Strong trend environment detected
            
            # Check for bullish crossover signal
            bullish_crossover = (
                current_plus_di > current_minus_di and 
                previous_plus_di <= previous_minus_di
            )
            
            if bullish_crossover:
                self.log(f'BULLISH SIGNAL DETECTED: +DI ({current_plus_di:.2f}) '
                        f'crossed above -DI ({current_minus_di:.2f}), '
                        f'ADX Strength: {current_adx:.2f}')
                
                # Enter long position if not already positioned
                if not self.position:
                    self.log(f'ENTERING LONG POSITION at {current_price:.2f}')
                    self.order = self.buy(size=self.params.position_size)
            
            # Check for bearish crossover signal
            bearish_crossover = (
                current_minus_di > current_plus_di and 
                previous_minus_di <= previous_plus_di
            )
            
            if bearish_crossover:
                self.log(f'BEARISH SIGNAL DETECTED: -DI ({current_minus_di:.2f}) '
                        f'crossed above +DI ({current_plus_di:.2f}), '
                        f'ADX Strength: {current_adx:.2f}')
                
                # Close long position if currently held
                if self.position.size > 0:
                    self.log(f'CLOSING LONG POSITION at {current_price:.2f}')
                    self.order = self.close()
                
                # Optional: Could implement short selling here
                # elif not self.position:
                #     self.log(f'ENTERING SHORT POSITION at {current_price:.2f}')
                #     self.order = self.sell(size=self.params.position_size)
        
        else:
            # === WEAK TREND ENVIRONMENT ===
            
            # Close any existing positions when trend weakens
            if self.position:
                self.log(f'WEAK TREND DETECTED: ADX ({current_adx:.2f}) '
                        f'below threshold ({self.params.adx_threshold})')
                self.log(f'CLOSING POSITION due to weak trend at {current_price:.2f}')
                self.order = self.close()
    
    def stop(self):
        """Called when strategy execution ends"""
        self.log(f'--- Strategy {self.__class__.__name__} Execution Complete ---')
        self.log(f'Final ADX: {self.adx[0]:.2f}, '
                f'+DI: {self.plus_di[0]:.2f}, '
                f'-DI: {self.minus_di[0]:.2f}')
```

## Strategy Features

### 1. Comprehensive Signal Logic
- **Trend Strength Filter**: Only trades when ADX exceeds threshold
- **Directional Signals**: Uses +DI/-DI crossovers for entry/exit timing
- **Weak Trend Handling**: Automatically exits positions when trends weaken

### 2. Enhanced Order Management
- **Order Tracking**: Prevents duplicate orders while one is pending
- **Position Monitoring**: Tracks entry prices and position status
- **Status Notifications**: Detailed logging of all order events

### 3. Risk Management Features
- **Position Sizing**: Configurable position size parameter
- **Automatic Exits**: Closes positions when trend strength deteriorates
- **Trade Performance**: Tracks and reports individual trade results

### 4. Flexibility and Customization
- **Parameterized Settings**: Easy optimization of ADX period and threshold
- **Extensible Design**: Framework supports additional features
- **Multiple Timeframes**: Compatible with any data frequency

## Usage Example

```python
# Strategy implementation ready for Cerebro integration
cerebro = bt.Cerebro()

# Add strategy with custom parameters
cerebro.addstrategy(
    AdxStrategyExample,
    adx_period=14,
    adx_threshold=25.0,
    position_size=1.0
)

# Continue with data loading and execution...
```

This complete implementation serves as a foundation for ADX-based trading strategies. The next section will provide a detailed walkthrough of the code structure and logic flow.



---

[← Back to Outline](../outline.md)