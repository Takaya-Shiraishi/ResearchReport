---
title: "Small Difference"
date: 2023-05-22T11:01:24+09:00
# bookComments: false
# bookSearchExclude: false
---
# アルゴリズムの違い
アルゴリズムの小さな違いが結果に大きな影響を齎していることを発見しました。

## 今回検証したトレードアルゴリズム
```bash
Heikin_Ashi_D1 = GET(日足の平均足)
Heikin_Ashi_H4 = GET(４時間足の平均足)
Heikin_Ashi_H1 = GET(１時間足の平均足)
Heikin_Ashi_M30 = GET(３０分足の平均足)
Heikin_Ashi_M15 = GET(１５分足の平均足)

IF(Heikin_Ashi_D1.now == 陽線 AND Heikin_Ashi_D1.now(下ひげ) == なし):
    IF(Heikin_Ashi_H4.now == 陽線 AND Heikin_Ashi_H4.now(下ひげ) == なし):
        IF(Heikin_Ashi_H1.now == 陽線 AND Heikin_Ashi_H1.now(下ひげ) == なし):
            IF(Heikin_Ashi_M30.now == 陽線 AND Heikin_Ashi_M30.now(下ひげ) == なし):
                IF(Heikin_Ashi_M15.now == 陽線 AND Heikin_Ashi_M15.now(下ひげ) == なし):
                    Entry_Position = BUY(Heikin_Ashi_M15.now)
                    
ELSE IF(Heikin_Ashi_D1.now == 陰線 AND Heikin_Ashi_D1.now(上ひげ) == なし):
    IF(Heikin_Ashi_H4.now == 陰線 AND Heikin_Ashi_H4.now(上ひげ) == なし):
        IF(Heikin_Ashi_H1.now == 陰線 AND Heikin_Ashi_H1.now(上ひげ) == なし):
            IF(Heikin_Ashi_M30.now == 陰線 AND Heikin_Ashi_M30.now(上ひげ) == なし):
                IF(Heikin_Ashi_M15.now == 陰線 AND Heikin_Ashi_M15.now(上ひげ) == なし):
                    Entry_Position = SELL(Heikin_Ashi_M15.now)

IF(COLOR(Entry_Position) != COLOR(Heikin_Ashi_M15.now)):
    CLOSE(Entry_Position)
```

## 前回勝率が高かった（winrateは80%超え）アルゴリズムとの違い
- ＭＴＦ分析の基本は同じ
- 最終的な判断が違いました。
  ５分足にて５ＭＡを計算し、その５ＭＡより上であれば買い、下であれば売りというアルゴリズムでした。
- 今回は、単にMTF分析の結果に従って順張りするだけのアルゴリズムです。

## 検証結果
今回のアルゴリズムの方が、前回のアルゴリズムよりも勝率が低いことが分かりました。
![](https://Takaya-Shiraishi.github.io/ResearchReport/img/2023-05-22-11-08-38.png)

## 今回の検証で得られた知見
シンプルなトレード手法は例外が発生しやすくなることを意味するため、裁量で行うことが最適。
一方移動平均線は例外や異常を中和したり無視するためのキーファクターとなりそう。

## 今後
１５分足で5MAを計算し、その5MAより上であれば買い、下であれば売りというアルゴリズムを検証する。

## 付録
今回のトレードアルゴリズムのバックテスト用ソースコードは以下の通りです。

```python
import MetaTrader5 as mt5
import pandas as pd

# MT5に接続
if not mt5.initialize(login=数字, server="XMTrading-MT5", password="パスワード"):
    print("initialize() failed, error code =",mt5.last_error())
    mt5.shutdown()

# バックテスト期間の設定
start_date = pd.to_datetime('2021-05-01').to_pydatetime()
end_date = pd.to_datetime('2023-04-30').to_pydatetime()

# 指定した期間でのデータを取得（この例ではEURUSD）
rates = mt5.copy_rates_range("EURUSD", mt5.TIMEFRAME_M15, start_date, end_date)

# データフレームに変換
df = pd.DataFrame(rates)

# タイムスタンプをdatetime形式に変換
df['time']=pd.to_datetime(df['time'], unit='s')

# 初期設定
initial_balance = 10000
balance = initial_balance
total_trades = 0
winning_trades = 0

# 平均足の計算
df['avg_open'] = (df['open'].shift() + df['close'].shift()) / 2
df['avg_close'] = (df['open'] + df['close'] + df['high'] + df['low']) / 4
df['avg_high'] = df['high']
df['avg_low'] = df['low']

# エントリーとイグジットの条件
for i in range(1, len(df)):
    
    # 陰線（売りエントリー）
    if df['avg_open'].iloc[i] > df['avg_close'].iloc[i] and df['avg_high'].iloc[i] == df['avg_open'].iloc[i]:
        total_trades += 1
        # エントリー価格（始値）
        entry_price = df['avg_open'].iloc[i+1]
        # イグジット条件を満たすまでループ
        for j in range(i+1, len(df)):
            if df['avg_open'].iloc[j] < df['avg_close'].iloc[j]:
                # イグジット価格（始値）
                exit_price = df['avg_open'].iloc[j]
                # プロフィット計算
                profit = entry_price - exit_price
                balance += profit
                if profit > 0:
                    winning_trades += 1
                break
    
    # 陽線（買いエントリー）
    if df['avg_open'].iloc[i] < df['avg_close'].iloc[i] and df['avg_low'].iloc[i] == df['avg_close'].iloc[i]:
        total_trades += 1
        # エントリー価格（始値）
        entry_price = df['avg_open'].iloc[i+1]
        # イグジット条件を満たすまでループ
        for j in range(i+1, len(df)):
            if df['avg_open'].iloc[j] > df['avg_close'].iloc[j]:
                # イグジット価格（始値）
                exit_price = df['avg_open'].iloc[j]
                #プロフィット計算
                profit = exit_price - entry_price
                balance += profit
                if profit > 0:
                    winning_trades += 1
                break

# バックテスト結果
win_rate = (winning_trades / total_trades) * 100

print(f"Initial balance: {initial_balance}")
print(f"Final balance: {balance:.2f}")
print(f"Total trades: {total_trades}")
print(f"Winning trades: {winning_trades}")
print(f"Win rate: {win_rate:.2f}%")

# MT5をシャットダウン
mt5.shutdown()
```