---
title: "Keeper Commanderで管理者承認を自動化する方法"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Keeper]
published: false
---


# Keeper Commanderで管理者承認を自動化する方法

## 1. はじめに

Keeperは、セキュリティを理由に新規デバイスの場合等で管理者による承認が必要となります。

>### 5. SSOプロバイダを保護する
>
>KeeperをSSO IDプロバイダと連携させる場合は、IDプロバイダに多要素認証ポリシーが設定され、権限が制限されているようにしてください。IDプロバイダからの指示と最善の対応策に従い、管理者アカウントには必要最低限の特権のみ付与されているようにします。
>
>Keeperオートメイターサービスを使用すると、クラウドSSOを使用するユーザーが新しいデバイスまたは未承認のデバイスでボルトにアクセスする際の処理がスムーズになります。これにより利便性は向上しますが、SSO IDプロバイダを不正アクセスから保護する必要もあります。Keeperオートメーターサービスを有効にすると、IDプロバイダによる認証とユーザープロビジョニング処理を完全に信頼することになりますので、セキュリティを強化するために自動デバイス承認を特定のIP範囲に制限したり、無効の状態にしてユーザーに新しいデバイスを手動で承認させることもできます。

https://docs.keeper.io/jp/enterprise-guide-jp/recommended-security-settings#id-5-ssopurobaidawosuru

### 1.1. Keeperオートメーターサービスは構築が手間

上記引用にもあるようにKeeperオートメーターサービスを用いれば管理者承認をバイパスできます。しかし、Google Cloud Run等で環境を構築する必要があるため、環境構築に一定の工数が必要となります。

https://docs.keeper.io/jp/keeper-sso-connect-cloud-jp/device-approvals/automator

## 2. そうだ、Keeper Commanderがあるじゃない

Keeper CommanderはKeeperに対するコマンドラインおよびSDKのインターフェースです。Keeper Vaultへのアクセスと制御、管理機能 (エンドユーザーのオンボーディングやデータのインポート/エクスポートなど) の実行等に利用できます。

Keeper CommanderはX秒ごとに承認を実行する自動化モードがサポートされています。
この機能により、ローカル環境で自動承認を実行できるため必要なときだけ、自動承認ができるようになります。

オンボーディングや、Keeper導入時など多くのユーザーのデバイスを承認しなければならない場合に効果的です。

https://docs.keeper.io/jp/keeper-sso-connect-cloud-jp/device-approvals/commander-cli

## 3. 導入手順

本章では、macOSを例にKeeper Commanderの導入手順を記します。
大まかに次の工程で説明します。

- Keeper Commander CLIのインストールとログイン
- 自動承認のconfig.jsonを構築

### 3.1. 前提条件

macOSでKeeper Commander CLIを利用するには、以下の環境が必要です：

- macOS 10.15以降
- Python
- pip3

ご利用する環境ごとに必要な要件が記載されていますので、次の記事を参照して要件とインストール方法をを確認してください。

https://docs.keeper.io/jp/secrets-manager-jp/commander-cli/commander-installation-setup

### 3.2. Keeper Commander CLIにログインする

1. ターミナルで、`keeper shell`と実行すると、ログインユーザーを聞かれます。
2. SSOしている場合は、URLが発行されます。
   1. URLをブラウザにコピーペーストして、表示されるトークンをコピーしたら、ターミナルに戻り`selection` にペーストするとログインできます
3. 管理者承認のため、`2` , `r` を入力
4. ログインに成功すると、`Not logged in>` が `[My Vault>` に変わります

ターミナルでの流れ

```shell
Not logged in> login
...      User(Email): test@exapmle.com
Logging in to Keeper Commander
SSO Login URL:
https://keepersecurity.jp/api/rest/sso/saml/login/xxxxxxxxxxxxxxxxxxx
Navigate to SSO Login URL with your browser and complete login.
Copy a returned SSO Token into clipboard.
Paste that token into Commander
NOTE: To copy SSO Token please click "Copy login token" button on "SSO Connect" page.
  a. SSO User with a Master Password
  c. Copy SSO Login URL to clipboard
  o. Navigate to SSO Login URL with the default web browser
  p. Paste SSO Token from clipboard
  q. Quit SSO login attempt and return to Commander prompt
Selection: {ここにトークンをペーストする}
Approve this device by selecting a method below:
  1. Keeper Push. Send a push notification to your device.
  2. Admin Approval. Request your admin to approve this device.
  r. Resume SSO login after device is approved.
  q. Quit SSO login attempt and return to Commander prompt.
Selection: 2
Selection: r
Successfully authenticated with SSO Login
Syncing...
```

### 3.3. 自動承認用の `config.json` ファイルを設定する

公式のドキュメント通り、30秒ごとの自動承認を組みます。
`config.json` を開き次の記述を追記します。

```json
"commands":["device-approve --reload","device-approve --approve"],
"timedelay":30
```

:::message
`config.json` の設置箇所
- macOS: `~/.keeper/config.json`
- Windows: `C:\Users\Administrator.keeper\config.json`
:::

`config.json` は次のようになります。

```json
{
  "user": "test@exapmle.com",
  "server": "keepersecurity.jp",
  "device_token": "xxxxxxxxxxxxx",
  "private_key": "xxxxxxxxxxxxx",
  "clone_code": "xxxxxxxxxxxxx",
  "commands": [
    "device-approve --reload",
    "device-approve --approve"
  ],
  "timedelay": 30
}
```

`device-approve --approve --trusted-ip`オプションを使用することで、過去に正常なログインが認識されたIPからのデバイスをすべて承認することも可能です。

### 3.4. 躓いたところ

`"メールアドレス" have no accepted account transfers or did not share encryption key`
と表示されて、管理画面からも承認・拒否できない場合がありました。
明確な原因はわからないのですが、移管の同意を設定していてアカウント招待からしばらくサインアップがなかった場合に再現しやすいようです。

本件が発生したら、対象アカウントを削除して再招待が必要です。

## 4. 終わりに

Keeperのデバイス承認により、仮にIdPが不正アクセスされてもKeeperは不正アクセスされずに済みます。オンボーディングや、Keeper導入時等の限定的なシチュエーションで自動承認を用いることは、業務効率の向上に有効な手段となります。
