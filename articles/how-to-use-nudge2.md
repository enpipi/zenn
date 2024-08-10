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
2.0になってから多くの機能が追加されたので、簡単に触ってみた結果を共有します。

## Nudge2.0からの大きな変更点
- Nudge 2.0からmacOS 11はサポート外になりました。
- Markdownに対応しました。利用できるMarkdwonは以下記事を参照してください
    - [Using Markdown with Apple News Format | Apple Developer Documentation](https://developer.apple.com/documentation/apple_news/apple_news_format/components/using_markdown_with_apple_news_format#2975763)
- SOFA Feedに対応しました。
    - これにより、最新のmacOSのバージョンを自動で確認してNudgeを発火できるようになりました

:::info
SOFA(Simple Organized Feed for Apple Software Updates)はmacOSとiOSのアップデート情報を効率的に追跡するためのツールです。OSアップデートやX Protectに関する情報をフィードしてくれます。
[Getting Started | SOFA - by MAOS](https://sofa.macadmins.io/getting-started.html)
::::

## 注意点
Installomaterは既にNudge2.0を取得するようになっています。
サポート外であるmacOS11への対応はNudge 1.xを引き続き利用する必要があります。
Mardownに対応したことでNudge内の説明文に意図しないスタイルが割り当たる可能性があります。


## 詳細な変更点
- `requiredInstallationDate` にUTCに加えて、ローカルのタイムゾーンを渡せるようになりました。
 - ただし、後述のSOFAを利用した変数を利用する場合は基本UTCとなります。ローカルのタイムゾーンを用いるにはSOFAのカスタムフィードを仕様して加工が必要なようです。
- `requiredMinimumOSVersion` にSOFAを利用した新たな変数を3つ渡せるようになりました。
    -  `latest` ： 常に最新リリースを強制します。マシンがこのバージョンに対応していない場合は、「サポート対象外デバイス」専用の画面を表示します。
    - `latest-supported`: このデバイスでサポートされている最新バージョンのリリースを参照します。
    - `latest-minor`： 現在のメジャー・リリースにとどまり、最新のマイナー・アップデートを参照します
        - SOFAをセルフホスティングする場合や参照しない場合はSOFAを別途オプトアウトする必要があります。
    - 上記、変数を利用する場合に限り、アップデート内容にあわせて`requiredInstallationDate`を自動設定できるようになりました。
        - それぞれの日数は`osVersionRequirement`key配下にある以下keyで設定ができます。
            - `activelyExploitedCVEsMajorUpgradeSLA`:14日
            - `activelyExploitedCVEsMinorUpdateSLA`: 14日
            - `nonActivelyExploitedCVEsMajorUpgradeSLA`: 21日
            - `nonActivelyExploitedCVEsMinorUpdateSLA` : 21日
            - `standardMajorUpgradeSLA`: 28日
            - `standardMinorUpdateSLA` : 28日
        -  これらの日付はSOFAフィードの`ReleaseDate`キーに対して計算され、UTCフォーマットされます。
        -  ユーザーの遅延実行については`userExperience` 配下の `nudgeMajorUpgradeEventLaunchDelay`と `nudgeMinorUpdateEventLaunchDelay`にそれぞれ遅延可能な最大日数を設定してください
    - 既知のCVEがないリリースのナッジイベントを発生させないように設定ができます。
        - `optionalFeatures` の `disableNudgeForStandardInstalls` を`true`に設定してください。
- UIの左側にある「更新までの残り日数」の項目を無効にできるようになりました。
    - `userInterface`の `showDaysRemainingToUpdate`をfalseに設定してください。



## JamfProを用いたNudge2.0の構成
Nudge 2.0に対応したCustomSchemaを持ちます。
[nudge/Schema/jamf/com.github.macadmins.Nudge.json at main · macadmins/nudge · GitHub](https://github.com/macadmins/nudge/blob/main/Schema/jamf/com.github.macadmins.Nudge.json)


