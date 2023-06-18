---
title: "Mid Report"
date: 2023-06-19T03:24:56+09:00
# bookComments: false
# bookSearchExclude: false
---

# 前期中間課題レポート
まず、先行研究の調査対象として、
- https://scholar.google.co.jp/schhp?hl=ja
- https://cir.nii.ac.jp
- https://www.jstage.jst.go.jp/browse/-char/ja/
を対象とした。そして、効率的な検索を目的とするために
スクレイピングが可能か`robots.txt`を確認した。
- https://cir.nii.ac.jp/robots.txt
```bash
User-agent: bingbot
Crawl-Delay: 1
Disallow: /
Allow: /$
Allow: /crid/
Allow: /sitemaps/
Allow: /resourcesync/
Allow: /.well-known/resourcesync$
Disallow: /ja$
Disallow: /en$
Disallow: /crid/*.bix$
Disallow: /crid/*.ris$
Disallow: /crid/*.bib$
Disallow: /crid/*.tsv$
Disallow: /crid/*/refworks$
Disallow: /crid/*/endnote$
Disallow: /crid/*/mendeley$
Disallow: /feedback/
Sitemap: https://cir.nii.ac.jp/sitemaps/sitemaps_top.xml
Sitemap: https://cir.nii.ac.jp/resourcesync/resourcelist-index.xml

User-agent: *
Disallow: /
Allow: /$
Allow: /crid/
Allow: /sitemaps/
Allow: /resourcesync/
Allow: /.well-known/resourcesync$
Disallow: /ja$
Disallow: /en$
Disallow: /crid/*.bix$
Disallow: /crid/*.ris$
Disallow: /crid/*.bib$
Disallow: /crid/*.tsv$
Disallow: /crid/*/refworks$
Disallow: /crid/*/endnote$
Disallow: /crid/*/mendeley$
Disallow: /feedback/
Sitemap: https://cir.nii.ac.jp/sitemaps/sitemaps_top.xml
Sitemap: https://cir.nii.ac.jp/resourcesync/resourcelist-index.xml
```
- https://scholar.google.co.jp/robots.txt
```bash
User-agent: *
Disallow: /search
Disallow: /index.html
Disallow: /scholar
Disallow: /citations?
Allow: /citations?user=
Disallow: /citations?*cstart=
Disallow: /citations?user=*%40
Disallow: /citations?user=*@
Allow: /citations?view_op=list_classic_articles
Allow: /citations?view_op=metrics_intro
Allow: /citations?view_op=new_profile
Allow: /citations?view_op=sitemap
Allow: /citations?view_op=top_venues

User-agent: Twitterbot
Disallow:

User-agent: facebookexternalhit
Disallow:

User-agent: PetalBot
Disallow: /
```
- https://www.jstage.jst.go.jp/robots.txt
```bash
User-agent: *
Disallow: /*_pdf

User-agent: Googlebot
Allow: /

User-agent: TurnitinBot
Allow: /

User-agent: NBDCbot(biosciencedbc.jp/dbsearch)
Allow: /

User-agent: ciniibot/1.0
Allow: /

User-agent: ASCPro16
Allow: /

User-agent: Summon
Allow: /

User-agent: bingbot
Crawl-delay: 5
```

ということで、スクレイピングはしない方が良いと判断した。
この様子ではseleniumを用いた自動ブラウザ検索も難しいと考えられる。
CiNii Books APIを用いて検索する事は可能だと考えられるが、
それも検索ワードによって大量アクセスを招いてしまう可能性があるため、
APIの使用も行わないことにした。
そこで、Elicitという論文AI検索ツールを使って横断検索を行うことにした。

## 検索実行・先行研究調査
ChatGPTベースの検索ツールなので、以下のように入力した。
```
Are there any papers using "Heikin-Ashi" in FX research?
```

すると、以下のような結果が得られた。
```
None of the papers collected address the research question of whether "Heikin-Ashi" is used in FX research. Instead, the papers focus on other topics such as the use of hedging in research articles in applied linguistics (Livytska 2019, Hashemi 2016, Orta 2008) and the design of exhibitions (Wang 2019).
```
平均足の英語スペルはMT5の標準搭載インジケーター「Heikin_Ashi」の名前から引用した。
なお、`-`を`_`に置き換えても検索結果は同じだった。
この結果から、平均足に関する論文はないということがわかった。
以下はCiNii Research、Google Scholar、J-STAGEの検索結果である。
- CiNii Research
  - 検索ワード: 平均足、Heikin_Ashi
  - 検索結果: 0件
