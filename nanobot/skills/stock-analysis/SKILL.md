---
name: stock-analysis
description: "Analyze stock market data from global exchanges (NYSE, DSE, CSE) and forex (FX) with real-time quotes, technical indicators, and AI-powered trading suggestions."
metadata: {"nanobot":{"emoji":"📈","requires":{"bins":["curl"]}}}
---

# Stock Market Analysis

Use free financial APIs to fetch and analyze stock market data from global exchanges (NYSE, DSE, CSE) and forex markets. Includes AI-powered trading analysis and suggestions.

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

## Global Stock Exchanges

Access live data from major stock exchanges worldwide:

### NYSE (New York Stock Exchange)
```bash
# NYSE stocks (default US market)
curl -s "https://query1.finance.yahoo.com/v8/finance/chart/IBM?interval=1d&range=1d" | jq '.chart.result[0].meta'

# Get multiple NYSE stocks
for symbol in IBM GE JPM; do
  echo "NYSE: $symbol"
  curl -s "https://query1.finance.yahoo.com/v8/finance/chart/$symbol?interval=1d&range=1d" | jq '.chart.result[0].meta | {symbol, price: .regularMarketPrice, change: .regularMarketChangePercent}'
done
```

### DSE (Dhaka Stock Exchange - Bangladesh)
```bash
# DSE stocks use .DH suffix
# Example: SQURPHARMA (Square Pharmaceuticals)
curl -s "https://query1.finance.yahoo.com/v8/finance/chart/SQURPHARMA.DH?interval=1d&range=1d" | jq '.chart.result[0].meta'

# DSE major stocks
for symbol in SQURPHARMA.DH GP.DH BRAC.DH; do
  echo "DSE: $symbol"
  curl -s "https://query1.finance.yahoo.com/v8/finance/chart/$symbol?interval=1d&range=1d" | jq -r '.chart.result[0].meta | select(.) | {symbol, price: .regularMarketPrice, currency} | @json' 2>/dev/null || echo "Data not available"
done

# Alternative: Use Alpha Vantage for DSE data
# curl -s "https://www.alphavantage.co/query?function=GLOBAL_QUOTE&symbol=SQURPHARMA.DH&apikey=YOUR_KEY"
```

### CSE (Colombo Stock Exchange - Sri Lanka)
```bash
# CSE stocks use .CM suffix
# Example: JKH (John Keells Holdings)
curl -s "https://query1.finance.yahoo.com/v8/finance/chart/JKH.N0000?interval=1d&range=1d" | jq '.chart.result[0].meta'

# CSE major indices and stocks
for symbol in JKH.N0000 DIAL.N0000 COMB.N0000; do
  echo "CSE: $symbol"
  curl -s "https://query1.finance.yahoo.com/v8/finance/chart/$symbol?interval=1d&range=1d" | jq -r '.chart.result[0].meta | select(.) | {symbol, price: .regularMarketPrice} | @json' 2>/dev/null || echo "Data not available"
done
```

### Market Indices

Track major global market indices:

```bash
# US Indices
curl -s "https://query1.finance.yahoo.com/v8/finance/chart/%5EGSPC?interval=1d&range=1d" | jq '.chart.result[0].meta'  # S&P 500
curl -s "https://query1.finance.yahoo.com/v8/finance/chart/%5EDJI?interval=1d&range=1d" | jq '.chart.result[0].meta'   # Dow Jones
curl -s "https://query1.finance.yahoo.com/v8/finance/chart/%5EIXIC?interval=1d&range=1d" | jq '.chart.result[0].meta'  # NASDAQ

# Asian Indices
curl -s "https://query1.finance.yahoo.com/v8/finance/chart/%5EN225?interval=1d&range=1d" | jq '.chart.result[0].meta'  # Nikkei 225
curl -s "https://query1.finance.yahoo.com/v8/finance/chart/%5EHSI?interval=1d&range=1d" | jq '.chart.result[0].meta'   # Hang Seng
```

## Foreign Exchange (FX) & Forex

Access live forex rates and currency pairs:

