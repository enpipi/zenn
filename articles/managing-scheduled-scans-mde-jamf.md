---
title: "Jamf ProのMDE用CustomSchemaを修正して定期スキャンに対応させた"
emoji: "🕒"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [JamfPro,Security,Microsoft,Tech]
published: true
publication_name: "visasq"
---


## はじめに
Microsoft Defender for Endpoint（MDE）は、MDMで設定を管理でき、Jamf Pro向けにCustomSchemaを提供しています。Microsoftは、このCustomSchemaを使ったJamf ProでのMDE導入方法を公開しています。多くの企業がこのCustomSchemaを利用しているのではないでしょうか。

この記事では、MDEの定期スキャンに対応したCustomSchemaと利用手順を紹介します。

## 問題の背景

以下ドキュメントを参考にMDEの定期スキャンを構築しましたが、スケジュールした時刻にスキャンが行われず、期待通りに動作しませんでした。
https://learn.microsoft.com/ja-jp/defender-endpoint/mac-schedule-scan

そもそも `com.microsoft.wdav` の設定は、[Jamf Pro で macOS ポリシーでMicrosoft Defender for Endpointを設定する - Microsoft Defender for Endpoint | Microsoft Learn](https://learn.microsoft.com/ja-jp/defender-endpoint/mac-jamfpro-policies#gui-method)に紹介される[schema.json](https://github.com/microsoft/mdatp-xplat/tree/master/macos/schema/schema.json)で管理していました。


もしかしたら、`com.microsoft.wdav` の設定が2つあると認識されないのかもしれません。
そこで、定期スキャンをCustomSchemaに対応させることにしました。
これにより、設定変更が容易になり、設定の一元管理が可能になります。

## 成果物
定期スキャンに対応させたCustomSchemaです。

:::message
必ずご自身でも検証してください。不備があればぜひプルリクエストしてください。
:::

https://github.com/enpipi/mdatp-xplat/blob/master/macos/schema/schema.json

## 利用方法
ここでは定期スキャンの設定のみ紹介します。既にCustomSchemaを使って設定構築済みの場合、該当の構成プロファイルを新たに作り直す必要があります。

以下の手順は [Microsoftの公式ドキュメント](https://learn.microsoft.com/ja-jp/defender-endpoint/mac-schedule-scan) に記載の内容をもとにしています。詳細や他のオプションについては上記記事を参照してください。

ここでは毎日14:40に、除外を無視して低優先度でクイックスキャンをする設定方法を例に利用手順を紹介します。

1. Jamf Proで構成プロファイルを新規作成し、上記のjsonファイルをアップロードします。
2. アプリケーションとカスタム設定 > 外部アプリケーション > 追加を選択し、カスタムスキーマを使用します。
3. 環境設定ドメインに `com.microsoft.wdav` と入力し、スキーマを追加から[今回のschema.json](https://github.com/enpipi/mdatp-xplat/blob/master/macos/schema/schema.json)をアップロードします。
4. `features > ScheduledScan`を有効にして `ScheduledScan = enabled` にします。
5. `ScheduledScan`にチェックを入れて `ignoreExclusions`, `lowPriorityScheduledScan`,  `DailyConfiguration` の `timeOfDay` を表示します。
6. それぞれ以下設定にして保存します。
   - `ignoreExclusions = enabled`
   - `lowPriorityScheduledScan= enabled`
   - `timeOfDay = 880`

### timeOfDayについて補足
スケジュールされたスキャンを実行する時刻を、0:00から起算して1分刻みで指定します。例えば16:00の場合は16*60 = 960となります。時刻はコンピュータのローカル時刻を指します。

:::message alert
スケジュールされたスキャンは、デバイスがスリープしている間はスケジュールされた時間に実行されません。デバイスの電源がオフになっている場合、スキャンは次のスケジュールされたスキャン時間に実行されます。
:::

## 配信後の確認方法
設定を配信したら、実際にエンドポイント側で以下のコマンドを実行して確認します。

1. `mdatp health --details` を実行し、`scheduled_scan`の設定箇所が`[managed]`になっていることを確認します。
2. 設定した時間を過ぎたら、`mdatp scan list` を実行し、スキャンが実行されたことを確認します。

## おわりに
CustomSchemaがOSSで公開されていたのでやりたいことがより良い形で対応できました🎉
今回、これがきっかけで初めてプルリクエストも出しました。執筆時点ではレビュー待ちですが、もし反映されたら記事内で修正させていただきます。

以上がCustom Schemaを利用したJamf ProでのMDEの定期スキャン管理の方法です。
正しく設定すれば、定期スキャンを効率的に管理することができます。