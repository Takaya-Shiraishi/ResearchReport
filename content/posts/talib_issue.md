---
title: "Talib Issue"
date: 2023-05-01T07:57:49+09:00
image: "2023-05-01-08-09-25.png"
# bookComments: false
# bookSearchExclude: false
---
# TA-Libのインストール問題
pypiの環境だとta-libは所謂
```pip install ta-lib```
だとインストールがうまくいかない。

## 原因推測
どうも`concorde`と同じでC言語版をpython向けにビルドするらしく、concordeと似たようなエラーが出る。

## ワークアラウンド
しかしta-libにはwhlファイルをインストール対象とすることでこのエラーを回避できる。
[https://www.lfd.uci.edu/~gohlke/pythonlibs/#ta-lib](https://www.lfd.uci.edu/~gohlke/pythonlibs/#ta-lib)から
実行したいpythonのバージョンに合わせたwhlファイルをダウンロードして、
```pip install ダウンロードしたファイル名```でインストールする。
たとえば、python3.9を使っている場合は
```pip install TA_Lib-0.4.24-cp39-cp39-win_amd64.whl```
という具合になる。

## その後
普通に`import talib`できるようになるはず。
余談だけどconda環境だと`conda install -c conda-forge ta-lib`で普通にインストールできた。