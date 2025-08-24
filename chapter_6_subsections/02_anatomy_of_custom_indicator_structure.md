[← Back to Outline](../outline.md)

---

# Anatomy of a Custom Indicator: Structure

## Understanding the Foundation

Every indicator in backtrader, whether built-in or custom-developed, inherits from the `bt.Indicator` base class or its specialized subclasses like `bt.indicators.PeriodN`. Creating your own indicator involves defining a Python class that extends `bt.Indicator` and includes specific class attributes and methods that define its operational behavior.

## Essential Components

### 1. Class Inheritance

Begin by establishing your class hierarchy:

```python
import backtrader as bt

class MyCustomIndicator(bt.Indicator):
    # Indicator implementation follows...
    pass
```

### 2. Lines Declaration (Required)

The `lines` attribute is a tuple that specifies the names of output data streams your indicator will generate. These names serve as the interface for accessing calculated results:

```python
class MySimpleIndicator(bt.Indicator):
    # Single output line
    lines = ('value',)

class MyComplexIndicator(bt.Indicator):
    # Multiple output lines
    lines = ('primary_signal', 'secondary_signal',)
```

**Important Note:** Even for single-output indicators, `lines` must be defined as a tuple (note the trailing comma for single-element tuples).

### 3. Parameters Configuration (Optional)

Similar to strategy classes, indicators support configurable parameters through the `params` attribute. This tuple of tuples structure allows for flexible, reusable indicator implementations:

```python
class MyParameterizedIndicator(bt.Indicator):
    lines = ('smoothed_output',)
    
    params = (
        ('lookback_period', 20),      # Default lookback window
        ('smoothing_alpha', 0.1),     # Smoothing coefficient
        ('threshold_level', 0.0),     # Signal threshold
    )
```

Parameters are accessed within indicator methods using `self.params.parameter_name` or the shorthand `self.p.parameter_name`.

### 4. Plotting Configuration (Optional)

The `plotinfo` dictionary provides visualization hints to backtrader's plotting engine:

```python
class MyOscillatorIndicator(bt.Indicator):
    lines = ('oscillator',)
    params = (('period', 14),)
    
    plotinfo = dict(
        subplot=True,                    # Display in separate panel
        plotname='Custom Oscillator',   # Panel title
        plotlinelabels=True,            # Show line names in legend
        plothlines=[0.0],               # Add horizontal reference lines
    )
```

**Key Plotting Options:**

- `subplot=True`: Creates a new panel below the main price chart (ideal for oscillators)
- `subplot=False`: Attempts overlay on the main price chart (suitable for moving averages)
- `plotname='...'`: Sets the subplot panel title
- `plothlines=[...]`: Adds horizontal reference lines (useful for oscillators with zero-line crossovers)

## Calculation Logic Implementation

### The `__init__` Method and Line Arithmetic

The core computational logic resides primarily within the `__init__` method, ideally utilizing backtrader's efficient Line Arithmetic system.

#### Constructor Responsibilities

The `__init__(self)` method, called once during indicator instantiation, handles:

**Data Access:** Obtaining references to required input data lines through `self.data` (equivalent to `self.datas[0]`). Access specific price components via `self.data.close`, `self.data.high`, `self.data.volume`, etc.

**Indicator Composition:** Instantiating other indicators if your custom calculation builds upon existing analytical tools.

**Calculation Definition:** Expressing the computational logic using Line Arithmetic operations.

#### Line Arithmetic: The Preferred Approach

Backtrader's line objects represent entire arrays of time-series values. The framework overloads standard Python operators (`+`, `-`, `*`, `/`), comparison operators (`>`, `<`, `==`), logical operators (`&` for AND, `|` for OR), and mathematical functions to operate directly on these line objects.

When you perform operations like `self.data.high - self.data.low`, backtrader automatically applies this calculation across all available data points, returning another line object representing the computed results.

#### Implementation Examples

```python
# Example 1: Daily trading range calculation
def __init__(self):
    self.lines.daily_range = self.data.high - self.data.low

# Example 2: Midpoint price calculation
def __init__(self):
    self.lines.midpoint = (self.data.high + self.data.low) / 2.0

# Example 3: Time-lagged operations
def __init__(self):
    previous_close = self.data.close(-1)  # Previous bar's close
    self.lines.gap = self.data.open - previous_close

# Example 4: Incorporating existing indicators
def __init__(self):
    sma = bt.indicators.SimpleMovingAverage(
        self.data.close, 
        period=self.p.period
    )
    self.lines.smoothed_price = sma

# Example 5: Boolean conditions (results in 1/0 values)
def __init__(self):
    self.lines.bullish_bar = self.data.close > self.data.open
```

#### Advantages of Line Arithmetic

**Performance:** Operations are executed using optimized vectorized calculations, significantly faster than iterative Python loops.

**Simplicity:** Many indicator formulas can be expressed concisely using this notation.

**Automatic Period Detection:** Backtrader automatically determines minimum data requirements based on time lags and indicator periods.

### The `next()` Method (Special Cases Only)

While Line Arithmetic handles most scenarios, backtrader provides a `next(self)` method for complex logic that cannot be expressed through vectorized operations.

#### When to Use `next()`

Reserve the `next()` method for scenarios involving:

1. **Complex Conditional Logic:** Calculations depending on intricate state conditions or multi-bar pattern recognition
2. **Recursive Calculations:** Values explicitly depending on the indicator's own previous calculated values
3. **State Machine Logic:** Indicators requiring complex state tracking over time

#### Implementation Considerations

```python
def next(self):
    # Access current and historical values
    current_price = self.data.close[0]
    previous_output = self.lines.my_output[-1]
    
    # Perform complex calculations
    if current_price > previous_output * 1.01:
        calculated_value = current_price
    elif self.data.volume[0] > some_threshold:
        calculated_value = previous_output
    else:
        calculated_value = (current_price + previous_output) / 2
    
    # Assign result to current bar
    self.lines.my_output[0] = calculated_value
```

**Performance Warning:** Bar-by-bar calculations in `next()` are significantly slower than vectorized Line Arithmetic. Always attempt Line Arithmetic implementation first.

## Minimum Period Considerations

Backtrader typically determines the `minperiod` automatically when using Line Arithmetic with time lags or standard indicators. For complex `next()` implementations requiring specific historical data not obvious from simple lags, manually set the minimum period using `self.addminperiod(required_bars)` within `__init__`.

This comprehensive structure provides the foundation for creating powerful, efficient custom indicators that integrate seamlessly with backtrader's analytical framework.


---

[← Back to Outline](../outline.md)