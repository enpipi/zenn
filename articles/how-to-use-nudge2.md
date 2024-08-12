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
NudgeはmacOSでのOSバージョン要件を満たさない場合にアップデートを促す仕組みです。

## Nudge2.0からの大きな変更点
- Nudge 2.0からmacOS 11はサポート外になりました。
- Markdownに対応しました。利用できるMarkdwonは以下記事を参照してください
    - [Using Markdown with Apple News Format | Apple Developer Documentation](https://developer.apple.com/documentation/apple_news/apple_news_format/components/using_markdown_with_apple_news_format#2975763)
- SOFA Feedに対応しました。
    - これにより、最新のmacOSのバージョンを自動で確認し、Nudgeをトリガーできるようになりました
- 検証用のコマンドライン引数の拡充
 - コマンドラインで期限日やハードウェア、OSを引数で指定できるようになりました。これにより検証に用いた端末とは異なる環境での検証が容易になりました

:::info
SOFA(Simple Organized Feed for Apple Software Updates)はmacOSとiOSのアップデート情報を効率的に追跡するためのツールです。OSアップデートやX Protectに関する情報をフィードしてくれます。
[Getting Started | SOFA - by MAOS](https://sofa.macadmins.io/getting-started.html)
::::

## Nudge2.0の注意点
- InstallomaterやJamfAppCatalogでは既にNudge2.0を取得するようになっています。
- サポート外であるmacOS11への対応はNudge 1.xを引き続き利用する必要があります。
- Markdownに対応したことでNudge内の説明文に意図しないスタイルが割り当たる可能性があります。
- randomDelayの初期値がfalseからtrueに変わりました
  - 検証時に用いるコマンドに引数 `-disable-random-delay`を渡す必要があります

# SOFA Feedを用いたバージョン指定と期限について
## 必要最小OSバージョンの自動指定
Nudgeでは、Nudgeのトリガー対象とするOSのVesionを指定できます。
Nudge1.xでは、手動で更新が必要でしたが、Nudge2.0からはSOFA Feedの情報を参照して自動で指定できるようになりました。
また、Nudge1.xではバージョンアップデートに該当しないセキュリティアップデートは検知できない仕様でしたが、Nudge2.0からはSOFAに対応したことであらゆるアップデートを検知できるようになりました。
これにより最新のOSバージョンやセキュリティアップデートが未適用の端末にNudgeがトリガーするということが可能になりました。

### 具体的な設定箇所
`requiredMinimumOSVersion` に以下いずれかの値を設定します。
- `latest` ： 常に最新リリースを強制します。デバイスがこのバージョンに対応していない場合は、「サポート対象外デバイス」専用の画面を表示します。
- `latest-supported`: このデバイスでサポートされている最新バージョンのリリースを参照します。
- `latest-minor`： 現在のメジャー・リリースにとどまり、最新のマイナー・アップデートを参照します


サポート対象外デバイスの場合次のような画面が表示されるようになります。
文言やボタンの遷移先を変更できます。

## アップデート内容にあわせた期限日の自動設定
上記いずれかの設定を利用した場合に限り、アップデート内容にあわせて期限日や実行日をそれぞれ設定できるようになりました。


### 具体的な設定箇所
アップデート内容に合わせて既定値が設定されています。
既定値を変更するには、`osVersionRequirement` 配下にある該当Keyを編集してください。

| アップデート内容 | Key | 既定値 |
| :----: | :----: | :----: |
| 悪用確認済みのCVEを含むメジャーアップグレード | `activelyExploitedCVEsMajorUpgradeSLA` | 14日 |
| 悪用確認済みのCVEを含むマイナーアップデート | `activelyExploitedCVEsMinorUpdateSLA` | 14日 |
| 悪用未確認のCVEを含むメジャーアップグレード | `nonActivelyExploitedCVEsMajorUpgradeSLA` | 21日 |
| 悪用未確認のCVEを含むマイナーアップデート | `nonActivelyExploitedCVEsMinorUpdateSLA` | 21日 |
| CVEが含まれないメジャーアップグレード | `standardMajorUpgradeSLA` | 28日 |
| CVEが含まれないマイナーアップデート | `standardMinorUpdateSLA` | 28日 |

CVEが含まれないアップデートをNudgeの対象外にすることもできます。
脆弱性対策だけにNudgeを用いたい場合に有効です。
`optionalFeatures` の `disableNudgeForStandardInstalls` を`true`に設定してください。

既定値では新しいバージョンを検知次第、Nudgeをトリガーしますが、日単位で遅延できます。
`userExperience`配下にある`nudgeMajorUpgradeEventLaunchDelay`,`nudgeMinorUpdateEventLaunchDelay`に遅延日数を定義してください




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
構成にはNudge 2.0に対応したCustomSchemaを用いると便利です。
[nudge/Schema/jamf/com.github.macadmins.Nudge.json at main · macadmins/nudge · GitHub](https://github.com/macadmins/nudge/blob/main/Schema/jamf/com.github.macadmins.Nudge.json)


