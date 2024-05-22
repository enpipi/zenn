---
title: "Jamf Proを使ったMDEの定期スキャン管理"
emoji: "🕒"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [JamfPro,Security,Microsoft,Tech]
published: false
publication_name: "visasq"
---


## ゴール
この記事を読めば、Jamf Proを使ったMicrosoft Defender for Endpoint（MDE）の定期スキャンを管理する方法がわかります。

## はじめに
Jamf Proを使用してMDEの設定を管理することができますが、定期スキャンについても同様に管理することができます。今回、予期せぬところで問題に遭遇したため、その知見を共有します。

前提として、MDEが導入されており、Jamf Proが利用可能であることを想定しています。

## 問題の概要
多くの方は、[こちらのドキュメント](https://learn.microsoft.com/ja-jp/defender-endpoint/mac-jamfpro-policies#3b-set-policies-using-jamf) を参照して[schema.json](https://github.com/microsoft/mdatp-xplat/tree/master/macos/schema/schema.json)を使ってMDEの設定を行ったのではないでしょうか。しかし、その状態で[定期スキャンの設定](https://learn.microsoft.com/ja-jp/defender-endpoint/mac-schedule-scan) をそのまま実装しても、期待通りに動作しません。

### 原因
どちらの設定も`com.microsoft.wdav`ドメインの構成プロファイルを使用するため、設定を一つにマージする必要があります。
しかし、現状の[schema.json](https://github.com/microsoft/mdatp-xplat/tree/master/macos/schema/schema.json) には定期スキャンの設定が含まれていません。

## 解決策
上記の[schema.json](https://github.com/microsoft/mdatp-xplat/tree/master/macos/schema/schema.json)に定期スキャンの設定を追加したものが[こちら](https://github.com/enpipi/mdatp-xplat/blob/master/macos/schema/schema.json)です。PRも出しましたが、この記事の執筆時点ではレビュー待ちのため、自己責任でご利用ください。

## 利用方法
以下の手順は[定期スキャンの設定](https://learn.microsoft.com/ja-jp/defender-endpoint/mac-schedule-scan)に準拠しています。詳細な説明や他のオプションについては上記記事を参照してください。ここでは例に基づいた構成方法を記載します。

1. **Jamf Proで構成プロファイルを開き、上記のjsonファイルをアップロードします。**
2. **Future > ScheduledScanを有効化する。**
   - `ScheduledScan`にチェックを入れ、`timeofday`に値を設定します。

## 配信後の確認方法
設定を配信したら、実際にエンドポイント側で以下のコマンドを実行して確認します。

1. `mdatp health --details` を実行し、`scheduled_scan`の設定箇所が`[managed]`になっていることを確認します。
2. 設定した時間を過ぎたら、`mdatp scan list` を実行し、スキャンが実行されたことを確認します。

---

以上がJamf Proを使用したMDEの定期スキャン管理の方法です。正しく設定すれば、定期スキャンを効率的に管理することができます。