[← Back to Outline](../outline.md)

---

# SMA Use Case: Crossover Logic Examples

## Building Trading Signals with Crossover Analysis

Accessing current and historical SMA values enables implementation of sophisticated trading signals based on crossover patterns—one of the most fundamental and widely-used signal generation techniques in technical analysis.

## Primary Crossover Strategy: Price-SMA Intersection

This approach generates signals when price action crosses above or below the SMA line, indicating potential trend changes or momentum shifts.

### Implementation Example

```python
# Within the next() method

def next(self):
    # Detect bullish crossover (price crosses above SMA)
    if (self.dataclose[0] > self.sma[0] and 
        self.dataclose[-1] <= self.sma[-1]):
        
        self.log(
            f'BULLISH CROSSOVER: Price {self.dataclose[0]:.2f} '
            f'crossed above SMA {self.sma[0]:.2f}'
        )
        # Potential buy signal implementation
        # self.buy()
    
    # Detect bearish crossover (price crosses below SMA)
    elif (self.dataclose[0] < self.sma[0] and 
          self.dataclose[-1] >= self.sma[-1]):
        
        self.log(
            f'BEARISH CROSSOVER: Price {self.dataclose[0]:.2f} '
            f'crossed below SMA {self.sma[0]:.2f}'
        )
        # Potential sell signal implementation
        # self.sell() or self.close()
```

### Logic Analysis

**Crossover Detection Method**: The algorithm validates two conditions simultaneously:
1. **Current Relationship**: Establishes the present price-SMA relationship (above/below)
2. **Previous Relationship**: Confirms the opposite relationship existed in the prior bar

**Signal Precision**: This dual-condition approach ensures signals trigger only when the relationship actually changes, preventing repeated signals while price remains on one side of the SMA.

## Advanced Strategy: Dual-SMA Crossover System

This sophisticated approach employs two SMAs with different periods—a responsive "fast" SMA and a stable "slow" SMA—to generate more robust trend-following signals.

### Popular Crossover Patterns

**Golden Cross**: Fast SMA crosses above slow SMA, suggesting strengthening bullish momentum and potential uptrend initiation.

**Death Cross**: Fast SMA crosses below slow SMA, indicating weakening momentum and potential downtrend development.

### Implementation Framework

```python
# Requires additional SMA definitions in __init__:
# self.sma_fast = bt.ind.SMA(period=10)  # Fast-moving average
# self.sma_slow = bt.ind.SMA(period=30)  # Slow-moving average

def next(self):
    # Detect bullish Golden Cross
    if (self.sma_fast[0] > self.sma_slow[0] and 
        self.sma_fast[-1] <= self.sma_slow[-1]):
        
        self.log(
            f'GOLDEN CROSS: Fast SMA ({self.sma_fast[0]:.2f}) '
            f'crossed above Slow SMA ({self.sma_slow[0]:.2f})'
        )
        # Potential long entry signal
    
    # Detect bearish Death Cross  
    elif (self.sma_fast[0] < self.sma_slow[0] and 
          self.sma_fast[-1] >= self.sma_slow[-1]):
        
        self.log(
            f'DEATH CROSS: Fast SMA ({self.sma_fast[0]:.2f}) '
            f'crossed below Slow SMA ({self.sma_slow[0]:.2f})'
        )
        # Potential short entry or long exit signal
```

## Visual Validation and Analysis

**Plotting Integration**: When executing `cerebro.plot()`, these SMA lines appear directly overlaid on the price chart, providing immediate visual confirmation of crossover events.

**Strategy Verification**: The visual representation allows validation that your code correctly identifies crossover points by comparing programmatic signals with visually observable intersections.

**Pattern Recognition**: Charts reveal the relationship between SMA crossovers and subsequent price movements, helping refine signal interpretation and timing strategies.

This visual feedback proves invaluable for developing intuition about indicator behavior and validating trading logic before deploying capital in live markets.


---

[← Back to Outline](../outline.md)