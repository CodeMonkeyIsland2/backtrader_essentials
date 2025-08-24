[← Back to Outline](../outline.md)

---

# Feeding Data into Backtrader

## Data Feed Conversion

Backtrader operates with its own optimized data structures rather than directly using Pandas DataFrames. This section explains how to convert your market data into Backtrader-compatible feeds.

## Understanding Backtrader Data Feeds

Backtrader uses specialized data feed objects that provide:
- Optimized memory usage for large datasets
- Built-in data validation and error handling
- Automatic synchronization across multiple timeframes
- Seamless integration with indicators and strategies

## Converting Pandas Data to Backtrader Format

### Basic Data Feed Creation

```python
# Transform DataFrame into Backtrader data feed
data_feed = bt.feeds.PandasData(
    dataname=stock_data,
    fromdate=datetime.datetime.strptime(start_date, '%Y-%m-%d'),
    todate=datetime.datetime.strptime(end_date, '%Y-%m-%d')
)

print(f"Backtrader data feed created: {type(data_feed)}")
```

### Column Mapping Configuration

If your DataFrame columns don't match Backtrader's expected names, you can specify custom mappings:

```python
# Custom column mapping example
custom_data_feed = bt.feeds.PandasData(
    dataname=stock_data,
    datetime=None,  # Use DataFrame index
    open='Open',
    high='High', 
    low='Low',
    close='Close',
    volume='Volume',
    openinterest=None,  # Not available in stock data
    fromdate=datetime.datetime.strptime(start_date, '%Y-%m-%d'),
    todate=datetime.datetime.strptime(end_date, '%Y-%m-%d')
)
```

## Advanced Data Feed Options

### Using Adjusted Close Prices

For accurate backtesting that accounts for dividends and stock splits:

```python
# Use adjusted close for more realistic results
adjusted_data_feed = bt.feeds.PandasData(
    dataname=stock_data,
    close='Adj Close',  # Use adjusted close instead of regular close
    fromdate=datetime.datetime.strptime(start_date, '%Y-%m-%d'),
    todate=datetime.datetime.strptime(end_date, '%Y-%m-%d')
)
```

### Data Feed Validation

```python
# Verify data feed properties
print(f"Data feed time range: {data_feed.fromdate} to {data_feed.todate}")
print(f"Total data points: {len(stock_data)}")

# Check for data completeness
if len(stock_data) == 0:
    raise ValueError("Data feed is empty - check your data source")
```

## Multiple Data Feeds

For strategies trading multiple instruments:

```python
# Create feeds for multiple assets
symbols = ['AAPL', 'GOOGL', 'MSFT']
data_feeds = []

for symbol in symbols:
    symbol_data = yf.download(symbol, start=start_date, end=end_date)
    
    if symbol_data.columns.nlevels > 1:
        symbol_data.columns = symbol_data.columns.droplevel(1)
    
    feed = bt.feeds.PandasData(
        dataname=symbol_data,
        fromdate=datetime.datetime.strptime(start_date, '%Y-%m-%d'),
        todate=datetime.datetime.strptime(end_date, '%Y-%m-%d')
    )
    
    data_feeds.append(feed)
    print(f"Created data feed for {symbol}")
```

## Data Feed Performance Tips

1. **Date Filtering**: Set appropriate fromdate/todate to limit memory usage
2. **Column Selection**: Only include necessary columns in your DataFrame
3. **Data Types**: Ensure numeric columns use appropriate data types (float64, int64)
4. **Index Sorting**: Verify your DataFrame index is sorted chronologically

## Common Issues and Solutions

### Issue: Column Name Mismatches
```python
# Solution: Standardize column names
stock_data.columns = [col.lower() for col in stock_data.columns]
```

### Issue: Missing Date Index
```python
# Solution: Ensure proper datetime indexing
if 'Date' in stock_data.columns:
    stock_data.set_index('Date', inplace=True)
    stock_data.index = pd.to_datetime(stock_data.index)
```

### Issue: Data Gaps
```python
# Solution: Check for and handle missing data
print(f"Missing data points: {stock_data.isnull().sum().sum()}")
# Consider forward-fill, interpolation, or exclusion based on your strategy
```

Your data is now properly formatted and ready for integration with the Backtrader engine.


---

[← Back to Outline](../outline.md)