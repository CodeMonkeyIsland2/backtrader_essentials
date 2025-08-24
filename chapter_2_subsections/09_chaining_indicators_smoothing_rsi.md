[← Back to Outline](../outline.md)

---

# Chaining Indicators: Smoothing the RSI

## Advanced Indicator Composition Techniques

One of backtrader's most powerful architectural features is **indicator composability**—the ability to use one indicator's output as input data for another indicator. This capability enables sophisticated multi-layered analysis that can significantly enhance signal quality and reduce noise.

## The Motivation for Indicator Chaining

**Noise Reduction Challenge**: Raw indicators, particularly oscillators like RSI, can exhibit rapid fluctuations that may generate excessive signals, including potentially false whipsaw movements.

**Smoothing Solution**: Applying smoothing techniques directly to indicator outputs helps filter erratic movements while preserving underlying trend information, potentially improving signal reliability and reducing trading frequency.

## Practical Application: SMA-Smoothed RSI

A widely-employed technique involves calculating a Simple Moving Average of the RSI values themselves, creating a secondary smoothed oscillator that responds more gradually to market changes.

### Strategic Applications

**Cross-Signal Generation**: Generate trading signals when the raw RSI line crosses above or below its smoothed counterpart (RSI-SMA crossovers).

**Threshold Analysis**: Use the smoothed RSI-SMA line for overbought/oversold analysis instead of raw RSI values, potentially reducing false threshold crossings.

**Trend Analysis**: Analyze the RSI's own directional bias by examining the slope and direction of the RSI-SMA line.

## Implementation Framework

### Indicator Chain Setup

```python
# Within the strategy's __init__ method

def __init__(self):
    # Step 1: Create the primary RSI indicator
    self.rsi = bt.indicators.RelativeStrengthIndex(
        period=self.params.rsi_period  # e.g., 14
    )
    
    # Step 2: Create SMA using RSI as input data source
    self.rsi_sma = bt.indicators.SimpleMovingAverage(
        self.rsi,                              # Pass RSI indicator as data input!
        period=self.params.rsi_sma_period      # e.g., 10-period smoothing
    )
    
    # Alternative concise notation:
    # self.rsi_sma = bt.ind.SMA(self.rsi, period=self.params.rsi_sma_period)
    
    self.log(f'RSI smoothing SMA created with {self.params.rsi_sma_period}-period calculation')
```

### Key Implementation Details

**Data Source Specification**: Instead of passing `self.datas[0]` or `self.data.close`, we pass `self.rsi` as the data input. Backtrader recognizes that `self.rsi` represents a calculated indicator line and applies the SMA to those computed values.

**Parameter Management**: Consider adding `rsi_sma_period` to your strategy parameters for flexible smoothing period adjustment during optimization testing.

## Accessing Chained Indicator Values

Value access for chained indicators follows the same indexing pattern as all other indicators:

```python
# Within the next() method

def next(self):
    # Extract both raw and smoothed RSI values
    current_rsi_raw = self.rsi[0]
    current_rsi_smoothed = self.rsi_sma[0]
    
    # Example: RSI crossover signal logic
    if (current_rsi_raw > current_rsi_smoothed and 
        self.rsi[-1] <= self.rsi_sma[-1]):
        
        self.log(
            f'BULLISH RSI CROSSOVER: Raw RSI ({current_rsi_raw:.2f}) '
            f'crossed above smoothed RSI ({current_rsi_smoothed:.2f})'
        )
        # Potential buy signal consideration
    
    elif (current_rsi_raw < current_rsi_smoothed and 
          self.rsi[-1] >= self.rsi_sma[-1]):
        
        self.log(
            f'BEARISH RSI CROSSOVER: Raw RSI ({current_rsi_raw:.2f}) '
            f'crossed below smoothed RSI ({current_rsi_smoothed:.2f})'
        )
        # Potential sell signal consideration
```

## Advanced Chaining Possibilities

**Multi-Level Chains**: Indicators can be chained multiple levels deep, such as applying an RSI to a moving average of price, then smoothing that RSI with another moving average.

**Cross-Indicator Analysis**: Different indicator types can be combined, such as using MACD output as input for RSI calculations, enabling unique analytical perspectives.

**Complex Filtering**: Create sophisticated filtering mechanisms by combining multiple indicator chains with different parameters and timeframes.

This chaining capability transforms backtrader from a simple indicator library into a powerful analytical framework for developing sophisticated technical analysis systems.


---

[← Back to Outline](../outline.md)