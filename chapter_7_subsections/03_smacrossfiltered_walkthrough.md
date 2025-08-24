[← Back to Outline](../outline.md)

---

# SmaCrossFiltered - Strategy Walkthrough

Let's examine the key components of this combined signal strategy to understand how different elements work together to create a more sophisticated trading approach.

## Strategy Parameters

The `params` section defines configurable values that control the strategy's behavior:

- **fast_ma, slow_ma**: Periods for the fast and slow Simple Moving Averages
- **rsi_period**: Period for RSI calculation
- **rsi_overbought**: RSI threshold used for filtering entry signals

This parameterization allows for easy testing and optimization of different combinations without modifying the core logic.

## Initialization Phase

The `__init__` method sets up the strategy foundation:

- **Indicator Instantiation**: Creates two `SimpleMovingAverage` indicators (`self.sma_fast`, `self.sma_slow`) and one `RelativeStrengthIndex` indicator (`self.rsi`)
- **Order Tracking**: Initializes `self.order = None` for managing pending order states
- **Data References**: Establishes connection to price data through `self.dataclose`

## Order and Trade Management

**notify_order/notify_trade**: These methods implement standard patterns for handling order execution feedback and maintaining the `self.order` state. They provide essential logging for trade analysis and ensure proper order state management.

## Core Strategy Logic (next method)

The `next()` method contains the heart of the strategy's decision-making process:

### Order State Management
```python
if self.order: return
```
This critical check ensures no new logic executes while an order awaits completion, preventing conflicting signals.

### Position-Based Logic Flow
```python
if not self.position:  # Looking for entry
```
The strategy uses position status to determine whether to evaluate entry or exit conditions.

### Entry Logic Analysis

**Bullish Crossover Detection**:
```python
is_bullish_cross = (self.sma_fast[0] > self.sma_slow[0] and
                   self.sma_fast[-1] <= self.sma_slow[-1])
```
This condition requires checking both current state (fast > slow) and previous state (fast <= slow), ensuring the crossover occurred specifically on the current bar.

**RSI Filter Application**:
```python
is_rsi_acceptable = (self.rsi[0] < self.params.rsi_overbought)
```
This second condition acts as a momentum filter, preventing entries when RSI indicates potentially unsustainable momentum levels.

**Combined Signal Logic**:
```python
if is_bullish_cross and is_rsi_acceptable:
```
The `and` operator creates the core combination requirement. Both the bullish crossover signal AND the RSI filter condition must be true simultaneously. When the SMA crossover occurs but RSI exceeds the threshold, `is_rsi_acceptable` becomes `False`, preventing trade entry.

### Exit Logic Structure

**Position Status Check**:
```python
else:  # Currently in position
```
The `else` block handles scenarios where `self.position` evaluates to `True`.

**Bearish Crossover Detection**:
```python
is_bearish_cross = (self.sma_fast[0] < self.sma_slow[0] and
                   self.sma_fast[-1] >= self.sma_slow[-1])
```
This calculates the exit condition using the opposite crossover pattern.

**Position Closure**:
```python
if is_bearish_cross:
    self.order = self.close()
```
When the bearish crossover occurs, the position closes using `self.close()`.

## Strategy Benefits and Limitations

**Benefits**:
- **Reduced False Signals**: The RSI filter helps avoid entries during momentum extremes
- **Clear Logic Flow**: Well-defined entry and exit conditions
- **Flexible Parameters**: Easy adjustment for different market conditions

**Limitations**:
- **Lagging Nature**: Moving average crossovers inherently lag price action
- **Simple Exit Strategy**: Relies only on opposite crossover for exits
- **No Risk Management**: Lacks stop-loss or profit-taking mechanisms

This strategy demonstrates practical implementation of indicator filtering, using RSI to qualify SMA crossover signals and improve overall signal quality by avoiding entries during potentially unfavorable momentum conditions.


---

[← Back to Outline](../outline.md)