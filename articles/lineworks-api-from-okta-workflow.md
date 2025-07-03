---
title: "LINE WORKS APIで躓いたこと、OktaWorkflowでService Account認証をしてみた"
emoji: "⛳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
publication_name: "notahotel"
---


NOT A HOTELでは、顧客とのやりとりでLINE WORKSを利用するケースがあります。
LINE WORKSはSCIMに対応していないため、Okta Workflowsを用いてLINE WORKSのアカウント作成と削除を自動化しています。

本記事では、LINE WORKSのAPIをOkta Workflowから利用する際に躓いた点を紹介しながら、対応方法をステップ・バイ・ステップで説明します。

## LINE WORKS APIの事前の注意

LINE WORKS APIを利用する前に、以下の注意点を確認してください。

### フリープランでは利用できるAPIに制限がある

LINE WORKSでは、プランによって実行できるAPIが異なります。
最初にこの点に気づかず、フリープランの検証環境を使用してしまい、時間を無駄にしてしまいました。

例えば、Directory APIの概要記事には次のように記載されています：

> フリープランでは、user.read、group.read、orgunit.read、user.email.read、user.profile.read のみが利用できます。directory、directory.read、user、group、orgunit は利用できません。
参考：[Directory API | Developers](https://developers.worksmobile.com/jp/docs/directory)

プラン別の利用可能APIについては、以下を参照してください：
参考：[OAuth Scopes | Developers](https://developers.worksmobile.com/jp/docs/auth-scope#line-works)

### Directory APIでドメインのリソースを管理する場合は、Service Account認証を使用する

LINE WORKSは2023年より以下のように仕様を変更しています：

> 最高管理者を除く管理者は、通常のユーザーと同様に、ユーザーとして与えられた権限に基づいて、自身の情報など限定的なリソースに限り参照・管理ができます。与えられた管理権限に基づいてドメイン内のすべてのリソースにアクセスできるわけではありません。
参考：[Developers](https://developers.worksmobile.com/jp/news/detail?id=614)

最初にこの点に気づかず、User Account認証で実装してしまい、時間を無駄にしてしまいました。
Service Account認証で利用できないAPIについては、以下を参照してください：

[Service Account 認証 (JWT) > Service Account では利用できない API](https://developers.worksmobile.com/jp/docs/auth-jwt#prohibited-api)

### 指定するスコープは1つで十分

複数のスコープが指定されている場合、いずれか1つのスコープがあれば実行できます。
例えば、[ユーザーの取得](https://developers.worksmobile.com/jp/docs/user-get)ではスコープに `user`、`user.read`、`directory`、`directory.read` とありますが、いずれか1つのスコープを指定すれば利用できます。

### 開発ドキュメントのサポートは受けられないため、LINE WORKS Developers Communityを活用する

LINE WORKSは開発系のサポートを対象外としています。LINE WORKS Developers Communityを活用しましょう。

https://forum.worksmobile.com/jp/

## Okta WorkflowからAPIを叩く

それでは、Service Account認証を用いてAccess Tokenを取得します。
今回はOkta WorkflowでLINE WORKSの認証を通してみます。

## 全体の流れ

1. 事前準備
2. JSON Web Token（JWT）の署名〜Access Tokenの取得
3. 実行してみる

### 事前準備

LINE WORKS Developer Consoleにサインインします：
https://dev.worksmobile.com/jp/console/openapi/v2/app/list/view

1. アプリの新規追加から、新しいアプリを作成します
2. アプリ名等の設定をしたら **[保存]** を押します
3. Service Accountの **[発行]** を押します
4. 以下の値をこのあと利用します：
   - Client ID
   - Client Secret
   - Service Account
   - Private Key
5. **[変更]** を押します

### JSON Web Token（JWT）の署名〜Access Tokenの取得

![](/images/lineworks-api-from-okta-workflow/lineworks_auth_oktaworkflow.png)

LINE WORKSでユーザー作成のAPIを実行するには、Access Tokenが必要です。Access Tokenの発行にはJWTを使用して署名する必要があります。
Service Account認証は、JSON Web Token（以降、JWT）を使用してアプリ専用の仮想管理者アカウントで認証を行い、Access Tokenを発行してAPIを利用する方法です。

https://developers.worksmobile.com/jp/docs/auth-jwt

:::message
JSON Web Token（JWT）生成時の `expires_in` は3600秒にする
ドキュメントでは、`expires_in` は設定に従うと記載がありますが、検証した際には設定に関わらず3600秒にしないとエラーになりました。

> 有効期限は、Developer Console > API > ClientApp の [トークン設定 > Access Tokenの有効期限] の設定に従う。
> ・1 hour (3600 秒)
> ・24 hours (86400 秒)
> 設定された有効期限が経過すると、自動的に満了する。
参考：[Service Account 認証 (JWT) > Response Body (JSON)](https://developers.worksmobile.com/jp/docs/auth-jwt#issue-access-token-response-body)
:::

1. Okta Workflowの関数からJWT > `Sign` を選び、以下の値を入力します：
   - key : Private Keyの値
   - issuer : Client IDの値
   - expiresin : `3600`
   - noTimeStamp : `False`
   - header : `{"alg" : "RS256", "typ" : "JWT"}`
   - subject : Service Accountの値
   - algorithm : HS256

2. 関数から `Construct` を選び、以下の値を入力します：
   - grant_type : `urn:ietf:params:oauth:grant-type:jwt-bearer`
   - assertion : 1で作成した `Sign` の出力 `jwt`
   - client_id : scope
   - client_secret : Client Secretの値
   - scope : `directory`

3. Applicationから `API Connector` の `POST` を選び、以下の値を入力します：
   - URL : `https://auth.worksmobile.com/oauth2/v2.0/token`
   - Query : `{}`
   - Headers : `{"Content-Type": "application/x-www-form-urlencoded"}`
   - Body : 2で作成した `Construct` の出力 `output`
   - Connection : 使わないので何でも良いです

### 実行してみる

問題なければ `access_token` が返ってきます。この `access_token` を使ってLINE WORKSの様々なAPIを実行できるようになりました。
トリガーをUser Assigned to Applicationにすれば、LINE WORKSがアサインされたときにアカウント自動作成といったことができます。

## むすび

LINE WORKS APIの仕様のキャッチアップや、慣れないJWTの生成の参考になれば幸いです。
