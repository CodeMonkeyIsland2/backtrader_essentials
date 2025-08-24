[← Back to Outline](../outline.md)

---

# Understanding the Directional Indicators: +DI and -DI

Before diving into the ADX calculation itself, it's essential to understand its foundational components: the Plus Directional Indicator (+DI) and Minus Directional Indicator (-DI). These indicators work together to quantify the directional pressure within the market over a specified time period.

## The Building Blocks: Directional Movement

The directional indicators are built upon the concept of **Directional Movement**, which Wilder designed to capture significant price movements in either direction. The calculation involves several key steps:

### 1. Calculating Directional Movement (+DM and -DM)

For each trading period, the system compares current price levels to previous levels to determine directional movement:

**Plus Directional Movement (+DM)**:
- Captures significant upward price expansion
- Calculated when the current high exceeds the previous high
- Only registered if the upward move is greater than any downward move
- Formula: +DM = Current High - Previous High (when conditions are met)
- Otherwise, +DM = 0

**Minus Directional Movement (-DM)**:
- Captures significant downward price expansion  
- Calculated when the current low falls below the previous low
- Only registered if the downward move is greater than any upward move
- Formula: -DM = Previous Low - Current Low (when conditions are met)
- Otherwise, -DM = 0

**Important Note**: When neither upward nor downward movement dominates (such as during inside bars), both +DM and -DM equal zero.

### 2. True Range (TR) Component

The True Range serves as the denominator in the directional indicator calculations, representing the period's total price volatility. TR is the maximum of:

- Current High - Current Low
- |Current High - Previous Close|
- |Current Low - Previous Close|

### 3. Computing the Directional Indicators

The final +DI and -DI values are derived by:

1. Smoothing the raw +DM, -DM, and TR values over a specified period (typically 14)
2. Using Wilder's smoothing technique (similar to exponential moving averages)
3. Applying the formula:

```
+DI = 100 × (Smoothed +DM / Smoothed TR)
-DI = 100 × (Smoothed -DM / Smoothed TR)
```

## Key Characteristics

- **Range**: Both indicators typically oscillate between 0 and 100
- **Scaling**: Values are expressed as percentages for easier interpretation
- **Smoothing**: The use of smoothed values reduces noise and provides more stable readings
- **Relative Measurement**: The indicators measure directional strength relative to overall market volatility

## Mathematical Foundation

The mathematical elegance of the DI system lies in its ability to normalize directional movement against market volatility. By dividing smoothed directional movement by smoothed True Range, the indicators provide a standardized measure that works across different markets and volatility environments.

This normalization ensures that a $1 move in a $10 stock carries the same relative weight as a $10 move in a $100 stock, making the indicators universally applicable across various financial instruments.

Understanding these building blocks is crucial before exploring how the +DI and -DI indicators are interpreted and used in trading decisions.



---

[← Back to Outline](../outline.md)