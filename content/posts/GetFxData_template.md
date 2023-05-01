---
title: "Get Fx Data Template"
date: 2023-05-01T09:06:04+09:00
image: "img/2023-05-01-09-19-38.png"
# bookComments: false
# bookSearchExclude: false
---


# FXの時間足データを取ってくる
意外とめんどくさいので自分用のテンプレートを記載。

```python
import MetaTrader5 as mt5
import pandas as pd
from datetime import datetime, timezone

# connect to MetaTrader 5
if not mt5.initialize(login=ここは数字が入る, server="XMTrading-MT5", password="なんとかかんとか"):
    print("initialize() failed, error code =",mt5.last_error())
    mt5.shutdown()

# 銘柄、時間足、開始日時、終了日時を指定
symbol = "GBPJPY"
date_from = datetime(2023,1,1, tzinfo=timezone.utc)
date_to = datetime(2023,1,31, tzinfo=timezone.utc)

# 各足
d_tf=mt5.TIMEFRAME_D1
h4_tf=mt5.TIMEFRAME_H4
h1_tf=mt5.TIMEFRAME_H1
m5_tf=mt5.TIMEFRAME_M5

# 足を引数として受け取ってレートを取得、dfを返す関数
def get_rates(tf):
    rates = mt5.copy_rates_range(symbol, tf, date_from, date_to)
    df = pd.DataFrame(rates)
    df['Time'] = pd.to_datetime(df['time'], unit='s')
    df.set_index(df['Time'],inplace=True)
    ohlc =df.rename(columns={"open":'open','high':'high','low':'low','close':'close','tick_volume':'volume'})
    ohlc = ohlc.drop(columns=['spread','real_volume','Time','time'],axis=1)
    return ohlc

d_df = get_rates(d_tf)
h4_df = get_rates(h4_tf)
h1_df = get_rates(h1_tf)
m5_df = get_rates(m5_tf)
```

こうすると`d_df`には日足、`h4_df`には4時間足、`h1_df`には1時間足、`m5_df`には5分足のデータが入る。
そして、全ての時間足で一貫の`Time`, `open`, `high`, `low`, `close`, `volume`の6列のデータフレームが得られる。
よって、`Time`列をインデックスにしておくと、`d_df['2023-01-01':'2023-01-05']`のように日付でスライスできる。
また、終了は`mt5.shutdown()`で行う。

## 余談
試しに簡単なアルゴリズムをバックテストしてみる。
- 日足、４時間足、１時間足の陰線か陽線かが一致したタイミングで、以下のトレードを行う。
- ５分足の平均足でトレードタイミングをつかむ。上位足と同じ色の線と逆の平均足が出るまで待ち、
  それから再び上位足のトレンド方向と一致した平均足が出たタイミングでトレンド方向にエントリー。
- エントリー後再び上位足のトレンド方向と逆の平均足が出た所で利確。

chatGPTに書かせたためどこまで再現できているかわからないが、以下のようになった。
```python
def trade_signal(d_df, h4_df, h1_df, m5_df):
    trade_entry = None
    trade_direction = None

    # Check if daily, 4-hour, and 1-hour candles are all bullish or bearish
    if d_df.iloc[-1]['close'] > d_df.iloc[-1]['open'] and h4_df.iloc[-1]['close'] > h4_df.iloc[-1]['open'] and h1_df.iloc[-1]['close'] > h1_df.iloc[-1]['open']:
        trade_direction = "buy"
    elif d_df.iloc[-1]['close'] < d_df.iloc[-1]['open'] and h4_df.iloc[-1]['close'] < h4_df.iloc[-1]['open'] and h1_df.iloc[-1]['close'] < h1_df.iloc[-1]['open']:
        trade_direction = "sell"

    # Calculate 5-minute moving average
    m5_df['MA'] = m5_df['close'].rolling(window=5).mean()

    # Entry signal
    if trade_direction:
        for i in range(len(m5_df) - 1, 4, -1):
            if trade_direction == "buy" and m5_df.iloc[i]['MA'] < m5_df.iloc[i-1]['MA'] and m5_df.iloc[i-1]['MA'] > m5_df.iloc[i-2]['MA']:
                trade_entry = "long"
                break
            elif trade_direction == "sell" and m5_df.iloc[i]['MA'] > m5_df.iloc[i-1]['MA'] and m5_df.iloc[i-1]['MA'] < m5_df.iloc[i-2]['MA']:
                trade_entry = "short"
                break

    return trade_entry, trade_direction


def backtest(trade_entry, trade_direction, m5_df):
    open_position = False
    entry_price = 0
    profit = 0
    trades = []

    for i in range(5, len(m5_df)):
        if not open_position:
            # Check for entry signal
            if trade_entry == "long" and m5_df.iloc[i]['MA'] > m5_df.iloc[i-1]['MA']:
                entry_price = m5_df.iloc[i]['open']
                open_position = True
                trades.append(("Entry", i, entry_price))
            elif trade_entry == "short" and m5_df.iloc[i]['MA'] < m5_df.iloc[i-1]['MA']:
                entry_price = m5_df.iloc[i]['open']
                open_position = True
                trades.append(("Entry", i, entry_price))

        else:
            # Check for exit signal
            if trade_direction == "buy" and m5_df.iloc[i]['MA'] < m5_df.iloc[i-1]['MA']:
                exit_price = m5_df.iloc[i]['open']
                profit += exit_price - entry_price
                open_position = False
                trades.append(("Exit", i, exit_price))
            elif trade_direction == "sell" and m5_df.iloc[i]['MA'] > m5_df.iloc[i-1]['MA']:
                exit_price = m5_df.iloc[i]['open']
                profit += entry_price - exit_price
                open_position = False
                trades.append(("Exit", i, exit_price))

    return profit, trades

trade_entry, trade_direction = trade_signal(d_df, h4_df, h1_df, m5_df)
profit, trades = backtest(trade_entry, trade_direction, m5_df)

print("Profit:", profit)
print("Trades:")
for trade in trades:
    print(trade)

mt5.shutdown()


def calculate_win_rate(trades):
    win_count = 0
    loss_count = 0
    total_trades = len(trades) // 2

    for i in range(0, len(trades) - 1, 2):
        entry_trade = trades[i]
        exit_trade = trades[i+1]

        if entry_trade[0] == "Entry" and exit_trade[0] == "Exit":
            if (trade_direction == "buy" and exit_trade[2] > entry_trade[2]) or (trade_direction == "sell" and exit_trade[2] < entry_trade[2]):
                win_count += 1
            else:
                loss_count += 1

    win_rate = win_count / total_trades * 100

    return win_rate

win_rate = calculate_win_rate(trades)
print("Win Rate: {:.2f}%".format(win_rate))
```
結果は以下のようになった。
![](https://Takaya-Shiraishi.github.io/ResearchReport/img/2023-05-01-09-19-38.png)

## 今後
今後は200MAと離れている時に、200MAがある位置に向かってトレンドフォローするというアルゴリズムも組み込んでみたい。