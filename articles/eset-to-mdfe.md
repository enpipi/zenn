---
title: "Jamfを使ってESETからMicrosoft Defender for Endpointへリプレースする"
emoji: "🛡️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Jamf, ESET, Microsoft]
published: ture
---




# はじめに
ゼロデイ攻撃など昨今の脅威を踏まえて、EPPからEDRにリプレースを検討している企業も多いのではないでしょうか。
本記事ではESET Endpoint AntiVirus（以下、ESET） から Microsoft Defender for Endpoint（以下、MDfE）にリプレースした際にJamfを使ってどのように進めたかを紹介します。
本記事が誰かの参考になれば幸いです。

:::message alert
必ず手元の環境で一連の流れをテストしてから全端末に配布をしましょう。
:::

## 工夫したところ
- パッシブモードとアクティブモードの切り替えをJamfを使って自動化
  - セキュリティソフトが何も入っていない時間を作らずに配布が完了する
- MDfEのインストールでエラーが発生しないよう、M1対策および事前のプロファイル配布を明示化
  - M1とIntel Macを意識せず配布が可能


## 大まかな流れ

1. MDfE導入に必要なプロファイルを作成し、事前に配布する
2. ESETにMDfEを除外するポリシーを入れる
3. MDfEをパッシブモードで配布する
4. ESETをアンインストールすると、自動でMDfEがアクティブモードにする



### パッシブモードとアクティブモードの違い
MDfEにはセキュリティソフトがインストール済みの端末に配布できるよう、パッシブモードという機能が存在します。

今回は、ESETがインストールされている端末にパッシブモードでMDfEを配布して、
ESETをアンインストールしたらMDfEをアクティブモードに変更するという流れで導入をしました。


:::message
**参考：パッシブモードにした場合のMDfEの挙動**
- リアルタイム保護がオフになる
- オンデマンドのスキャンがオンになる
- 脅威に対する自動修復機能がオフになる
- セキュリティインテリジェンスの更新がオンになる
- ステータスメニューのアイコンが表示されない
:::




# 1. MDfE導入に必要なプロファイルの作成、事前配布
:::message alert
MDfEはシステム拡張およびカーネル拡張の許可が必要であるため、MDfEを配布する前にプロファイルの配布を済ませる必要があります。
:::

## 1-1. 必要なプロファイルとポリシーの作成


https://docs.microsoft.com/ja-jp/microsoft-365/security/defender-endpoint/mac-jamfpro-policies?view=o365-worldwide
https://docs.microsoft.com/ja-jp/microsoft-365/security/defender-endpoint/mac-preferences?view=o365-worldwide

上記記事にしたがってMDfEのプロファイルをJamfに作成します。
手順3に登場する `Defender schema.js` を導入することで、Jamfのプロファイル上で簡単にMDfEの設定を管理することができるようになります。
また、 `MDATP MDAV 構成設定` の各設定項目の説明および推奨設定も記載がありますので参考にしてください。


:::message
構成プロファイルがすべて作成できたら、  `MDATP MDAV 構成設定` **以外のプロファイル** を配布をしましょう。
:::


:::message alert
カーネル拡張機能は、M1とMacOS10.15移行、配布をしないでください
:::




# 2. ESETにMDfEを除外するポリシーを作成

https://eset-support.canon-its.jp/faq/show/39?back=front%2Fcategory%3Asearch&category_id=61&commit=&keyword=%E9%99%A4%E5%A4%96&page=1&search_attribute_id=&search_target=0&site_domain=business&site_id=7&sort=sort_keyword&sort_order=desc&utf8=%E2%9C%93

https://eset-support.canon-its.jp/faq/show/3565?back=front%2Fcategory%3Asearch&category_id=61&commit=&keyword=%E9%99%A4%E5%A4%96&page=1&search_attribute_id=&search_target=0&site_domain=business&site_id=7&sort=sort_keyword&sort_order=desc&utf8=%E2%9C%93

ESETがMDfEの挙動を止めてしまわないようにします。
上記記事を参考にESETにMDfEを除外するポリシーを反映させます。


# 3. パッシブモードとアクティブモードのプロファイル作成とターゲットの作成
パッシブモードのプロファイルとアクティブモードのプロファイルをそれぞれ作ります。

パッシブモードのプロファイルはESETとMDfE両方がインストールされている端末に配布、アクティブモードのプロファイルはMDfEのみインストールされている端末に配布とすることでESETをアンインストールすると自動でMDfEがアクティブモードに切り替わるようになります。



## 3-1. プロファイルの作成


