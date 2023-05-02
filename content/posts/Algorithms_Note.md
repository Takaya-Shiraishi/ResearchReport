---
title: "Algorithms Note"
date: 2023-05-03T02:34:49+09:00
# bookComments: false
# bookSearchExclude: false
---
# トレードアルゴリズムの整理用ノート
不定期で内容変わります。

## 大枠
- マルチタイムフレーム分析でトレンドを把握
- 平均足を使ってエントリーポイントを探す
- トレンドフォロー型
- 利益確定は平均足の陰線か陽線かが切り替わるタイミングで行う
- 上位足の動きに逆らうトレードは絶対にしない。
  むしろ上位足の分析は売り/買い目線の固定が目的。

---

## 平均足
### 平均足の算出方法
- 平均足の始値
  ひとつ前の足の始値と終値の平均。
- 平均足の終値
  現在の足の始値・終値・高値・安値の平均。
- 平均足の高値
  現在の足の高値。ローソク足の高値と同じ。
- 平均足の安値
  現在の足の安値。ローソク足の安値と同じ。

### 平均足の特性（ひげと実体について）
- トレンドが強い場合、その方向と逆のひげがなくなる。
- 実体が小さく上下にひげがある場合、トレンド転換期であるため
  エントリーは危険。

## マルチタイムフレーム分析
**以下は日足、４時間足、１時間足の全てに適用する。**
- ZigZag
  この３つの時間足のZigZagの方向が一致している場合は
  トレンドが確定していると判断する。
- 200SMA
  200SMAからの乖離が大きい場合は200MAに戻ると判断する。
  ZigZagの方向と現在の足から見た200MAの方向が一致している
  場合にのみトレードを行う。
- 平均足
  平均足の方向がZigZagの方向と一致しており、平均足の特性から
  今後も下降・上昇が続くと判断される場合にのみトレードを行う。

---

## エントリーの方法
**５分足の平均足でのみ以下のエントリー・利益確定を行う。**
1. 上位足のMTF分析でトレンド方向が決定し判断条件もクリアした場合、
   トレンドと反対の陰線/陽線が出るまで待機する。
2. その後トレンドと反対の陰線/陽線が切り替わりトレンドと同じ陰線/陽線が出た
   場合、その足の次の足が前の足と同じ陽線/陰線なら始値でエントリーする。
   ただし平均足の特性によりトレンド転換期と判断される場合は
   エントリーを行わない。
3. エントリー後平均足の陰線/陽線で切り替わるタイミングで利益を確定する。

---

# テンプレート？
一応平均足を中心としたテンプレート的なものがあったので。
## MTF分析
|エントリー足|中期足|長期足|
|:-:|:-:|:-:|
|１５分足|１時間足|日足|

## 条件
- 長期足・中期足
  両足の平均足の陽線または陰線が合致している。
- エントリー足
  長期足・中期足の合致している方向にエントリー足の平均足が
  変わったタイミングでエントリーする。
  - 例
    > 長期・中期足が陽線であれば、エントリー足の平均足が
    > 陰線の常体から陽線へ切り替わった時。
- 利益確定
  陽線・陰線が切り替わったタイミングで利益確定する。

## でもテンプレート通りだと50%ちょっとしか勝てなかったので
![](img/2023-05-03-05-05-31.png)
こうしました。
すると、
![](img/2023-05-03-05-06-02.png)
こうなりました。

### ソースコード
```python
import MetaTrader5 as mt5
import pandas as pd
import numpy as np
from datetime import datetime, timezone

# Initialize MT5 connection
if not mt5.initialize():
    print("initialize() failed, error code =", mt5.last_error())
    quit()

# Set the symbol and timeframes
symbol = "GBPJPY"
timeframes = [mt5.TIMEFRAME_M5, mt5.TIMEFRAME_H1, mt5.TIMEFRAME_H4, mt5.TIMEFRAME_D1]
date_from = datetime(2022,1,1, tzinfo=timezone.utc)
date_to = datetime(2023,4,30, tzinfo=timezone.utc)
# Retrieve historical data for each timeframe
data = {}
for timeframe in timeframes:
    rates = mt5.copy_rates_range(symbol, timeframe, date_from, date_to)
    df = pd.DataFrame(rates)
    df['time'] = pd.to_datetime(df['time'], unit='s')
    data[timeframe] = df

# Calculate Heikin-Ashi (平均足) for each timeframe
def heikin_ashi(df):
    ha_df = pd.DataFrame(index=df.index, columns=df.columns)
    ha_df['time'] = df['time']
    ha_df['open'] = (df['open'].shift(1) + df['close'].shift(1)) / 2
    ha_df['close'] = (df['open'] + df['high'] + df['low'] + df['close']) / 4
    ha_df['high'] = df['high']
    ha_df['low'] = df['low']
    return ha_df

for timeframe in timeframes:
    data[timeframe] = heikin_ashi(data[timeframe])

# Check if all timeframes are bullish or bearish
def check_alignment(ha_data, timeframes):
    alignment = None
    for timeframe in timeframes[:-1]:
        current_candle = ha_data[timeframe].iloc[-1]
        if current_candle['open'] < current_candle['close']:
            if alignment is None:
                alignment = "bullish"
            elif alignment != "bullish":
                return None
        else:
            if alignment is None:
                alignment = "bearish"
            elif alignment != "bearish":
                return None
    return alignment

initial_balance = 100000
    
def backtest(data, timeframes):
    balance = initial_balance
    position = None
    entry_price = None
    win_count = 0
    loss_count = 0

    entry_data = data[timeframes[0]]
    for index, row in entry_data.iterrows():
        alignment = check_alignment(data, timeframes)
        if alignment is None:
            position = None
            continue

        # Check for entry signal
        if position is None and (row['open'] < row['close']) == (alignment == "bullish"):
            position = "long" if alignment == "bullish" else "short"
            entry_price = row['close']
            #print(f"{row['time']} - Entered {position} at {entry_price}")

        # Check for exit signal
        if position is not None and (row['open'] < row['close']) != (alignment == "bullish"):
            exit_price = row['close']
            if position == "long":
                profit = exit_price - entry_price
            else:
                profit = entry_price - exit_price
            balance += profit * 100000  # Assuming a trade volume of 100,000 units

            # Update win/loss counts
            if profit > 0:
                win_count += 1
            else:
                loss_count += 1

            #print(f"{row['time']} - Exited {position} at {exit_price}, profit: {profit:.5f}, balance: {balance:.2f}")
            position = None

    # Calculate win rate
    win_rate = win_count / (win_count + loss_count) * 100

    return balance, win_rate

# Run the backtest
final_balance, win_rate = backtest(data, timeframes)
print(f"Initial balance: {initial_balance:.2f}")
print(f"Final balance: {final_balance:.2f}")
print(f"Win rate: {win_rate:.2f}%")

# Shutdown MT5 connection
mt5.shutdown()
```