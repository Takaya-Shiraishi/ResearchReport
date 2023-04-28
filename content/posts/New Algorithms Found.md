---
title: "New Algorithms Found"
date: 2023-04-28T16:37:14+09:00
# bookComments: false
# bookSearchExclude: false
---

# New Algorithms Found
新しいアルゴリズムを発見しました。

## 平均足
平均足とは、以下の数式で表されます。

【平均足の計算方法】

{{< katex >}}
Open: \frac{Open_{t-1} + Close_{t-1}}{2}
\\\\
High: max(High_t, Open_t, Close_t)
\\\\
Low: min(Low_t, Open_t, Close_t)
\\\\
Close: \frac{Open_t + High_t + Low_t + Close_t}{4}
\\\\
\text{平均足始値} = \frac{\text{前日平均足始値} + \text{前日平均足終値}}{2}
\\\\
\text{平均足終値} = \frac{\text{当日始値} + \text{当日高値} + \text{当日安値} + \text{当日終値}}{4}
{{</ katex >}}