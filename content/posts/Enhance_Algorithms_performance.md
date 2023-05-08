---
title: "Enhance Algorithms Performance"
date: 2023-05-08T10:50:39+09:00
# bookComments: false
# bookSearchExclude: false
---
# 改善したアルゴリズム
記事「Algorithms_Note」で紹介したアルゴリズムを色々改善しました。
結果的に以下のようになりました。

## 平均足の算出方法
 - 平均足の始値
    ひとつ前の足の始値と終値の平均。
 - 平均足の終値
    現在の足の始値・終値・高値・安値の平均。
 - 平均足の高値
    現在の足の高値。ローソク足の高値と同じ。
 - 平均足の安値
    現在の足の安値。ローソク足の安値と同じ。

## マルチタイムフレーム分析
長期足：日足の平均足
中期足１：４時間足の平均足
中期足２：１時間足の平均足
短期足１：３０分足の平均足
短期足２：１５分足の平均足
エントリー足：５分足の平均足

## 条件
１．全ての時間足で平均足の陰線または陽線が合致している。
２．全ての時間足で平均足が陰線且つ200SMAより上ならば売り固定
３．全ての時間足で平均足が陽線且つ200SMAより下ならば買い固定

## エントリ・利益確定方法
５分足の平均足にて全ての時間足の平均足と同じ陰線または陽線が出たタイミングから
５分足の平均足を監視、トレンド方向と逆方向のひげが出なければその平均足でエントリー。
陽線・陰線が切り替わったタイミングで利益確定。

---

# 結果
## テスト対象期間
![](https://Takaya-Shiraishi.github.io/ResearchReport/img/2023-05-08-10-52-57.png)

## テスト結果
![](https://Takaya-Shiraishi.github.io/ResearchReport/img/2023-05-08-10-54-01.png)

## ソースコード
```python
import MetaTrader5 as mt5
import pandas as pd
import numpy as np
from datetime import datetime

# MT5に接続
if not mt5.initialize(login=口座番号, server="XMTrading-MT5", password="パスワード"):
    print("initialize() failed, error code =", mt5.last_error())
    mt5.shutdown()

# バックテスト期間を設定
start_date = datetime(2021, 5, 1)
end_date = datetime(2023, 4, 30)

# シンボルと時間足を設定
symbol = "USDJPY"
timeframes = [mt5.TIMEFRAME_D1, mt5.TIMEFRAME_H4, mt5.TIMEFRAME_H1, mt5.TIMEFRAME_M30, mt5.TIMEFRAME_M15, mt5.TIMEFRAME_M5]

# データを取得
rates = []
for tf in timeframes:
    rates.append(pd.DataFrame(mt5.copy_rates_range(symbol, tf, start_date, end_date)))

# 平均足を計算
def calc_heikin_ashi(df):
    ha_df = pd.DataFrame(index=df.index, columns=["open", "high", "low", "close"])
    ha_df["open"] = (df["open"].shift(1) + df["close"].shift(1)) / 2
    ha_df["close"] = (df["open"] + df["high"] + df["low"] + df["close"]) / 4
    ha_df["high"] = df["high"]
    ha_df["low"] = df["low"]
    return ha_df

heikin_ashi_rates = [calc_heikin_ashi(df) for df in rates]

# 200SMAを計算
sma200 = heikin_ashi_rates[-1]["close"].rolling(window=200).mean()

# エントリー判定
def should_enter(index, direction):
    if direction == "buy":
        for ha, df in zip(heikin_ashi_rates, rates):
            # 現在の5分足の時刻を取得
            current_time = rates[-1].iloc[index]['time']

            # 現在の5分足の時刻に最も近いデータポイントを他の時間足で探す
            closest_index = df.index[df['time'] <= current_time].max()

            if ha["close"].iloc[closest_index] <= ha["open"].iloc[closest_index]:
                return False
        if heikin_ashi_rates[-1]["close"].iloc[index] <= sma200.iloc[index]:
            return False
        return True
    elif direction == "sell":
        for ha, df in zip(heikin_ashi_rates, rates):
            # 現在の5分足の時刻を取得
            current_time = rates[-1].iloc[index]['time']

            # 現在の5分足の時刻に最も近いデータポイントを他の時間足で探す
            closest_index = df.index[df['time'] <= current_time].max()

            if ha["close"].iloc[closest_index] >= ha["open"].iloc[closest_index]:
                return False
        if heikin_ashi_rates[-1]["close"].iloc[index + 1] >= sma200.iloc[index + 1]:
            return False
        return True
    return False

# バックテストの実行
entry_price = None
position = None
initial_balance = 10000
balance = initial_balance
total_trades = 0
winning_trades = 0

lot_size = 0.01  # 1,000通貨単位
pip_value = 0.01  # 1pipあたりの価値（USDJPYの場合）

# バックテストの実行
for index in range(200, len(rates[-1]) - 1):
    if position is None:
        if should_enter(index, "buy"):
            position = "buy"
            entry_price = heikin_ashi_rates[-1]["close"].iloc[index + 1]
        elif should_enter(index, "sell"):
            position = "sell"
            entry_price = heikin_ashi_rates[-1]["close"].iloc[index + 1]
    else:
        ha_current = heikin_ashi_rates[-1].iloc[index]
        ha_next = heikin_ashi_rates[-1].iloc[index + 1]

        if position == "buy" and ha_current["close"] > ha_current["open"] and ha_next["close"] <= ha_next["open"]:
            profit_pips = (ha_next["open"] - entry_price) / pip_value
            profit = profit_pips * lot_size * 100
            balance += profit
            total_trades += 1
            if profit > 0:
                winning_trades += 1
            position = None
        elif position == "sell" and ha_current["close"] < ha_current["open"] and ha_next["close"] >= ha_next["open"]:
            profit_pips = (entry_price - ha_next["open"]) / pip_value
            profit = profit_pips * lot_size * 100
            balance += profit
            total_trades += 1
            if profit > 0:
                winning_trades += 1
            position = None

win_rate = winning_trades / total_trades * 100
print(f"Initial balance: {initial_balance}")
print(f"Final balance: {balance:.2f}")
print(f"Total trades: {total_trades}")
print(f"Winning trades: {winning_trades}")
print(f"Win rate: {win_rate:.2f}%")

mt5.shutdown()


```