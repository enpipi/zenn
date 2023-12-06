---
title: "システム管理者のためのFigma解説 Business Plan編"
emoji: "🏢"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Figma]
published: true
---


この記事は [corp-engr 情シスSlack（コーポレートエンジニア x 情シス）#2 Advent Calendar 2023](https://adventar.org/calendars/8869)　の5日目の記事です。

追記
2023/12/05 : 以下を加筆しました。
- ユーザーの一覧機能について
- TrueUpの詳細


# 1.はじめに
Figmaはブラウザベースのコラボレーション・インターフェイス・デザイン・ツールです。
今やトップのシェア率を誇り、UIデザイナーにとって必要不可欠のツールになりつつあります。

Figmaは非常に強力なツールですが、システム管理者からみると独自の設計方針に頭を悩ませることが多いことも事実です。
この記事ではFigma Businessプランをシステム管理者向けに解説します。
フリー・プロプランについてはこちらをご覧ください。
https://note.com/enpipi_work/n/n41499d921b8d

本記事では、デザイナー向けの機能については割愛いたします🙏

また、執筆に誤りなどあればぜひご指摘ください。

## 1.1 FigmaとFigJam
FigmaにはFigmaとFigJam2つのプロダクトがあり、1つのテナントで管理されます。
前者はデザインツール、後者はMiroなどに近いコラボレーションツールとなっています。
それぞれ課金は別となっていますが、どちらかをオフにしたりプランを個別に設定することができません。
FigmaをビジネスプランにアップグレードするとFigJamもビジネスプランにアップグレードされます。

:::message
FigmaとFigJamについては以下記事を参照ください。
[FigmaとFigJamの比較 – Figma Learn - ヘルプセンター](https://help.figma.com/hc/ja/articles/1500004290201-Figma%E3%81%A8FigJam%E3%81%AE%E6%AF%94%E8%BC%83)
:::

## 1.2 Figma Business Planの概要紹介
フリープラン、プロプランに比べてビジネスプランからは企業向けの機能が多数利用できるようになっています。

### オーガニゼーション
フリープラン、プロプランでは組織構造の最上位がチームでしたが、ビジネスプランから複数のチームを束ねるオーガニゼーション機能が利用できます。

### 共用ライブラリ・デザインシステム
オーガニゼーションを利用できることでメンバーやチームは勿論のことプラグインやライブラリ、フォントも管理の一元化が実現できるようになりました。
とくにデザインシステム、バリアブルなどは組織内のアセットの効率化や多言語の対応管理コストが下がるためデザイナーにとってより働きやすい世界になはずです。

https://www.figma.com/ja/design-systems/


### セキュリティ機能の充実
セキュリティ機能も充実し、SAMLによるSSOやログの取得もできるようになっています。

### 異なる請求方式
請求方式もTrueUpという概念が入り、ビジネスプランからは管理運用が大きく異なっています。


## 1.3 Business Planの利用シーンと目標
ビジネスプランは企業向けのプランです。
組織内で複数のチームで1つのプロダクトを開発している場合、複数プロダクトを一元管理する場合、セキュリティを向上させたい場合に適しています。
企業規模が大きくなりより多くのプロダクトや部署でのFigma利用を一元管理したい場合は最上位のエンタープライズプランが適していそうです。

# 2. Figma Business Planの特長と価格
## 2.1 Business Planのコストとその内訳
ビジネスプランは年払いのみの提供です。月払いはありません。
Figma / FigJamどちらもEditorが対象となります。
- Figma : ¥81,000/Editor（月額¥6,750）
- FigJam : ¥9,000/Editor（月額¥750）
:::message
執筆時点での価格になります。正確な金額はFigmaのページを参照ください
https://www.figma.com/ja/pricing/
:::

詳細や例外は後述しますが、Figmaは内部・外部の招待が誰でも可能。Editor以上の権限を持っていれば他の人をEditorとして招待できます。そのため、組織によってはEditorの数がどんどん増えていきます。プロプランまでは即時請求でしたが、ビジネスプランからはTrueUpと呼ばれる請求の仕組みに則り請求されます。

## 2.2 TrueUp
![demo](https://help.figma.com/hc/article_attachments/10278334122391)引用元：[ビジネスプランとエンタープライズプランの請求管理 – Figma Learn - ヘルプセンター](https://help.figma.com/hc/ja/articles/360040328293-%E3%83%93%E3%82%B8%E3%83%8D%E3%82%B9%E3%83%97%E3%83%A9%E3%83%B3%E3%81%A8%E3%82%A8%E3%83%B3%E3%82%BF%E3%83%BC%E3%83%97%E3%83%A9%E3%82%A4%E3%82%BA%E3%83%97%E3%83%A9%E3%83%B3%E3%81%AE%E8%AB%8B%E6%B1%82%E7%AE%A1%E7%90%86)


Figmaはビジネスプランから即時請求ではなく3か月に一度の請求となります。
3か月に一度の請求までは、Editorの数をどれだけ増やしても請求されません。
TrueUpの期限までに棚卸をして、追加分だけが請求となります。

TrueUpにより一時的にプロトタイプを第三者に触ってもらう、普段はEditorにする必要がないメンバーを一時的にEditorにするなど柔軟な対応が可能になりました。

※2023/12/05追記
請求のひと月前にメールで通知が来るほか管理者には管理画面でアラートが表示されます。
TrueUpの期日までであれば、こちらのタイミングで確定することができるので、期日ギリギリに誰かがいきなり100名招待したというような事象は避けることができます。
また、後述するオーガニゼーションの実装によりメンバーとゲストの一覧を確認できます。
一覧画面では、フィルター機能により前回のTrueUpから新たにEditorになったユーザーのみ一覧化することも可能です。

詳細はFigmaの公式ページを参照ください。

## 2.3 評価版の案内
ビジネスプランのトライアルは提供されていません。
Figmaは2022年3月より日本法人が設立されており、日本語でのセールスサポートを受けることができます。

## 2.4 アップグレード
アップグレード時に今までテナント外にあったチームもサポートと連携することで1つのテナントに移管できます。
移管しても配下のプロジェクトやファイルのURLは変わりません。
ただしメンバーとゲストの概念が変わるため、後述の「メンバーとゲストの違い、判定方法」をご確認ください。

# 3. Figma Business Planの主な機能
ここでは一部私が抜粋したものを紹介します。
各プランでの比較、機能の詳細は以下ページをご確認ください。
[Figmaのプランと特徴 – Figma Learn - ヘルプセンター](https://help.figma.com/hc/ja/articles/360040328273-Figma%E3%81%AE%E3%83%97%E3%83%A9%E3%83%B3%E3%81%A8%E7%89%B9%E5%BE%B4)

## 3.1 チーム協働と共有機能

### オーガニゼーション
![Org_Plan___Visualization.png](https://help.figma.com/hc/article_attachments/4421092527511/Org_Plan___Visualization.png)
引用元：[組織入門 – Figma Learn - ヘルプセンター](https://help.figma.com/hc/ja/articles/360039957374-%E7%B5%84%E7%B9%94%E5%85%A5%E9%96%80)


プロプランまでは単一のチームのみでしたが、ビジネスプランからはチームを無制限で作成でき、オーガニゼーションで1つにまとめることができます。
そのため、今まではProject単位で管理していたデザインファイルをチーム単位で管理することなどができるようになりました。

※2023/12/05追記
また、オーガニゼーションが実装されたことで現在Figmaにアクセスできるメンバーとゲストの一覧を管理者が確認できるようになりました。
ゲストがどのファイルにアクセスしているかも把握できるようになり、一覧のエクスポートもできます。

### 3種のチームタイプ
複数のチームを管理できることに伴い、チームにも3種類のタイプができました。

- オープン
    - メンバーならだれでも閲覧可能
- クローズ
    - チーム名とアイコンはだれでも閲覧可能（存在することがわかる）
    - 参加には申請が必要。チームのアドミンが承認することでアクセス可能。
- シークレット
    - 招待されない限り、チームの存在がわからない。
    - チーム配下の情報にもアクセスができない

### メンバーとゲストの違い、判定方法
プロプランまでは、プロジェクトやファイルなど単一のオブジェクトに自由に招待ができました。
ビジネスプランからは登録したドメインは必ずメンバーとなり、登録外のドメインは必ずゲストとなります。
たとえば業務委託の方に同一ドメインで払い出していた場合にアクセス権がフルアクセスになるので注意が必要です。

[メンバーとゲストの比較 – Figma Learn - ヘルプセンター](https://help.figma.com/hc/ja/articles/4420557314967-%E3%83%A1%E3%83%B3%E3%83%90%E3%83%BC%E3%81%A8%E3%82%B2%E3%82%B9%E3%83%88%E3%81%AE%E6%AF%94%E8%BC%83)

### ファイル共有
プロプランまではメンバーによる公開リンクの作成を制限できませんでした。
ビジネスプランは公開リンクを無効化できます。

:::message
エンタープライズプランにすると、公開リンクのパスワード要求の必須化・公開期限の設定が可能です。
[公開リンクの共有とオープンセッションの管理 – Figma Learn - ヘルプセンター](https://help.figma.com/hc/ja/articles/5726756336791)
:::


### 役割（権限）について
Figma, FigJamそれぞれにEditor・Viewer・Viewer-restrictedという役割があります。
管理者権限は上記役割とは別概念で設けてあり、Viewerでも管理者になれます。
Figmaを課金せず管理権限を付与できるのでシステム管理者としてはありがたいです。

残念ながらゲスト招待の制限ができるのはエンタープライズプランからとなっており、ビジネスプランでは誰でもユーザー招待ができます。


### Viewer-restricted
Viewer-restricted（限定閲覧者）はビジネスプランより新たに増えた役割です。
Figmaでは、Viewerのユーザーがファイルを作成すると、役割が自動的にEditorにアップグレードされます。
管理者がEditorからViewerにダウングレードするとそのユーザーはViewerではなくViewer-restrictedという役割になります。
Viewer-restrictedはViewerと違い、新たなファイルを作成するなどEditorにアップグレードするトリガーを実行することができません。
代わりに実行しようとすると、管理者にアップグレードを申請できます。

[Figmaの役割 – Figma Learn - ヘルプセンター](https://help.figma.com/hc/ja/articles/360039960434-Figma%E3%81%AE%E5%BD%B9%E5%89%B2)

### ドメインキャプチャー
![Domain_Capture___Visualization](https://help.figma.com/hc/article_attachments/4421062533143/Domain_Capture___Visualization.png)
引用元：[ドメインキャプチャーの有効化または無効化 – Figma Learn - ヘルプセンター](https://help.figma.com/hc/ja/articles/4421069370391-%E3%83%89%E3%83%A1%E3%82%A4%E3%83%B3%E3%82%AD%E3%83%A3%E3%83%97%E3%83%81%E3%83%A3%E3%83%BC%E3%81%AE%E6%9C%89%E5%8A%B9%E5%8C%96%E3%81%BE%E3%81%9F%E3%81%AF%E7%84%A1%E5%8A%B9%E5%8C%96)

ドメインキャプチャーは平たく言えば、対象のドメインをもつアカウントで新たなFigmaテナントを作成できなくなるためシャドーITを防ぐことができます。
また、ドメインキャプチャーを有効にしたタイミングで既存のテナントが表示され、オーガニゼーションに取り込めます。

:::message
エンタープライズプランではテナント外のコンテンツへのアクセスも制限できます
[組織外のコンテンツへのアクセスを制限する – Figma Learn - ヘルプセンター](https://help.figma.com/hc/ja/articles/12080587805719-%E7%B5%84%E7%B9%94%E5%A4%96%E3%81%AE%E3%82%B3%E3%83%B3%E3%83%86%E3%83%B3%E3%83%84%E3%81%B8%E3%81%AE%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B9%E3%82%92%E5%88%B6%E9%99%90%E3%81%99%E3%82%8B)
:::

# 4. Figma Business Planのセキュリティ機能
## 4.1 設定可能なセキュリティ機能の詳細
### SAML SSO
ビジネスプランからSAMLによるSSOが可能です。
ゲストはSSOの対象外です。

### プロビジョニング
プロビジョニングもビジネスプランから対応しています。
[SCIMからの自動プロビジョニングのセットアップ – Figma Learn - ヘルプセンター](https://help.figma.com/hc/ja/articles/360048514653-SCIM%E3%81%8B%E3%82%89%E3%81%AE%E8%87%AA%E5%8B%95%E3%83%97%E3%83%AD%E3%83%93%E3%82%B8%E3%83%A7%E3%83%8B%E3%83%B3%E3%82%B0%E3%81%AE%E3%82%BB%E3%83%83%E3%83%88%E3%82%A2%E3%83%83%E3%83%97)

:::message
エンタープライズプランでは、さらに役割の指定が可能です。
[SCIMでメンバーの役割を設定する – Figma Learn - ヘルプセンター](https://help.figma.com/hc/ja/articles/360048787534-SCIM%E3%81%A7%E3%83%A1%E3%83%B3%E3%83%90%E3%83%BC%E3%81%AE%E5%BD%B9%E5%89%B2%E3%82%92%E8%A8%AD%E5%AE%9A%E3%81%99%E3%82%8B)
:::

### ライブラリ・プラグインの承認
プラグインとウィジェットは、FigmaデザインやFigJamで動作するサードパーティ製のアプリケーションです。
ライブラリのインストール、プラグインのインストールを承認制にできます。

[Manage plugins and widgets in an organization – Figma Learn - Help Center](https://help.figma.com/hc/en-us/articles/4404228724759-Approve-plugins-in-an-organization)

### ログの取得・閲覧
プロプランまではログが閲覧できないため、インシデントが発生しても原因の特定や影響範囲を追求することができませんでした。
アクティビティーログイベントは、過去1年間、最大365日分利用できます。
アクティビティーログ内のすべてのイベントについては、次の記事を参照してください。
[アクティビティーログの閲覧とエクスポート – Figma Learn - ヘルプセンター](https://help.figma.com/hc/ja/articles/360040449533-%E3%82%A2%E3%82%AF%E3%83%86%E3%82%A3%E3%83%93%E3%83%86%E3%82%A3%E3%83%BC%E3%83%AD%E3%82%B0%E3%81%AE%E9%96%B2%E8%A6%A7%E3%81%A8%E3%82%A8%E3%82%AF%E3%82%B9%E3%83%9D%E3%83%BC%E3%83%88)

:::message
エンタープライズプランからアクティビティーログをAPIで取得できるようになります。
:::

## 4.2 データ保護と法的なコンプライアンス
FigmaはSOC 2 Type II、ISMSをはじめとしたさまざまな外部認証を取得しています。
https://www.figma.com/ja/security/

また、Figmaは、適用されるすべてのデータ保護法と規制 (GDPRおよびCCPAなど) も遵守すると明記しています。
https://www.figma.com/ja/privacy/


# 5. Figma Business Planの利用を最大化するためのヒント
## 5.1 チーム活用のヒント

### TrueUpへの対策チームを組む
3か月に一度のTrueUpは体感では驚くほどすぐやってきます。
膨れ上がったEditorを棚卸するには現場との連携が必要不可欠です。
複数のチームがあるならば、チームの中に1名担当者を設置し、仕組みの共有ができると運用が円滑になります。

### Editorを厳密に管理しようとすると死ぬ
FigmaはEditorなら誰でもEditorで招待ができるため、Editorを厳密に管理しようとすると大変なことになります。
割り切ってTrueUpの際に棚卸するようにしましょう。
何よりEditorが制限されるとFigmaでの生産性も下がってしまうので許されるのなら予算は多めに確保しておきましょう。

# 6. Figma Business Planのサポートとリソース

## 6.1 サポートの種類とそのアクセス方法
チケットタイプの問い合わせができ、サポートからはメールで返信が届きます。
日本語のサポートにも対応しています。

## 6.2 教育リソースと学習コンテンツ
サポートフォーラムがあります。
https://forum.figma.com/

## 6.3 コミュニティと他のリソース
デザイナー向けのコミュニティがあります。
https://www.figma.com/community


# 7. 総括
## 7.1 Business Planを選ぶ理由
ビジネスプランでは、組織・管理・セキュリティ機能が拡充されフロントメンバーにとっても管理者にとっても一元管理するメリットが最大化する充実した内容になっています。

## 7.2 まとめと次のステップ
エンタープライズプランについても解説記事を出したいと思います。