**パッシブモードのプロファイル作成**
![](https://storage.googleapis.com/zenn-user-upload/541df262f58043dfb40ce56a.jpg)


1. `MDATP MDAV 構成設定` のプロファイルをコピー
2. 一般 > 名称に `MDATP MDAV 構成設定 パッシブモード`と記入
3. アプリケーションとカスタム設定 > 外部アプリケーション > 環境設定ドメインのプロパティ > AntiVirus engine > Add/Remove Propertiesをクリック、Pasive modeにチェックを入れる

---

**アクティブモードのプロファイル作成**
1. `MDATP MDAV 構成設定` のプロファイルをコピー
2. 一般 > 名称に `MDATP MDAV 構成設定 アクティブモード`と記入
3. アプリケーションとカスタム設定 > 外部アプリケーション > 環境設定ドメインのプロパティ > AntiVirus engine > Add/Remove Propertiesをクリック、Pasive modeのチェックを外す


以後、 `MDATP MDAV 構成設定` のプロファイルは使わないので削除します。


## 3-2. プロファイルに割り当てるSmart Computer Groupsの作成

以下2つのSmart Computer Groupを作成しプロファイルに割り当てます。

- ESETとMDfE両方インストールされているグループ
  - `MDATP MDAV 構成設定 パッシブモード` に割り当てる
- MDfEのみインストールされているグループ
  - `MDATP MDAV 構成設定 アクティブモード` に割り当てる


## 3-3. ESETとMDfE両方インストールされているグループの作成
![](https://storage.googleapis.com/zenn-user-upload/4a0f3943aefb0b5d9be08da1.jpg)
1. Smart Computer Groups > 新規 を選択
2. 表示名に適当な名前を入れる
3. クライテリアに以下を `and` で追加
    - `Application Title is ESET Endpoint Antivirus.app`
    - `Application Title is Microsoft Defender ATP.app`



##  3-4. MDfEのみインストールされているグループの作成
![](https://storage.googleapis.com/zenn-user-upload/ae16ad2d89f1344b2a9c6e7f.jpg)
1. Smart Computer Groups > 新規 を選択
2. 表示名に適当な名前を入れる
3. クライテリアに以下を `and` で追加
    - `Application Title is not ESET Endpoint Antivirus.app`
    - `Application Title is Microsoft Defender ATP.app`


# 4. MDfE本体、Microsoft Auto Update の配布
MDfEはUniversalアプリではないため、M1 MacにインストールするためにはRosetta Stoneの事前インストールが必要になります。



よって、カスタムトリガーをRosetta, MDfEでそれぞれ作成し、順番にインストールをするスクリプトを作成します。
本章の設定が済めば、ESETがインストールされている端末にMDfEがパッシブモードで配布されます。

## 4-1. Rosetta stone配布トリガーの作成

https://community.jamf.com/t5/jamf-pro/deploy-rosetta-on-m1-machines-before-everything-else/m-p/223510/highlight/true#M211919

1. 上記記事のスクリプトをコピーして新規スクリプトで登録する
2. ポリシー > 新規 > スクリプトに上記スクリプトを割当
3. トリガー > カスタムを選択し、 `install_rosetta`と記入
4. 実行頻度 > `Ongoing` を選択
3. Scope > `All Computer` を選択
   1. スクリプト上でM1かIntelかを判別してくれますが、気になる場合は Scopeを「M1 Macのみ」とするようにしましょう。


## 4-2. MDfEの配布トリガーの作成

1. 上記記事に従ってMDfEの配布ポリシーを作成
2. トリガー > カスタムを選択し、 `install_mdfe`と記入
3. 実行頻度 > `Ongoing` を選択
4. Scope > `All Computer` を選択


## 4-3. MDfE本体の配布対象をSmart Computer Groupsで作成
「MDfE導入に必要なプロファイルがすべてインストール済み」というグループを作成します。
前述のとおり、MDfEを配布する前にプロファイルを予め配布しておく必要があるからです。


![](https://storage.googleapis.com/zenn-user-upload/a774e9e7979c26ccf08851da.jpg)

1. Smart Computer Groupsを選択
2. 新規を選択
3. 表示名に適当な名前を記入する
4. クライテリア > 追加 > アドバンスクライテリアを表示 > profile name を選択
5. 3,4を繰り返して1-1で作成したプロファイルを追加して保存

:::message alert
以下のプロファイルは事前配布する必要がないので、本グループのクライテリアに含めません
- `MDATP MDAV 構成設定 パッシブモード`
- `MDATP MDAV 構成設定 アクティブモード`
:::


## 4-5. 順番にインストールをするスクリプトを作成
```shellscript
#!/bin/sh

/usr/local/jamf/bin/jamf policy -trigger install_rosetta
/usr/local/jamf/bin/jamf policy -trigger install_mdfe -verbose

```
1. 上記スクリプトをコピーして新規スクリプトで登録する
2. ポリシー > 新規 > スクリプトに上記スクリプトを割当
3. 前述のグループをScopeに割当


---


これで、必要な構成プロファイルがインストールされた端末にRosseta Stone, MDfE, MAUの順番でアプリケーションがインストールされるポリシーができました。


# 5. ESETアンインストールのスクリプトを作成し、配布する
ESET AntiVirusとESET Remote Admin Agentをアンインストールするスクリプトを割り当てたポリシーを作成し、ESETとMDfE両方インストールされているSmart Computer Groupを対象に実行します。
これで、ESETがアンインストールされると自動でパッシブモードからアクティブモードに切り替わります。

なお、アンインストールスクリプトの作成は以下の記事を参考にしました。
[Jamf Proを利用してESET for Macをアンインストールする │ システム運用日記](https://blog.intracker.net/archives/3361)


1.　上記スクリプトをコピーして新規スクリプトで登録する
2. ポリシー > 新規 > スクリプトに上記スクリプトを割当
3. 「3-3. ESETとMDfE両方インストールされているグループ」をScopeに割当

# 完了！
お疲れ様でした。
Jamfを使うことで、ユーザーの工数を使わずにリプレースを完了させることができました。
またパッシブモードの期間を伸ばすことで導入時の影響範囲をみることができます。
この方法は他のEPPでも利用できる方法だと思いますので、参考になれば幸いです。

## 余談
移行時には上記に加えて、動的グループ「MDfEをインストールするのに必要なプロファイルが入っていない端末」を用意してDashboard上で観察しました。
これによって稼働していない端末や誤作動の恐れがある端末も洗い出されるのでおすすめです。
