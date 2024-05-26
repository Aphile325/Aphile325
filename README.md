import MetaTrader5 as mt5
import pandas as pd

# Connect to MetaTrader 5
if not mt5.initialize():
    print("initialize() failed")
    mt5.shutdown()
    quit()

# Define the symbol and timeframe
symbol = "EURUSD"
timeframe = mt5.TIMEFRAME_M15

# Get historical data
rates = mt5.copy_rates_from_pos(symbol, timeframe, 0, 100)
df = pd.DataFrame(rates)
df['time'] = pd.to_datetime(df['time'], unit='s')

# Calculate moving averages
df['MA_50'] = df['close'].rolling(window=50).mean()
df['MA_200'] = df['close'].rolling(window=200).mean()

# Buy/sell logic
if df['MA_50'].iloc[-1] > df['MA_200'].iloc[-1]:
    lot_size = 0.1
    mt5.order_send(symbol, mt5.ORDER_TYPE_BUY, lot_size, 0, 0, 0, 0, "")
    print("Buy order sent")
elif df['MA_50'].iloc[-1] < df['MA_200'].iloc[-1]:
    lot_size = 0.1
    mt5.order_send(symbol, mt5.ORDER_TYPE_SELL, lot_size, 0, 0, 0, 0, "")
    print("Sell order sent")

# Disconnect from MetaTrader 5
mt5.shutdown()