```bash
# Major currency pairs
curl -s "https://query1.finance.yahoo.com/v8/finance/chart/EURUSD=X?interval=1d&range=1d" | jq '.chart.result[0].meta'  # EUR/USD
curl -s "https://query1.finance.yahoo.com/v8/finance/chart/GBPUSD=X?interval=1d&range=1d" | jq '.chart.result[0].meta'  # GBP/USD
curl -s "https://query1.finance.yahoo.com/v8/finance/chart/USDJPY=X?interval=1d&range=1d" | jq '.chart.result[0].meta'  # USD/JPY

# Emerging market currencies
curl -s "https://query1.finance.yahoo.com/v8/finance/chart/USDBDT=X?interval=1d&range=1d" | jq '.chart.result[0].meta'  # USD/BDT (Bangladesh Taka)
curl -s "https://query1.finance.yahoo.com/v8/finance/chart/USDLKR=X?interval=1d&range=1d" | jq '.chart.result[0].meta'  # USD/LKR (Sri Lankan Rupee)

# Multiple forex pairs with trend
for pair in EURUSD=X GBPUSD=X USDJPY=X USDBDT=X; do
  echo "=== $pair ==="
  curl -s "https://query1.finance.yahoo.com/v8/finance/chart/$pair?interval=1d&range=5d" | \
    jq -r '.chart.result[0] | {
      pair: .meta.symbol,
      current: .meta.regularMarketPrice,
      previous: .meta.previousClose,
      change_pct: .meta.regularMarketChangePercent
    }'
done
```

### Forex Analysis with Historical Data
```bash
# Download FX historical data for trend analysis
curl -s "https://query1.finance.yahoo.com/v8/finance/chart/EURUSD=X?interval=1d&range=3mo" | \
  jq -r '.chart.result[0] | .timestamp as $t | .indicators.quote[0].close as $c | 
  ["date","rate"], (range($t|length) | [($t[.]|todateiso8601), $c[.]]) | @csv' > eurusd_data.csv
```

## AI-Powered Trading Analysis & Suggestions

Use AI to analyze market data and generate trading insights:

### 1. Sentiment Analysis for Trading Signals

```python
# Combine price data with AI analysis for trading suggestions
import pandas as pd
import json
from urllib.request import urlopen

def fetch_stock_data(symbol, range_period='3mo'):
    """Fetch stock data and calculate indicators"""
    url = f"https://query1.finance.yahoo.com/v8/finance/chart/{symbol}?interval=1d&range={range_period}"
    data = json.loads(urlopen(url).read())
    
    result = data['chart']['result'][0]
    timestamps = result['timestamp']
    quote = result['indicators']['quote'][0]
    
    df = pd.DataFrame({
        'close': quote['close'],
        'high': quote['high'],
        'low': quote['low'],
        'volume': quote['volume']
    }, index=pd.to_datetime(timestamps, unit='s'))
    
    # Calculate technical indicators
    df['SMA_20'] = df['close'].rolling(window=20).mean()
    df['SMA_50'] = df['close'].rolling(window=50).mean()
    df['RSI'] = calculate_rsi(df['close'], 14)
    df['volatility'] = df['close'].rolling(window=20).std()
    
    return df

def calculate_rsi(prices, period=14):
    """Calculate Relative Strength Index"""
    delta = prices.diff()
    gain = (delta.where(delta > 0, 0)).rolling(window=period).mean()
    loss = (-delta.where(delta < 0, 0)).rolling(window=period).mean()
    rs = gain / loss
    rsi = 100 - (100 / (1 + rs))
    return rsi

def generate_trading_signal(df):
    """Generate AI-powered trading signal based on technical indicators"""
    latest = df.iloc[-1]
    prev = df.iloc[-2]
    
    signals = []
    confidence = 0
    
    # Signal 1: Moving Average Crossover
    if latest['SMA_20'] > latest['SMA_50'] and prev['SMA_20'] <= prev['SMA_50']:
        signals.append("BUY: Golden Cross detected (SMA20 crossed above SMA50)")
        confidence += 30
    elif latest['SMA_20'] < latest['SMA_50'] and prev['SMA_20'] >= prev['SMA_50']:
        signals.append("SELL: Death Cross detected (SMA20 crossed below SMA50)")
        confidence -= 30
    
    # Signal 2: RSI Analysis
    if latest['RSI'] < 30:
        signals.append("BUY: Oversold condition (RSI < 30)")
        confidence += 25
    elif latest['RSI'] > 70:
        signals.append("SELL: Overbought condition (RSI > 70)")
        confidence -= 25
    
    # Signal 3: Price Momentum
    price_change = (latest['close'] - prev['close']) / prev['close'] * 100
    if price_change > 2:
        signals.append(f"Strong upward momentum: +{price_change:.2f}%")
        confidence += 15
    elif price_change < -2:
        signals.append(f"Strong downward momentum: {price_change:.2f}%")
        confidence -= 15
    
    # Signal 4: Volatility Assessment
    if latest['volatility'] > df['volatility'].mean() * 1.5:
        signals.append("CAUTION: High volatility - increased risk")
        confidence -= 10
    
    # Determine overall recommendation
    if confidence > 40:
        recommendation = "STRONG BUY"
    elif confidence > 15:
        recommendation = "BUY"
    elif confidence > -15:
        recommendation = "HOLD"
    elif confidence > -40:
        recommendation = "SELL"
    else:
        recommendation = "STRONG SELL"
    
    return {
        'recommendation': recommendation,
        'confidence': confidence,
        'signals': signals,
        'price': latest['close'],
        'rsi': latest['RSI'],
        'sma_20': latest['SMA_20'],
        'sma_50': latest['SMA_50']
    }

# Example usage
symbol = "AAPL"  # Can use DSE stocks like "SQURPHARMA.DH", NYSE "IBM", or FX "EURUSD=X"
df = fetch_stock_data(symbol)
analysis = generate_trading_signal(df)

print(f"\n{'='*60}")
print(f"AI Trading Analysis for {symbol}")
print(f"{'='*60}")
print(f"Recommendation: {analysis['recommendation']} (Confidence: {analysis['confidence']})")
print(f"Current Price: ${analysis['price']:.2f}")
print(f"RSI: {analysis['rsi']:.2f}")
print(f"SMA 20: ${analysis['sma_20']:.2f} | SMA 50: ${analysis['sma_50']:.2f}")
print(f"\nSignals:")
for signal in analysis['signals']:
    print(f"  • {signal}")
print(f"{'='*60}\n")
```

