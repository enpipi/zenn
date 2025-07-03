---
title: "LINEWORKSのAPIで躓いたこと、OktaWorkflowでService Account 認証をしてみた"
emoji: "⛳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
publication_name: "notahotel"
---


NOT A HOTELでは、顧客とのやりとりでLINE WORKSを利用するケースがあります。
しかし、LINE WORKSはSCIMに対応していないので、Okta Workflowsを用いてLINE WORKSのアカウント作成と削除を自動化しています。

本記事では、LINE WORKSのAPIをOkta Workflowから利用する際に、躓いた点を紹介しながら対応方法の一部をステップ・バイ・ステップで紹介します。

## LINE WORKS APIの事前の注意

INE WORKS APIの事前の注意点を紹介します。

### フリープランでは利用できるAPIに制限がある

LINE WORKSではプランにより実行できるAPIに違いがあります。
最初その事を気が付かずにフリープランの検証環境を使ってしまい時間がかかってしまいました。

例えば、Direcotry  APIの概要記事には次のように記載されています
>フリープランでは、user.read、group.read、orgunit.read、user.email.read、user.profile.read のみが利用できます。directory、directory.read、user、group、orgunit は利用できません。
https://developers.worksmobile.com/jp/docs/directory

プランで利用できるAPIはこちらを参照してください
https://developers.worksmobile.com/jp/docs/auth-scope#line-works

### Directory API を用いてドメインのリソースを管理する場合には、Service Account 認証を用いる

LINE WORKSは2023年より以下のように仕様を変更しています。

>最高管理者を除く管理者は、通常のユーザーと同様に、ユーザーとして与えられた権限に基づいて、自身の情報など限定的なリソースに限り参照・管理ができます。与えられた管理権限に基づいてドメイン内のすべてのリソースにアクセスできるわけではありません。

https://developers.worksmobile.com/jp/news/detail?id=614

最初その事を気が付かずに、User Account 認証で実装して時間がかかってしまいました。

Service Account 認証で利用できないAPIについては以下記事を参照してください。
https://developers.worksmobile.com/jp/docs/auth-jwt#prohibited-api

### 指定するスコープはどれか1つで良い

複数のScopeが指定されている場合は、いずれか1つのScopeがあれば実行できます。
例えば以下のドキュメントではScopeに `user`, `user.read`, `directory`, `directory.read`とありますが、いずれか1つのScopeを指定すれば利用できます。

https://developers.worksmobile.com/jp/docs/user-get

### 開発ドキュメントのサポートは受けれないため LINE WORKS Developers Communityを活用する

LINE WORKSは開発系のサポートを対象外としています。LINE WORKS Developers Communityを活用しましょう。

https://forum.worksmobile.com/jp/

## Okta WorkflowからAPIを叩く

それではService Account 認証を用いて、POST /users を実行します。
今回は OktaWorkflow でLINEWORKSの認証を通してみます。

## 全体の流れ

1. 事前準備
2. JSON Web Token（JWT）の署名〜テスト認証の実装
3. 実行してみる

### 事前準備

あああ

### JSON Web Token（JWT）の署名

LINEWORKSでユーザー作成のAPIを実行するには、Acces Tokenが必要です。Acces Tokenの発行にはJWTを使用して署名する必要があります。
>Service Account 認証は、JSON Web Token (以降、JWT) を使用してアプリ専用の仮想管理者アカウントで認証を行い、Access Token を発行して API を利用する方法です。
https://developers.worksmobile.com/jp/docs/auth-jwt

:::
JSON Web Token (JWT) 生成時の `expires_in` は3600secにする
ドキュメントでは、`expires_in` は設定に従うと記載がありますが検証した際には設定に関わらず、3600secにしないとエラーになりました。
>有効期限は、Developer Console > API > ClientApp の [トークン設定 > Access Tokenの有効期限] の設定に従う。
>・1 hour (3600 秒)
>・24 hours (86400 秒)
>設定された有効期限が経過すると、自動的に満了する。
https://developers.worksmobile.com/jp/docs/auth-jwt#issue-access-token-response-body
:::

1. OktaWorkflowの関数からJWT > `Sign` を選び、以下の値を入力します
    - key : 秘密鍵
    - issuer : s
    - expiresin : `3600`
    - noTimeStamp : `False`
    - algorithm : HS256
2. 関数から `Construct` を選び、以下の値を入力します
    - grant_type : `urn:ietf:params:oauth:grant-type:jwt-bearer`
    - assertion : 1で作成した `Sign` の出力 `jwt`
    - client_id : scope
    - client_secret : a
    - scope : `directory`
3. Applicationから `API Connector` の `POST`を選び、以下の値を入力します
    - URL : `https://auth.worksmobile.com/oauth2/v2.0/token`
    - Query : `{}`
    - Headers : `{"Content-Type": "application/x-www-form-urlencoded"}`
    - Body : 2で作成した `Construct`の出力 `output`
    Connectionは使わないので何でも良いです

### 実行してみる

問題なければStausが200が返ってきます。
トリガーをUser Assigned to Applicationにすれば、LINEWORKSがアサインされたときにアカウント自動作成といったことができるようになりました。

## むすび

