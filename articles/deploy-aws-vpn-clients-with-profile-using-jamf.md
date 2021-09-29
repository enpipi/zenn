---
title: "Jamfを使ってAWS VPN Clientをプロファイル付きで配布する"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [AWS,AWSClientVPN,Jamf]
published: false
---

## はじめに
AWS VPN Clientを配布する際にシュッとプロファイルも一緒に配布したいなと思って作りました。

:::message alert
- 2021年8月時点でプロファイルは更新ではなく、 **上書きです**
- 2021年8月時点で **SAML認証のプロファイルのみ** 対応しています
:::

## 成果物
github url


## 配布準備
今回のスクリプトはAWS VPNがインストール済みであることが前提なので、以下の順番で配布するポリシーを作るのが最終目標です。

1. AWS VPN Clientの配布
2. プロファイルの配布

:::message
AWS VPN Clientとプロファイルを1つのポリシーにしても上手くいくかもしれません。
自分の場合は、インストールする順番を明確に定義したほうが確実なので今回はこのようなやり方にしています。
:::


### AWS VPN Clientを配布するトリガーを作る
JamfでAWS VPN Clientを配布するポリシーを作ります。
AWS VPN Clientの配布にはInstallomaterを使いました
triggerに `install_aws_vpn_client` と記載し、`Ongoing` にします。
今回は、ターゲットを `All Computer` にしています。


### .ovpnファイルの用意
1. AWS VPN Endpointから .ovpnファイルをダウンロードします。
2. `./private/temp/`に該当の.ovpnファイルを設置します。
3. `Composer` を起動し、2のファイルをpkgのところにドラッグアンドドロップします
4. hogehogeを選択する
5. pkgで書き出す
6. 書き出したpkgをJamf Adminでアップロードします



### プロファイルを配布するトリガーを作る
1. Github urlのソースをScriptに貼ります
2. 新規ポリシーでスクリプトをアサインします
3. 引数に以下の情報を入力します
4. カスタムトリガーに `install_aws_vpn_profile` と記載し、`Ongoing` にします
5. 今回は、ターゲットを `All Computer` にしています。


### AWS VPN Client と プロファイルを配布するためのスクリプトとポリシーを用意する
次のようなスクリプトで、ポリシーを作ります。

```bash
#!/bin/sh
/usr/local/jamf/bin/jamf policy -trigger install_rosseta
/usr/local/jamf/bin/jamf policy -trigger install_aws_vpn_client -verbose
/usr/local/jamf/bin/jamf policy -trigger install_aws_vpn_profile -verbose
```

:::message
M1を考慮して、 Rossetaの配布ポリシーも混ぜています。
Rossetaインストールのスクリプトは下記を使っています。
https://www.jamf.com/jamf-nation/discussions/37357/deploy-rosetta-on-m1-machines-before-everything-else#responseChild209441
:::

### おわりに
VPNのプロファイルには秘密鍵の情報もあるし、なるべくユーザーには意識させないことに越したことはありません。また、AWS VPNをSSOにしておけばIdP側で認可をコントロールできるので運用コストも最小限で済みますね。

肝心のスクリプトは、諸先輩方からみると甘いところも多々あると思いますし、自分も幾つかイケてないと感じてる箇所もあるので今後も更新したいです。動作で不明点があればお気軽にTwitterなどでDMいただければ幸いです。