### 2. Multi-Exchange Trading Suggestions

```python
def analyze_multi_exchange(symbols_dict):
    """
    Analyze stocks across multiple exchanges
    symbols_dict: {'NYSE': ['AAPL', 'IBM'], 'DSE': ['SQURPHARMA.DH'], 'FX': ['EURUSD=X']}
    """
    recommendations = {}
    
    for exchange, symbols in symbols_dict.items():
        recommendations[exchange] = {}
        for symbol in symbols:
            try:
                df = fetch_stock_data(symbol, range_period='1mo')
                analysis = generate_trading_signal(df)
                recommendations[exchange][symbol] = analysis
            except Exception as e:
                print(f"Error analyzing {symbol}: {e}")
    
    return recommendations

# Example: Analyze stocks from different exchanges
portfolio = {
    'NYSE': ['AAPL', 'MSFT', 'GOOGL'],
    'DSE': ['SQURPHARMA.DH', 'GP.DH'],  # If available
    'FX': ['EURUSD=X', 'GBPUSD=X']
}

results = analyze_multi_exchange(portfolio)

# Display recommendations
for exchange, stocks in results.items():
    print(f"\n{exchange} Trading Recommendations:")
    print("-" * 50)
    for symbol, analysis in stocks.items():
        print(f"{symbol:15s} | {analysis['recommendation']:12s} | Confidence: {analysis['confidence']:+3d}")
```

### 3. Risk Assessment & Portfolio Suggestions

