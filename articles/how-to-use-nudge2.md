---
title: "Nudge2.0触ってみた"
emoji: "⛳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [JamfPro,Nudge]
published: false
publication_name: "visasq"
---


## 概要
Nudgeがv2.0をリリースしました。
2.0になってから多くの機能が追加されたので、Jamf Proを使って簡単に触ってみた結果を共有します。


## Nudgeとは
NudgeはmacOSでのOSバージョン要件を満たさない場合にアップデートを促す仕組みです。
Nudgeの導入については以下記事がわかりやすいです
[macOSのアップデート通知にNudgeを設定してみた | システム運用日記](https://blog.intracker.net/archives/4203)

## Nudge2.0からの大きな変更点
- Nudge 2.0からmacOS 11はサポート外になりました。
- Markdownに対応しました。利用できるMarkdwonは以下記事を参照してください
    - [Using Markdown with Apple News Format | Apple Developer Documentation](https://developer.apple.com/documentation/apple_news/apple_news_format/components/using_markdown_with_apple_news_format#2975763)
- SOFA Feedに対応しました。
    - これにより、最新のmacOSのバージョンを自動で確認し、Nudgeをトリガーできるようになりました
- 検証用のコマンドライン引数の拡充
 - コマンドラインで期限日やハードウェア、OSを引数で指定できるようになりました。これにより検証に用いた端末とは異なる環境での検証が容易になりました
- 他にもローカルタイムゾーンを参照したり、自前で建てたSOFA Feedを参照することなどもできるようです。Nudgeの画面を動かすこともできるようになりました。詳しくは
https://github.com/macadmins/nudge/wiki/v2.0-features を参照してください

:::info
SOFA(Simple Organized Feed for Apple Software Updates)はmacOSとiOSのアップデート情報を効率的に追跡するためのツールです。OSアップデートやX Protectに関する情報をフィードしてくれます。
[Getting Started | SOFA - by MAOS](https://sofa.macadmins.io/getting-started.html)
::::

## Nudge2.0の注意点
- サポート外であるmacOS11が残っている場合、動作保証されませんのでNudge1.xを活用しましょう。
- Markdownに対応したことでNudge内の説明文に意図しないスタイルが割り当たる可能性があります。
- randomDelayの初期値がfalseからtrueに変わりました
  - 検証時に用いるコマンドに引数 `-disable-random-delay`を渡す必要があります
- InstallomaterやJamfAppCatalogでは既にNudge2.0を配信するようになっていますので意図せず上記影響を受けていないか確認を推奨します。

## JamfProを用いたNudge2.0の構成プロファイルの作成
[nudge/Schema/jamf/com.github.macadmins.Nudge.json at main · macadmins/nudge · GitHub](https://github.com/macadmins/nudge/blob/main/Schema/jamf/com.github.macadmins.Nudge.json)

### STEP 1 カスタムスキーマを用いたプロファイルの設定
1. Jamf Proにログインして新規構成プロファイルを作成します。
2. アプリケーションとカスタム設定 > 外部アプリケーションを選択します。
3. 右上の追加をクリックします
4. ソースからカスタムスキーマを選択し、環境設定ドメインに `com.github.macadmins.Nudge` と入力します
5. スキーマ追加を選択し、以下ページにある内容をペーストしてください
  - [nudge/Schema/jamf/com.github.macadmins.Nudge.json at main · macadmins/nudge · GitHub](https://github.com/macadmins/nudge/blob/main/Schema/jamf/com.github.macadmins.Nudge.json)
6. 保存をクリックします。


## STEP2 必要最小OSバージョンの自動指定
1. `optionalFeatures` > `utilizeSOFAFeed` を `true` に設定します
2. `osVersionRequirements` > `requiredMinimumOSVersion` に以下いずれかの値を設定します。
 - `latest` ： 常に最新リリースを強制します。デバイスがこのバージョンに対応していない場合は、「サポート対象外デバイス」専用の画面を表示します。
 - `latest-supported`: このデバイスでサポートされている最新バージョンのリリースを参照します。
 - `latest-minor`： 現在のメジャー・リリースにとどまり、最新のマイナー・アップデートを参照します

:::message
サポート対象外デバイスの場合、次のような画面が表示されるようになります。
![nudge-unsupported-device](nudge-unsupported-device/nudge-unsupported-device.png)
:::


## STEP3 【任意】 CVEを含まないアップデートを対象外にする
CVEが含まれないアップデートをNudgeの対象外にできます。
脆弱性対策だけにNudgeを用いたい場合に有効です。
`optionalFeatures` > `disableNudgeForStandardInstalls` を`true`に設定してください。（既定値はfalse）

## STEP4 【任意】 アップデート期限の設定
アップデート内容にあわせて期限をそれぞれ設定できます。
設定しない場合は既定値の日数となります。

期限を変更するには、`osVersionRequirement` 配下にある該当Keyをそれぞれ編集してください。

| アップデート内容 | Key | 期限の既定値 |
| :----: | :----: | :----: |
| 悪用確認済みのCVEを含むメジャーアップグレード | `activelyExploitedCVEsMajorUpgradeSLA` | 14日 |
| 悪用確認済みのCVEを含むマイナーアップデート | `activelyExploitedCVEsMinorUpdateSLA` | 14日 |
| 悪用未確認のCVEを含むメジャーアップグレード | `nonActivelyExploitedCVEsMajorUpgradeSLA` | 21日 |
| 悪用未確認のCVEを含むマイナーアップデート | `nonActivelyExploitedCVEsMinorUpdateSLA` | 21日 |
| CVEが含まれないメジャーアップグレード | `standardMajorUpgradeSLA` | 28日 |
| CVEが含まれないマイナーアップデート | `standardMinorUpdateSLA` | 28日 |



また、既定値では新しいバージョンを検知次第、Nudgeをトリガーしますが、日単位で遅延できます。
`userExperience`配下にある`nudgeMajorUpgradeEventLaunchDelay`,`nudgeMinorUpdateEventLaunchDelay`に遅延させたい日数を定義してください。

:::message
`nudgeMajorUpgradeEventLaunchDelay``nudgeMinorUpdateEventLaunchDelay`を設定すると動作確認時にコマンドラインからNudgeイベントを即時実行できくなるので注意が必要です。
:::

## STEP5 【任意】 表示情報の調整
userinterface配下にある以下Keyで画面左部に表示される情報を調整できます
- `showDaysRemainingToUpdate`
 - 更新までの残り日数(Days Remaining To Update:)の表示・非表示
- `showRequiredDate`
 - 必要な日付（RequiredDate）の表示・非表示
 - 更新期限と読み替えて貰えれば大丈夫です。
- `showActivelyExploitedCVEs`
 - Actively Exploited CVEsの表示・非表示





## 例
CVEを含むアップデートだけを対象に、OS14は最新のマイナーバージョン、それ以外のOSでは最新リリースのバージョンになるようにNudgeイベントをトリガーする

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
    <dict>
        <key>optionalFeatures</key>
        <dict>
            <key>disableNudgeForStandardInstalls</key>
            <true/>
            <key>utilizeSOFAFeed</key>
            <true/>
        </dict>
        <key>osVersionRequirements</key>
        <array>
            <dict>
                <key>aboutUpdateURL</key>
                <string>https://visasq.esa.io/posts/37968</string>
                <key>requiredMinimumOSVersion</key>
                <string>latest-minor</string>
                <key>targetedOSVersionsRule</key>
                <string>13</string>
            </dict>
            <dict>
                <key>aboutUpdateURL</key>
                <string>https://visasq.esa.io/posts/37968</string>
                <key>requiredMinimumOSVersion</key>
                <string>latest-minor</string>
                <key>targetedOSVersionsRule</key>
                <string>14</string>
            </dict>
            <dict>
                <key>aboutUpdateURL</key>
                <string>https://visasq.esa.io/posts/37968</string>
                <key>requiredMinimumOSVersion</key>
                <string>latest</string>
                <key>targetedOSVersionsRule</key>
                <string>default</string>
            </dict>
        </array>
    </dict>
</plist>

```




## 動作確認
ターミナルで以下のコマンドを実行することで動作確認ができます。

`/Applications/Utilities/Nudge.app/Contents/MacOS/Nudge -disable-random-delay `


Nudge2.0から引数が増えて、様々な環境をシミュレートできるようになり動作確認が容易になりました。

| 引数 | 説明 |
| --- | --- |
| -simulate-os-version "14.4.1" | 指定したOSバージョンでの動作を確認する |
| -simulate-hardware-id "J516cAP" | 指定したハードウェアのデバイスIDにオーバーライドする<br>必要なOSバージョンをサポートしていない端末の検証に用いる |
| -simulate-date "2024-08-01T08:00:00Z" | 実行日時以外の日時を用いる。フォーマットは `YYYY-MM-ddTHH:mm:ssZ` |
| -disable-random-delay | udge 2.0ではデフォルトで有効になっているRandom Delayを回避する。動作確認では必須の引数 |
https://github.com/macadmins/nudge/wiki/Command-Line-Arguments


例：OSサポート外のデバイスでのNudgeの動作確認
`/Applications/Utilities/Nudge.app/Contents/MacOS/Nudge -simulate-os-version "12" -disable-random-delay `



### トラブルシューティング
設定により、ターミナルで即時実行しても画面が表示されない場合があり公式ドキュメントでは以下のような対処法が記載されています。それでも表示されなければ、プロファイルの設定を確認したほうが良さそうです。

- https://github.com/macadmins/nudge/wiki/Common-Issues#i-deleted-the-sofa-macos_data_feedjson-and-now-nudge-will-not-launch
- https://github.com/macadmins/nudge/wiki/User-Deferrals#testing-and-resetting-nudge



## 最後に

Nudge2.0から追加された機能を使うことで定期的なOS更新や脆弱性の重要度に合わせて更新期限を変更できるようになりました。メンテナンスも楽になったのでより運用しやすくなったと思います。


