---
title: "OktaWorkflowでLINE"
emoji: "⛳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
publication_name: "notahotel"
---


NOT A HOTELでは、顧客とのやりとりでLINE WORKSを利用するケースがあります。
しかし、LINE WORKSはSCIMに対応していないので、Okta Workflowsを用いてLINE WORKSのアカウント作成と削除を自動化しています。

本記事では、LINE WORKSのAPIをOkta Workflowから利用する際に、躓いた点を紹介しながら対応方法をステップ・バイ・ステップで紹介します。

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

### JSON Web Token (JWT) 生成時のAccess Tokenのexpires_inは3600secにする

ドキュメントでは、`expires_in`は設定に従うと記載がありますが検証した際には設定に関わらず、3600secにしないとエラーになりました。
>有効期限は、Developer Console > API > ClientApp の [トークン設定 > Access Tokenの有効期限] の設定に従う。
>・1 hour (3600 秒)
>・24 hours (86400 秒)
>設定された有効期限が経過すると、自動的に満了する。
https://developers.worksmobile.com/jp/docs/auth-jwt#issue-access-token-response-body

### 開発ドキュメントのサポートは受けれないため LINE WORKS Developers Communityを活用する

LINE WORKSは開発系のサポートを対象外としていますので、LINE WORKS Developers Communityを活用しましょう！
https://forum.worksmobile.com/jp/

## Okta WorkflowからAPIを叩く

それではService Account 認証を用いて、POST /users を実行します。
今回は OktaでLINE WORKSにアサインされたらLINE WORKSアカウントを作成するワークフローから一部を抜粋します。


### JWTのSign

LINEWORKSでユーザー作成のAPIを実行するには、Acces Tokenが必要です。Acces Tokenの発行にはJWTを使用する必要があります。
>Service Account 認証は、JSON Web Token (以降、JWT) を使用してアプリ専用の仮想管理者アカウントで認証を行い、Access Token を発行して API を利用する方法です。
https://developers.worksmobile.com/jp/docs/auth-jwt

JWTの生成とデジタル署名

## むすび