```python
def assess_risk_and_suggest_portfolio(symbols, investment_amount=10000):
    """
    Assess risk across multiple assets and suggest portfolio allocation
    """
    analyses = {}
    total_confidence = 0
    
    for symbol in symbols:
        df = fetch_stock_data(symbol, range_period='3mo')
        analysis = generate_trading_signal(df)
        
        # Calculate risk score (0-100, higher = riskier)
        volatility_score = min(df['volatility'].iloc[-1] / df['close'].iloc[-1] * 100, 50)
        rsi_extremity = abs(50 - analysis['rsi']) / 50 * 30  # How far from neutral
        risk_score = volatility_score + rsi_extremity
        
        analysis['risk_score'] = risk_score
        analyses[symbol] = analysis
        total_confidence += max(analysis['confidence'], 0)
    
    # Suggest allocation based on confidence and risk
    print(f"\nPortfolio Allocation Suggestion (Total: ${investment_amount:,.0f})")
    print("=" * 70)
    
    for symbol, analysis in analyses.items():
        if analysis['confidence'] > 0:
            allocation_pct = max(analysis['confidence'], 0) / total_confidence if total_confidence > 0 else 0
            allocation_amount = investment_amount * allocation_pct
            
            print(f"{symbol:12s} | {analysis['recommendation']:12s} | "
                  f"Risk: {analysis['risk_score']:5.1f} | "
                  f"Allocation: {allocation_pct*100:5.1f}% (${allocation_amount:,.0f})")
    
    return analyses

# Example usage
watchlist = ['AAPL', 'MSFT', 'EURUSD=X']
portfolio_suggestion = assess_risk_and_suggest_portfolio(watchlist, investment_amount=10000)
```

### 4. Using AI/LLM for Enhanced Analysis

For deeper insights, use the nanobot's AI capabilities with collected data:

```bash
# After collecting stock data, ask nanobot for analysis
# Example prompt: "Based on this stock data, what's your trading recommendation?"

# Generate comprehensive analysis report
cat > stock_analysis_prompt.txt << 'EOF'
Analyze the following stock data and provide trading recommendations:

Symbol: AAPL
Current Price: $150.25
RSI: 45.2 (neutral)
SMA 20: $148.50
SMA 50: $145.00
Volume Trend: Increasing (+15% over 5 days)
News Sentiment: Positive (new product launch)

Provide:
1. Buy/Sell/Hold recommendation
2. Entry and exit points
3. Risk assessment
4. Time horizon suggestion
EOF

# Use with nanobot agent
nanobot agent -m "$(cat stock_analysis_prompt.txt)"
```

### 5. Real-Time Alert System

```python
def monitor_and_alert(symbols, check_interval=300):
    """
    Monitor stocks and generate alerts for trading opportunities
    check_interval: seconds between checks (default 5 minutes)
    """
    import time
    
    print("Starting real-time monitoring...")
    previous_signals = {}
    
    while True:
        for symbol in symbols:
            try:
                df = fetch_stock_data(symbol, range_period='1mo')
                analysis = generate_trading_signal(df)
                
                # Check if recommendation changed
                if symbol not in previous_signals or \
                   previous_signals[symbol]['recommendation'] != analysis['recommendation']:
                    
                    print(f"\n🔔 ALERT: {symbol}")
                    print(f"   New Signal: {analysis['recommendation']}")
                    print(f"   Price: ${analysis['price']:.2f}")
                    print(f"   Reasons: {', '.join(analysis['signals'][:2])}")
                    
                    previous_signals[symbol] = analysis
                    
            except Exception as e:
                print(f"Error monitoring {symbol}: {e}")
        
        time.sleep(check_interval)

# Example: Monitor NYSE, DSE, and FX
# monitor_and_alert(['AAPL', 'SQURPHARMA.DH', 'EURUSD=X'], check_interval=300)
```

### Trading Strategy Guidelines

**Important Considerations:**
1. **Do Your Research**: AI suggestions are based on technical indicators only. Always research fundamentals.
2. **Risk Management**: Never invest more than you can afford to lose. Diversify across exchanges and asset classes.
3. **Market Hours**: DSE, NYSE, and CSE have different trading hours. Check exchange schedules.
4. **Regulatory Compliance**: Ensure you comply with local trading regulations.
5. **Paper Trading**: Test strategies with virtual money before real trading.
6. **Stop Losses**: Always set stop-loss orders to limit potential losses.
7. **Position Sizing**: Use only 1-5% of portfolio per trade for risk management.

**Exchange-Specific Tips:**
- **NYSE**: High liquidity, suitable for day trading and swing trading
- **DSE/CSE**: Lower liquidity, better for medium to long-term positions
- **FX (Forex)**: 24/5 market, high leverage available but also high risk

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