- Google Scholar
  - 検索ワード: Heikin_Ashi
  - 検索結果: 96件
  - 検索詳細（白石の研究と関連性の高いもののみ抜粋）
    - [SMOOTHED HEIKIN-ASHI ALGORITHMS OPTIMIZED FOR
AUTOMATED TRADING SYSTEMS](https://www.itema-conference.com/wp-content/uploads/2019/09/pauna_smoothed_heikin-ashi_algorithms_optimized_for_automated_trading_systems_pp_514-525.pdf)
      - 翻訳結果:
        - タイトル
        > 自動取引システムに最適化された平滑化平均足アルゴリズム

        - 概要翻訳
        > このPDFファイルは、自動取引システムに最適化されたスムーズな平均足アルゴリズムについての情報を提供しています。平均足（Heikin-Ashi）とは、金融市場でトレンドを特定し追跡するための手法の一つであり、ノイズを除去し価格のトレンドをグラフィカルに表現するための新しい系列に価格の時系列を変換します。この手法は、高頻度取引においてトレンドを特定し追跡するための優れた方法として知られています。本論文では、平均足の手法を現代の制限条件と組み合わせることで、取引効率を向上させ、取引の自動化を実現する方法を紹介します。平均足に基づく取引シグナルの自動生成と使用方法、および同じ手法を使用して出口シグナルを自動化する方法も説明されます。また、最後の部分では、提案された手法を他の高頻度取引の戦略と比較するために、取引結果も提示されます。具体的には、フランクフルト証券取引所のDeutscher Aktienindex市場で得られた取引結果が示されます。

        - 本文抜粋（結果部分）
        > All trading and the exit conditions included in the presented methodology are based on the Heikin-Ashi price transformation and can constitute a stand-alone trading model for automated trading systems. Being exclusively a mathematical model, the methodology presented in this paper can be applied with good results for algorithmic trading and high-frequency trading. This model can also be used for manual trading.
      
    - [Using The Heikin-Ashi Technique](https://d1wqtxts1xzle7.cloudfront.net/36795921/Using_The_Heikin_Ashi_Technique_D_Valcu-libre.pdf?1425059227=&response-content-disposition=inline%3B+filename%3DUsing_The_Heikin_Ashi_Technique_D_Valcu.pdf&Expires=1687119611&Signature=fRbfrHVIV8LPGWiXATWSK~TNPjRHmlf72IO4xQsMIrm7wFkN-RrEKD5su2dIQFQ~gTwXjrA3oK5b1ymHQhZrdsKupfNqKYaF-5YxItOFVesg8Ug0l2mFkwkuUNsaEv4Hr4xOpNExZ5OBJYenMN5kzi1ALsk8H4NFVsg10icZsDo7bX3DItdYN~9eQTgeWlMkSeoMlylvJOOm3JFFxB-yvQNTHpCvXszaJ2MsFBQmRpxsQX52R890pWCdlY8KMkLHHM-QdWCZoKLhVxNL3wqifUerzAhQ9N1Fli13UQcBnrgYVX1BVZeKU~6Grr4Io69~aeYurSOPXfjW0hb8d~e~8Q__&Key-Pair-Id=APKAJLOHF5GGSLRBV4ZA)
      - 翻訳結果：
         - タイトル
         > 平均足テクニックの使用

         - 概要翻訳
         > このPDFファイルは、Heikin Ashiテクニックを使用したトレードテクニックについて説明しています。Heikin Ashi法は、通常のチャートから不規則性を除去した視覚的な表現を提供し、市場のトレンドや統合をより良く理解することができます。この技術を使用することで、ローソク足チャートを簡単に解釈し、情報に基づいたトレードの決定を行うことができます。Heikin Ashiテクニックの利点や、他のテクニカル指標との組み合わせ方、リスク管理の重要性についても説明されています。ただし、具体的な実装手順については提供されていません。

         - 本文抜粋（結果部分）
         > The main advantage of this simple method is a better visual perspective of the current status and strength of the trend or consolidation, and a possible anticipation of the next bar's strength. As with any other charting method, the heikin-ashi is not 100% reliable and therefore should be combined with other technical indicators. Your trading, of course, should also include risk- and capital-control strategies.

    - [Heikin-Ashi Technique with Use of Oriented Fuzzy Numbers](https://link.springer.com/chapter/10.1007/978-3-030-95929-6_5)
      - 翻訳結果：
         - タイトル
         > 指向性ファジィ数を用いた平均足手法
 
         - 概要翻訳
         > ローソク足とは、為替相場の変動を表すチャートの一種である。ローソク足の代表的なものは、ローソク足に基づく和製ローソク足であり、相場の変動を表現しています。また、金融市場の価格時系列を利用してトレンドを把握する手法の一つとして、平均足の手法が知られています。手法としては、通常のローソク足が変形したものである。また、ローソク足は、過去と現在の相場変動を表現しています。この点は、平均足チャートと比較した場合の大きな利点である。一方、平均足は指向性ファジィ数で記述することができる。本論文の主な目的は、指向性ファジィ数で記述された平均足チャートの変換として、任意の平均足を提示することである。さらに、平均足チャートの不正確さに対する平均足の影響を研究する。すべての考察は、数値的なケーススタディによって説明される。 
         - 本文抜粋（結果部分）
         > 本文は有償閲覧限定のため無し

- J-STAGE
  - 検索ワード：平均足、Heikin_Ashi
  - 検索結果：0件

### 平均足、Heikin_Ashiを検索ワードにした理由
私の研究の中核を成すインジゲータであり、かつマイナーであるためほかのワードで検索するよりも関連度の高い研究を上位に表示する事が可能であると判断した。

---
**ここからは読んでいただくことを意識するためですます調になります。**

## これまでに調べたこと
これを書くのは本来上述の内容の前ですが、上述の内容を踏まえたうえで書きたい事があった為この位置に書く事となりました。
その書きたかった事とは、FXの勝率に直結する研究は論文という形態を取りにくいのではないかという仮説です。
例えば、平均足についてGoogleで調べるとどういったものかは出てきます。しかしそれは、論文という形態ではなく銀行や証券会社のホームページという形態で出てくることが多かったです。
どの論文を見ても、平均足自体に明確なリファレンスがあるのではなく計算式のみが一般化されているという印象がありました。現行で存在する閲覧可能な全ての論文を読んだわけではないので、あくまで印象という形になってしまいますが・・・。
そして、検索して出てくる論文に関して一貫性があることとして、平均足のみ、或いは数学や機械学習等の分野に対するコラボレーションのみを取り上げており、例えば他のインジゲータとの組み合わせ等に関しては積極的に取り上げられていないという点がありました。

## 白石の研究と先行研究の違い
上述を考えると、自分の研究は間違えなく他のインジゲータとのコラボレーションであり、マルチタイムフレーム分析にも積極的な研究であるため、より勝率に直結する研究であると言えます。
また、金融と他分野のコラボではなく、金融、しかもチャート分析の中の分野（平均足：プライスアクション系、マルチタイムフレーム分析：ファンダメンタルズ系、他インジゲータによる分析：テクニカル系）でのコラボレーションであるため、よりFXやチャート分析に対して特化していると言えます。

## 自分の研究の現状
現在は以下の手法で続けています。
1. アルゴリズムを考える
2. ChatGPTにバックテストコードを書いてもらう
3. 結果から原因を考えつつコードを精査・評価
4. 改善案を思いついたら２に戻る。思いつかなかったら別スレッドのChatGPTに改善案を提案させて参考にする。改善案が思いつかなかったら他の事をする。

そして、状況としてはエントリ―軸にする時間足を調整している段階にあります。
具体的には、時間足が長くなれば一定期間あたりに生成される時間足の本数自体は減るため、エントリ―の条件を厳しくした場合は更にエントリーの機会が減りってしまうこととなります。
そのため、４月～５月上旬には短い時間足でエントリ―の条件を厳しくした場合の手法について模索してきました。しかしこの結果、エントリーの機会自体は増えるもののエントリーの条件によってソーティングできない例外的なノイズがあることを発見しました。
そのため、５月中旬からは時間足を長くしてエントリ―の条件を緩くすることでエントリーの機会を増やすことを模索しています。
また、ＭＴＦ分析で扱う各時間足に対して適応するインジゲータを調整する事でもこの問題の解決が出来るのではないかと考え、今はこの二方面から研究を進めています。

## 自分の研究の今後のスケジュール
就活と教職課程がどうなるかわからないので不明です。
以下はあくまで予定となります。

|6月|7月|8月|9月|10月|11月|12月|1月|2月|
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|エントリ―足と<br>エントリ―条件の<br>検証|エントリ―足と<br>エントリ―条件の<br>検証|エントリ―足と<br>エントリ―条件の<br>検証・決定・執筆開始|エントリ―足と<br>エントリ―条件の<br>検証・決定・執筆|エントリ―足と<br>エントリ―条件の<br>検証・決定・執筆|エントリ―足と<br>エントリ―条件の<br>検証・決定・執筆|論文推敲|猶予期間|猶予期間|

- エントリー足
  - ４時間足、１時間足、３０分足のどれかにしようと考えています。
- エントリー条件
  - 平均足含め、MTF分析で使うインジゲータの確定・調整も含まれます。