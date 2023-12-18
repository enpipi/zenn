---
title: "Notionのデータベースプロパティで更新日を管理する"
emoji: "🤖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["notion"]
published: false
---

## 背景

サービスの契約更新日などを把握しておきたいが、更新日を毎回入力するのが手間だったのでデータベースの関数を酷使しました。

:::message
関数はオートメーショントリガーに使えず、Zapierでも関数は拾えません。リマインドとかカレンダーにも予定として入れたい！ということであればNotionのdatabase APIを使う必要があります。
:::

## できたもの
今日の日付が契約日を過ぎていた場合に年次は一年後、月次は一ヶ月後を算出する。

![notion-database-renewal-date](/images/notion-automatic-renewal-date/notion-database-renewal-date.jpg)


## できること

- 今日を起算日として次回更新日を自動で算出する
- 年次・月次を判定する

## 手順

### 適当なデータベースを用意して、プロパティに契約日を用意する

更新日の基準値としてなんでも良いので日付を用意します。ここではプロパティ名を契約日とします。

### プロパティに年次・月次を追加

セレクトを使って年次・月次を追加。ここではプロパティ名は契約期間とします。

### 関数を使って次回更新日を算出

関数として以下を入力。プロパティ名は更新日とします。

```js:notion-formula
if(
    prop("契約期間") == "年次",
    if(
        0 > toNumber(formatDate(prop("契約日"), "MMDD")) - toNumber(formatDate(now(), "MMDD")),
        (year(now()) + 1 + "-" + formatDate(prop("契約日"), "MM-DD")).replaceAll("-", "").parseDate(),
        (year(now()) + "-" + formatDate(prop("契約日"), "MM-DD")).replaceAll("-", "").parseDate()
    ),
    if(
        0 > toNumber(formatDate(prop("契約日"), "DD")) - toNumber(formatDate(now(), "DD")),
        dateAdd(
            (year(now()) + "-" + month(now()) + "-" + formatDate(prop("契約日"), "DD")).replaceAll("-", "").parseDate(),
            1,
            "months"
        ),
        (year(now()) + "-" + month(now()) + "-" + formatDate(prop("契約日"), "DD")).replaceAll("-", "").parseDate()
    )
)
```

## ロジック

上のロジックを読み解くのは面倒ですね。
chatGPT 先生に言語化してもらいました。

> 1.  最初に、`if` 関数で「契約期間」が「年次」かどうかを判断します。
> 2.  「契約期間」が「年次」の場合:
>       - 契約日と現在の日付を比較し、結果が0より大きい（契約日が本日より前）なら、現在の年と契約日を結合して更新日を設定します。
>      - 結果が0より小さい（契約日が本日より後）場合、現在の年に1足したものと契約日を結合して更新日を設定します。
>
> 3.  「契約期間」が「年次」以外（たとえば「月次」）の場合:
>       - 契約日と現在の日付を比較し、結果が0より大きい（契約日が本日より前）なら、現在の年と月、契約日を結合して更新日を設定します。
>       - 結果が0より小さい（契約日が本日より後）場合、現在の年と月、契約日を結合した日付に1か月を足して更新日を設定します。
>
> この関数中の `replaceAll("-","").parseDate()` は、日付形式を整形し、正常な日付として Notion に認識させるために使用しています。また、`formatDate()` 関数は、日付を特定のフォーマットに変換するために使用されています

## [オプション] フィルター、ソートをかける
![notion-filter-sample](/images/notion-automatic-renewal-date/notion-filter-sample.jpg)

最後に表示するサービスを直近のものだけに絞る、更新日順にソートなどして完成です。

## おわりに

関数酷使は大変だ。手段が目的にならないようにしたいですが、notion 芸人するのは楽しいですね。

次回はNotion database APIを使って通知をするgasでも作ろうと思います。