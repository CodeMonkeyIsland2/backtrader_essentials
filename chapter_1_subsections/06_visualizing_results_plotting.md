[← Back to Outline](../outline.md)

---

# Visualizing Results: Plotting

## The Importance of Visual Analysis

Numerical results provide quantitative insights, but visual representations reveal patterns, trends, and behavioral nuances that numbers alone cannot convey. Backtrader's integrated plotting capabilities transform your backtest data into comprehensive charts.

## Backtrader's Plotting Architecture

Backtrader leverages matplotlib to generate publication-quality charts that include:
- Price action (candlestick or OHLC bars)
- Volume indicators
- Technical indicators and overlays
- Buy/sell transaction markers
- Portfolio equity curves

## Basic Plotting Implementation

### Simple Plot Generation

```python
# Generate comprehensive backtest visualization
print("\nGenerating backtest visualization...")

try:
    # Create the plot with enhanced styling
    cerebro.plot(
        style='candlestick',    # Use candlestick charts for price data
        barup='green',          # Color for up candles
        bardown='red',          # Color for down candles
        volume=True,            # Include volume subplot
        iplot=False             # Disable inline plotting for scripts
    )
    print("Visualization successfully generated")
    
except Exception as e:
    print(f"Plotting failed: {e}")
    print("Ensure matplotlib is properly installed and configured")
```

### Advanced Plotting Configuration

```python
# Enhanced plotting with custom parameters
import matplotlib
matplotlib.use('Agg')  # Use non-interactive backend for scripts

try:
    cerebro.plot(
        style='candlestick',
        barup='#26a69a',       # Custom green color
        bardown='#ef5350',     # Custom red color
        volume=True,
        volumeup='#26a69a',    # Volume bar colors
        volumedown='#ef5350',
        grid=True,             # Add grid lines
        plotdist=0.1,          # Distance between subplots
        figsize=(16, 10),      # Figure size in inches
        tight=True             # Tight layout
    )
    
    # Save plot to file
    plt.savefig('backtest_results.png', dpi=300, bbox_inches='tight')
    print("High-resolution plot saved as 'backtest_results.png'")
    
except Exception as e:
    print(f"Advanced plotting failed: {e}")
```

## Understanding Plot Components

### Main Price Chart
- **Candlesticks**: Green/red candles showing OHLC data
- **Transaction Markers**: 
  - ▲ (triangle up): Buy orders
  - ▼ (triangle down): Sell orders
- **Price Labels**: Execution prices displayed near markers

### Volume Subplot
- **Volume Bars**: Trading volume for each period
- **Color Coordination**: Matches price candle colors
- **Volume Spikes**: Highlight significant trading activity

### Indicator Overlays
- **Moving Averages**: Trend lines overlaid on price
- **Bollinger Bands**: Volatility bands around price action
- **Support/Resistance**: Key price levels marked

## Customizing Plot Appearance

### Color Schemes
```python
# Professional dark theme
plot_config = {
    'style': 'candlestick',
    'barup': '#00E676',      # Bright green
    'bardown': '#FF1744',    # Bright red
    'volume': True,
    'grid': True,
    'plotdist': 0.05,
    'figsize': (20, 12)
}

cerebro.plot(**plot_config)
```

### Multiple Timeframe Plotting
```python
# When using multiple data feeds
cerebro.plot(
    numfigs=2,              # Separate figures for different assets
    plotmaster=cerebro.datas[0],  # Primary data feed for synchronization
    subplot=True            # Use subplots within figures
)
```

## Saving and Exporting Plots

### High-Quality Image Export
```python
import matplotlib.pyplot as plt

# Configure matplotlib for high-quality output
plt.rcParams['figure.dpi'] = 300
plt.rcParams['savefig.dpi'] = 300
plt.rcParams['font.size'] = 10

# Generate and save plot
cerebro.plot(style='candlestick', volume=True)
plt.savefig(
    'strategy_backtest.png',
    dpi=300,
    bbox_inches='tight',
    facecolor='white',
    edgecolor='none'
)
plt.close()  # Free memory
```

### PDF Report Generation
```python
from matplotlib.backends.backend_pdf import PdfPages

# Create multi-page PDF report
with PdfPages('backtest_report.pdf') as pdf:
    cerebro.plot(style='candlestick', volume=True)
    pdf.savefig(bbox_inches='tight')
    plt.close()
```

## Interactive Plotting Options

### Jupyter Notebook Integration
```python
# For Jupyter notebooks
%matplotlib inline

# Enable interactive plotting
cerebro.plot(
    iplot=True,             # Interactive plotting
    style='candlestick',
    volume=True
)
```

### Plotly Integration (Advanced)
```python
# For modern web-based interactive charts
# Note: Requires additional setup
import plotly.graph_objects as go
from plotly.subplots import make_subplots

# Custom plotly implementation would go here
# (This is beyond basic Backtrader plotting)
```

## Troubleshooting Common Issues

### Issue: Plot Not Displaying
```python
# Solution 1: Check matplotlib backend
import matplotlib
print(f"Current backend: {matplotlib.get_backend()}")

# Solution 2: Force display
import matplotlib.pyplot as plt
cerebro.plot()
plt.show()
```

### Issue: Poor Quality Output
```python
# Solution: Increase DPI and figure size
plt.rcParams['figure.dpi'] = 150
cerebro.plot(figsize=(16, 10))
```

### Issue: Memory Errors with Large Datasets
```python
# Solution: Use selective plotting
cerebro.plot(
    start=datetime.datetime(2023, 1, 1),  # Plot subset of data
    end=datetime.datetime(2023, 12, 31)
)
```

## Plot Interpretation Guidelines

1. **Price Action**: Look for trend patterns and reversal signals
2. **Volume Confirmation**: High volume should confirm price movements
3. **Entry/Exit Points**: Verify strategy logic through marker placement
4. **Performance Consistency**: Check for smooth equity curve progression
5. **Risk Management**: Identify periods of high volatility or drawdown

Your backtest visualization provides crucial insights into strategy performance and market behavior. Use these charts to refine your trading logic and validate your approach.


---

[← Back to Outline](../outline.md)