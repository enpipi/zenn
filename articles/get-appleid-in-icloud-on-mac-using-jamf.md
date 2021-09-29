---
title: "Jamfを使ってiCloudに設定しているAppleIDを取得する"
emoji: "💬"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

# はじめに
iCloudに設定されているAppleIDの管理、みなさんどうされていますか？
運用ではプライベートのAppleIDを禁止していても、制御するのは難しいですよね。

ここではiCloudに設定したAppleIDをJamf Proを使って取得する方法を記します。

## 出オチ
他の方のスクリプトを参考に書いたものの、その後により良いスクリプトを見つけてしまいました。
皆さんはこちらのスクリプトを参考にしましょう。
https://github.com/bp88/Jamf-Pro-Extension-Attributes/blob/master/iCloud%20Account%20Details.sh

:::notes
というわけで以下内容は、私の習学のアウトプットおよびJamfでの拡張属性の使い方となりますのでご容赦ください。
iCloudに設定しているAppleIDをJamfで取得できる方法があるということだけでも知ってもらえたら幸いです。
:::

### iCloudに設定しているAppleIDを取得するスクリプト

```
#!/bin/sh

icloud=$(/usr/libexec/PlistBuddy -c "print" ~/Library/Preferences/MobileMeAccounts.plist | grep AccountID | sed 's/^[ \t]*AccountID = //g')

echo "<result>$icloud</result>"
```

### Jamfで拡張属性を設定する

1. 設定 > コンピュータ管理 > 拡張属性 > 新規
2. 適当な名前を付け、データタイプを `String`, インベントリ表示を `General`, 入力タイプを `Script` にして上記スクリプトを貼り付け保存する
3. (任意) 設定 > コンピュータ管理 > インベントリ表示 > Extention Attributes に2で作った拡張属性にチェックを入れる

:::notes
インベントリ表示でチェックを入れるとコンピュータ検索時の一覧に表示されるようになります
:::



## 謝辞
本スクリプトを書くにあたり Relaさんの記事を参考にしました。ありがとうございます。
https://rela1470.hatenablog.jp/entry/2021/08/03/000000

本記事でiCloudの管理統制に少しでも役に立てたら幸いです。
