[← Back to Outline](../outline.md)

---

# Implementation: Enhanced SmaCrossFiltered Strategy

Here's the complete implementation of our advanced multi-indicator strategy with bracket order risk management:

```python
import backtrader as bt
import datetime

class SmaCrossFilteredAdvanced(bt.Strategy):
    """
    Enhanced strategy that enters long positions on bullish SMA crossovers 
    when RSI is not overbought, ADX is increasing, and ATR is increasing.
    Exits on bearish SMA crossover, stop-loss, or take-profit.
    """
    
    params = (
        ('fast_ma', 10),           # Fast SMA period
        ('slow_ma', 30),           # Slow SMA period
        ('rsi_period', 14),        # RSI calculation period
        ('rsi_overbought', 70.0),  # RSI overbought threshold
        ('adx_period', 14),        # ADX calculation period
        ('atr_period', 14),        # ATR calculation period
        ('stop_loss_pct', 5.0),    # Stop-loss percentage (e.g., 5.0 for 5%)
        ('take_profit_pct', 10.0), # Take-profit percentage (e.g., 10.0 for 10%)
        ('printlog', True),        # Enable/disable logging
    )
    
    def log(self, txt, dt=None, doprint=False):
        """Logging function for strategy events"""
        if self.params.printlog or doprint:
            dt = dt or self.datas[0].datetime.date(0)
            print(f'{dt.isoformat()} - {txt}')
    
    def __init__(self):
        self.log(f'--- Strategy {self.__class__.__name__} Initializing ---')
        
        # Data references
        self.dataclose = self.datas[0].close
        
        # Order tracking variables
        self.order = None           # Track pending entry/exit orders
        self.buyprice = None        # Track entry price
        self.buycomm = None         # Track entry commission
        self.bracket_orders = None  # Track bracket order list [buy, stop, limit]
        
        # Initialize all indicators
        self.sma_fast = bt.indicators.SimpleMovingAverage(
            self.datas[0], period=self.params.fast_ma)
        self.sma_slow = bt.indicators.SimpleMovingAverage(
            self.datas[0], period=self.params.slow_ma)
        self.rsi = bt.indicators.RelativeStrengthIndex(
            period=self.params.rsi_period)
        self.adx = bt.indicators.AverageDirectionalMovementIndex(
            self.datas[0], period=self.params.adx_period)
        self.atr = bt.indicators.AverageTrueRange(
            self.datas[0], period=self.params.atr_period)
        
        # Log initialization details
        self.log(f'Indicators created: SMA({self.params.fast_ma}, {self.params.slow_ma}), '
                f'RSI({self.params.rsi_period}, OB={self.params.rsi_overbought}), '
                f'ADX({self.params.adx_period}), ATR({self.params.atr_period})')
        self.log(f'Risk Management: Stop Loss {self.params.stop_loss_pct}%, '
                f'Take Profit {self.params.take_profit_pct}%')
        self.log(f'--- Strategy {self.__class__.__name__} Initialized ---')
    
    def notify_order(self, order):
        """Enhanced order notification to handle bracket orders"""
        
        if order.status in [order.Submitted, order.Accepted]:
            # Order is pending, nothing to do
            return
        
        # Handle completed orders
        if order.status in [order.Completed]:
            if order.isbuy():
                self.log(f'BUY EXECUTED, Price:{order.executed.price:.2f}, '
                        f'Cost:{order.executed.value:.2f}, Comm:{order.executed.comm:.2f}')
                self.buyprice = order.executed.price
                self.buycomm = order.executed.comm
                
                # Check if this is the primary buy order of our bracket
                if self.bracket_orders and order == self.bracket_orders[0]:
                    stop_price = self.bracket_orders[1].created.price
                    profit_price = self.bracket_orders[2].created.price
                    self.log(f'---> Bracket BUY Executed - '
                            f'StopLoss: {stop_price:.2f}, TakeProfit: {profit_price:.2f}')
                    self.order = None  # Clear main order tracker
                    
            elif order.issell():
                self.log(f'SELL EXECUTED, Price:{order.executed.price:.2f}, '
                        f'Cost:{order.executed.value:.2f}, Comm:{order.executed.comm:.2f}')
                
                # Identify which type of sell order executed
                if self.bracket_orders:
                    if order == self.bracket_orders[1]:  # Stop Loss triggered
                        self.log(f'---> STOP LOSS EXECUTED')
                        self.bracket_orders = None
                    elif order == self.bracket_orders[2]:  # Take Profit triggered
                        self.log(f'---> TAKE PROFIT EXECUTED')
                        self.bracket_orders = None
                else:
                    # This was likely the SMA crossover exit
                    self.log(f'---> SMA Crossover SELL EXECUTED')
                    # Cancel any remaining bracket orders
                    if self.bracket_orders:
                        self.cancel(self.bracket_orders[1])
                        self.cancel(self.bracket_orders[2])
                        self.bracket_orders = None
                
                self.order = None  # Clear order tracker
                
        elif order.status in [order.Canceled, order.Margin, order.Rejected]:
            self.log(f'Order {order.getstatusname()}')
            
            # Handle failed bracket buy orders
            if self.bracket_orders and order == self.bracket_orders[0]:
                self.log(f'---> Bracket BUY Order {order.getstatusname()}, clearing bracket.')
                self.bracket_orders = None
            
            self.order = None
        
        # Reset tracking variables when flat
        if not self.position:
            self.bracket_orders = None
            self.buyprice = None
            self.buycomm = None
    
    def notify_trade(self, trade):
        """Trade notification handling"""
        if not trade.isclosed:
            return
        
        self.log(f'TRADE CLOSED: PNL Gross {trade.pnl:.2f}, PNL Net {trade.pnlcomm:.2f}')
        # Reset bracket tracking on trade close
        self.bracket_orders = None
    
    def next(self):
        """Main strategy logic called on each bar"""
        
        # 1. Check for pending orders
        if self.order or self.bracket_orders:
            # Allow SMA crossover check even if bracket orders are active
            if self.order:
                return  # Specific buy/sell order is pending
        
        # 2. Position-based logic
        if not self.position:
            # --- ENTRY LOGIC ---
            
            # Ensure sufficient data for all indicators
            min_data_check = (len(self.sma_fast) < self.p.slow_ma or 
                            len(self.rsi) < self.p.rsi_period + 1 or
                            len(self.adx.adx) < self.p.adx_period + 1 or 
                            len(self.atr.atr) < self.p.atr_period + 1)
            if min_data_check:
                return  # Insufficient indicator data
            
            # Define all entry conditions
            
            # Condition 1: Bullish SMA crossover
            is_bullish_cross = (self.sma_fast[0] > self.sma_slow[0] and
                              self.sma_fast[-1] <= self.sma_slow[-1])
            
            # Condition 2: RSI filter (not overbought)
            is_rsi_acceptable = (self.rsi[0] < self.params.rsi_overbought)
            
            # Condition 3: ADX increasing (strengthening trend)
            is_adx_rising = (self.adx.adx[0] > self.adx.adx[-1])
            
            # Condition 4: ATR increasing (expanding volatility)
            is_atr_rising = (self.atr.atr[0] > self.atr.atr[-1])
            
            # COMBINED ENTRY SIGNAL: All conditions must be true
            if (is_bullish_cross and is_rsi_acceptable and 
                is_adx_rising and is_atr_rising):
                
                # Calculate bracket order prices
                entry_price = self.dataclose[0]
                sl_pct = self.params.stop_loss_pct / 100.0
                stop_price = entry_price * (1.0 - sl_pct)
                tp_pct = self.params.take_profit_pct / 100.0
                limit_price = entry_price * (1.0 + tp_pct)
                
                self.log(f'BUY SIGNAL @ {entry_price:.2f}: '
                        f'SMA Cross(✓) RSI(✓, {self.rsi[0]:.2f}) '
                        f'ADX Rising(✓, {self.adx.adx[0]:.2f}) '
                        f'ATR Rising(✓, {self.atr.atr[0]:.2f})')
                self.log(f'---> Placing Bracket Order: '
                        f'Stop Loss @ {stop_price:.2f}, Take Profit @ {limit_price:.2f}')
                
                # Place bracket order
                self.bracket_orders = self.buy_bracket(
                    price=entry_price,
                    stopprice=stop_price,
                    limitprice=limit_price
                )
                
                # Track the main buy order
                self.order = self.bracket_orders[0]
        
        else:  # Currently in position
            # --- EXIT LOGIC ---
            
            # Check for SMA crossover exit (bracket orders handle stop/profit)
            is_bearish_cross = (self.sma_fast[0] < self.sma_slow[0] and
                              self.sma_fast[-1] >= self.sma_slow[-1])
            
            if is_bearish_cross:
                self.log(f'SELL/CLOSE SIGNAL (SMA CROSS): Price: {self.dataclose[0]:.2f}')
                # Close position (automatically cancels pending bracket orders)
                self.order = self.close()
```

## Key Implementation Features

### Comprehensive Filtering
- **Four-Layer Confirmation**: SMA crossover + RSI filter + ADX rising + ATR rising
- **Data Validation**: Ensures sufficient indicator data before signal evaluation
- **Clear Condition Logic**: Each filter is independently calculated and clearly documented

### Advanced Order Management
- **Bracket Order Integration**: Simultaneous placement of entry, stop-loss, and take-profit orders
- **Order State Tracking**: Sophisticated tracking of multiple order types and states
- **Automatic Risk Management**: Stop-loss and take-profit execute without manual intervention

### Robust Error Handling
- **Order Status Management**: Comprehensive handling of all order status possibilities
- **Position Synchronization**: Proper cleanup of tracking variables when positions close
- **Bracket Order Coordination**: Ensures proper cancellation of remaining orders when position closes manually

This implementation demonstrates how theoretical concepts translate into practical, executable trading code with proper risk management and error handling.


---

[← Back to Outline](../outline.md)