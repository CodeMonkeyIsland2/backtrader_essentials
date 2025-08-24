[← Back to Outline](../outline.md)

---

# Acquiring Historical Market Data

## Overview

Successful backtesting requires high-quality historical market data. This section demonstrates how to efficiently retrieve and prepare financial data for use in your trading strategies.

## Understanding Market Data Structure

Financial market data typically consists of:
- **Open**: Opening price for the period
- **High**: Highest price during the period  
- **Low**: Lowest price during the period
- **Close**: Closing price for the period
- **Volume**: Number of shares/contracts traded
- **Adjusted Close**: Price adjusted for dividends and stock splits

## Data Retrieval with yfinance

### Basic Data Download

```python
# Configure your data parameters
symbol = 'AAPL'
start_date = '2020-01-01'
end_date = '2023-12-31'

print(f"Fetching {symbol} data from {start_date} to {end_date}...")

# Retrieve historical data
try:
    # Download market data
    stock_data = yf.download(symbol, start=start_date, end=end_date)
    
    # Clean column structure (remove multi-level index if present)
    if stock_data.columns.nlevels > 1:
        stock_data.columns = stock_data.columns.droplevel(1)
    
    print(f"Successfully retrieved {stock_data.shape[0]} trading days")
    
    # Display sample data
    print("\nFirst 5 rows:")
    print(stock_data.head())
    
    print("\nData structure:")
    stock_data.info()
    
except Exception as error:
    print(f"Data retrieval failed: {error}")
    exit(1)
```

### Data Validation and Preparation

```python
# Ensure proper date indexing
if not isinstance(stock_data.index, pd.DatetimeIndex):
    print("Converting index to datetime format...")
    stock_data.index = pd.to_datetime(stock_data.index)

# Validate data completeness
if stock_data.empty:
    print("Warning: No data retrieved. Check symbol and date range.")
    exit(1)

print("Market data successfully prepared for backtesting.")
```

## Expected Output Structure

Your DataFrame should contain columns similar to:
```
            Open     High      Low    Close    Volume
Date                                                  
2020-01-02  72.72    72.78    71.47    72.01  135480400
2020-01-03  72.01    72.77    71.78    72.58  146322800
2020-01-06  72.58    72.62    70.88    72.24  118387200
```

## Data Quality Considerations

1. **Missing Data**: Check for gaps in your dataset
2. **Corporate Actions**: Use Adjusted Close for accurate backtesting
3. **Volume Data**: Ensure volume data is available for liquidity analysis
4. **Date Range**: Verify your data covers the intended testing period

## Alternative Data Sources

While yfinance is convenient for learning, consider these alternatives for production use:
- **Alpha Vantage**: Professional-grade API with extensive coverage
- **Quandl**: Economic and financial data platform
- **IEX Cloud**: Real-time and historical market data
- **Direct Broker APIs**: For live trading integration

## Data Storage Tips

For repeated backtests, consider caching your data:

```python
# Save data locally
stock_data.to_csv(f'{symbol}_data.csv')

# Load previously saved data
# stock_data = pd.read_csv(f'{symbol}_data.csv', index_col=0, parse_dates=True)
```

This approach reduces API calls and improves backtesting efficiency during strategy development.


---

[← Back to Outline](../outline.md)