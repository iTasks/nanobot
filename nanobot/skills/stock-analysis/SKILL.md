---
name: stock-analysis
description: "Analyze stock market data, get real-time quotes, historical data, and technical indicators using free financial APIs."
metadata: {"nanobot":{"emoji":"📈","requires":{"bins":["curl"]}}}
---

# Stock Market Analysis

Use free financial APIs to fetch and analyze stock market data. No API keys required for basic functionality.

## Quick Stock Quote

Get real-time stock data using Yahoo Finance (via finnhub-style endpoints):

```bash
# Get stock quote
curl -s "https://query1.finance.yahoo.com/v8/finance/chart/AAPL?interval=1d&range=1d" | jq '.chart.result[0].meta | {symbol, regularMarketPrice, currency}'

# Alternative: Using Alpha Vantage (requires free API key)
# Get your key at: https://www.alphavantage.co/support/#api-key
curl -s "https://www.alphavantage.co/query?function=GLOBAL_QUOTE&symbol=AAPL&apikey=demo" | jq '.["Global Quote"]'
```

## Historical Data

Fetch historical price data:

```bash
# Yahoo Finance - last 30 days
curl -s "https://query1.finance.yahoo.com/v8/finance/chart/AAPL?interval=1d&range=1mo" | jq '.chart.result[0].indicators.quote[0]'

# Get timestamps and closing prices
curl -s "https://query1.finance.yahoo.com/v8/finance/chart/AAPL?interval=1d&range=1mo" | jq -r '.chart.result[0] | .timestamp as $t | .indicators.quote[0].close as $c | range($t|length) | [$t[.], $c[.]] | @csv'
```

## Market Analysis Patterns

### Calculate Simple Moving Average (SMA)
```bash
# Fetch 30 days of closing prices and calculate SMA
curl -s "https://query1.finance.yahoo.com/v8/finance/chart/AAPL?interval=1d&range=1mo" | jq '.chart.result[0].indicators.quote[0].close | add / length'
```

### Volatility Analysis
```bash
# Get high and low prices for volatility range
curl -s "https://query1.finance.yahoo.com/v8/finance/chart/AAPL?interval=1d&range=1mo" | jq '.chart.result[0].indicators.quote[0] | {high: .high, low: .low}'
```

## Company Information

Get company fundamentals:

```bash
# Company profile (Yahoo Finance)
curl -s "https://query1.finance.yahoo.com/v10/finance/quoteSummary/AAPL?modules=assetProfile,financialData" | jq '.quoteSummary.result[0]'
```

## Multiple Stocks Comparison

Compare multiple stocks:

```bash
# Fetch quotes for multiple symbols
for symbol in AAPL MSFT GOOGL; do
  echo "=== $symbol ==="
  curl -s "https://query1.finance.yahoo.com/v8/finance/chart/$symbol?interval=1d&range=1d" | jq '.chart.result[0].meta | {symbol, price: .regularMarketPrice, change: .regularMarketChangePercent}'
done
```

## Market Indices

Track major market indices:

```bash
# S&P 500
curl -s "https://query1.finance.yahoo.com/v8/finance/chart/%5EGSPC?interval=1d&range=1d" | jq '.chart.result[0].meta'

# Dow Jones
curl -s "https://query1.finance.yahoo.com/v8/finance/chart/%5EDJI?interval=1d&range=1d" | jq '.chart.result[0].meta'

# NASDAQ
curl -s "https://query1.finance.yahoo.com/v8/finance/chart/%5EIXIC?interval=1d&range=1d" | jq '.chart.result[0].meta'
```

## Technical Indicators with Python

For advanced analysis, use Python with pandas:

```python
import pandas as pd
import json
from urllib.request import urlopen

# Fetch historical data
symbol = "AAPL"
url = f"https://query1.finance.yahoo.com/v8/finance/chart/{symbol}?interval=1d&range=3mo"
data = json.loads(urlopen(url).read())

# Extract price data
result = data['chart']['result'][0]
timestamps = result['timestamp']
closes = result['indicators']['quote'][0]['close']
highs = result['indicators']['quote'][0]['high']
lows = result['indicators']['quote'][0]['low']

# Create DataFrame
df = pd.DataFrame({
    'close': closes,
    'high': highs,
    'low': lows
}, index=pd.to_datetime(timestamps, unit='s'))

# Calculate technical indicators
df['SMA_20'] = df['close'].rolling(window=20).mean()
df['SMA_50'] = df['close'].rolling(window=50).mean()
df['volatility'] = df['close'].rolling(window=20).std()
df['daily_return'] = df['close'].pct_change()

print(df.tail(10))
```

## Tips and Best Practices

1. **Rate Limits**: Yahoo Finance is free but has rate limits. Add delays between requests if fetching multiple stocks.
2. **Data Quality**: Free APIs may have delayed data (15-20 minutes). For real-time data, consider paid services.
3. **Error Handling**: Always check if data exists before processing (use `jq` filters with defaults).
4. **Currency**: Yahoo Finance returns prices in the stock's trading currency.
5. **Symbol Format**: 
   - US stocks: `AAPL`
   - Indices: `^GSPC` (S&P 500), `^DJI` (Dow), `^IXIC` (NASDAQ)
   - International: `TSLA.L` (London), `7203.T` (Tokyo)

## Alternative APIs

If Yahoo Finance is unavailable, try these alternatives:

- **Finnhub** (free tier): https://finnhub.io (requires API key)
- **Alpha Vantage** (free tier): https://www.alphavantage.co (requires API key)
- **IEX Cloud** (free tier): https://iexcloud.io (requires API key)
- **Twelve Data** (free tier): https://twelvedata.com (requires API key)

## Data Engineering for Stock Analysis

Process and analyze large datasets:

```bash
# Download historical data and save to CSV
curl -s "https://query1.finance.yahoo.com/v8/finance/chart/AAPL?interval=1d&range=1y" | \
  jq -r '.chart.result[0] | .timestamp as $t | .indicators.quote[0] | .open as $o | .high as $h | .low as $l | .close as $c | .volume as $v | ["date","open","high","low","close","volume"], (range($t|length) | [($t[.]|todateiso8601), $o[.], $h[.], $l[.], $c[.], $v[.]]) | @csv' > stock_data.csv

# Analyze with awk for quick stats
awk -F, 'NR>1 {sum+=$5; count++} END {print "Average Close:", sum/count}' stock_data.csv
```
