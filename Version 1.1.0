import MetaTrader5 as mt5
import time

# 🛠️ MetaTrader 5 Credentials (Modify with your account details)
MT5_LOGIN = 12345678
MT5_PASSWORD = "yourpassword"
MT5_SERVER = "YourBroker-Server"

# ⚙️ Trading Parameters
SYMBOL = "EURUSD"
LOT_SIZE = 0.1
SLIPPAGE = 3  # Reduce slippage for fast execution
MAGIC_NUMBER = 123456
DELTA_THRESHOLD = 100  # Adjust for aggressiveness
STOP_LOSS_PIPS = 10  # Stop-loss in pips
TAKE_PROFIT_PIPS = 20  # Take-profit in pips

# ⚡ Fast Execution Settings
MAX_SPREAD = 2.0  # Maximum allowed spread (in pips)
CHECK_INTERVAL = 0.5  # Faster execution (every 0.5 seconds)

# 📡 Connect to MT5
def connect_mt5():
    if not mt5.initialize():
        print("MT5 Initialization Failed!")
        quit()
    
    if not mt5.login(MT5_LOGIN, password=MT5_PASSWORD, server=MT5_SERVER):
        print("MT5 Login Failed!")
        quit()
    
    print("✅ Connected to MT5!")

# 📊 Get Live Market Depth (DOM)
def get_dom():
    dom = mt5.market_book_get(SYMBOL)
    if dom is None:
        print("⚠️ No DOM data available.")
        return None
    return dom

# 🔄 Analyze DOM & Footprint Delta
def analyze_dom():
    dom = get_dom()
    if not dom:
        return None, 0

    buy_limits = sum(item.volume for item in dom if item.type == mt5.BOOK_TYPE_BUY_LIMIT)
    sell_limits = sum(item.volume for item in dom if item.type == mt5.BOOK_TYPE_SELL_LIMIT)
    market_buy_orders = sum(item.volume for item in dom if item.type == mt5.BOOK_TYPE_BUY)
    market_sell_orders = sum(item.volume for item in dom if item.type == mt5.BOOK_TYPE_SELL)

    footprint_delta = market_buy_orders - market_sell_orders
    spread = (mt5.symbol_info_tick(SYMBOL).ask - mt5.symbol_info_tick(SYMBOL).bid) / mt5.symbol_info(SYMBOL).point

    print(f"📊 Buy Limits: {buy_limits}, Sell Limits: {sell_limits}, Delta: {footprint_delta}, Spread: {spread}")

    if spread > MAX_SPREAD:  # Avoid trading in high spreads
        print("⚠️ Spread too high, skipping trade.")
        return None, footprint_delta

    if buy_limits > sell_limits and footprint_delta > DELTA_THRESHOLD:
        return "BUY", footprint_delta
    elif sell_limits > buy_limits and footprint_delta < -DELTA_THRESHOLD:
        return "SELL", footprint_delta
    return None, footprint_delta

# ⚡ Execute Live Trade
def place_order(order_type):
    order_type_mt5 = mt5.ORDER_TYPE_BUY if order_type == "BUY" else mt5.ORDER_TYPE_SELL
    tick = mt5.symbol_info_tick(SYMBOL)

    price = tick.ask if order_type == "BUY" else tick.bid
    sl = price - STOP_LOSS_PIPS * mt5.symbol_info(SYMBOL).point if order_type == "BUY" else price + STOP_LOSS_PIPS * mt5.symbol_info(SYMBOL).point
    tp = price + TAKE_PROFIT_PIPS * mt5.symbol_info(SYMBOL).point if order_type == "BUY" else price - TAKE_PROFIT_PIPS * mt5.symbol_info(SYMBOL).point

    request = {
        "action": mt5.TRADE_ACTION_DEAL,
        "symbol": SYMBOL,
        "volume": LOT_SIZE,
        "type": order_type_mt5,
        "price": price,
        "slippage": SLIPPAGE,
        "magic": MAGIC_NUMBER,
        "comment": "Live DOM + Delta Trade",
        "type_filling": mt5.ORDER_FILLING_IOC,
        "type_time": mt5.ORDER_TIME_GTC,
        "sl": sl,
        "tp": tp,
    }

    result = mt5.order_send(request)
    if result.retcode == mt5.TRADE_RETCODE_DONE:
        print(f"✅ Trade Executed: {order_type} at {price}")
    else:
        print(f"❌ Trade Failed: {result.comment}")

# 🚀 Start Fast Execution
connect_mt5()
while True:
    trade_signal, delta = analyze_dom()
    if trade_signal:
        print(f"⚡ Executing {trade_signal} due to DOM & Delta {delta}")
        place_order(trade_signal)
    time.sleep(CHECK_INTERVAL)  # Fast execution

# 🔴 Shutdown MT5 when finished
mt5.shutdown()
