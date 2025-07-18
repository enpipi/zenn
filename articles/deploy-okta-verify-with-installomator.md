---
title: "Installomatorを使ってOkta Verifyを配布する"
emoji: "📦️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Jamf Pro, Okta]
published: true
publication_name: "visasq"
---

:::message
本記事よりも、良いやり方が紹介されていました！
https://kenchan0130.github.io/post/2024-12-09-2
:::

InstallomatorではOkta Verifyが未対応ですが、配布する方法がMacAdminsで紹介されていたので、こちらで紹介します。

## 背景
Okta Verifyは毎月アップデートがリリースされており、アプリの更新を自動的に強制することを推奨しています。
そのため、macOSへのOkta Verifyの配布についてApple Business ManagerのVolume Purchase Program（以下、VPP）を用いた配布が紹介されています。
https://help.okta.com/oie/ja-jp/content/topics/identity-engine/devices/ov-install-options-macos.htm

VPPの仕組みを用いたアプリケーションの更新は安定しておらず、他の手段を構築するのがJamf Proにおいては主流となっています。
代替手段として最も有名なものはInstallomatorです。

Jamf Proでは多くのユーザーがInstallomatorを使ってアプリケーションの配布をしているのではないでしょうか。
https://github.com/Installomator/Installomator

しかし、Okta VerifyはInstallomatorに対応していません。
Okta VerifyのpkgをダウンロードするURLには契約しているOktaのサブドメインが含まれているため、OSSでは扱えないという事情があります。
>`https://subdomain-admin.okta.com/artifacts/OKTA_VERIFY_MACOS/download?releaseChannel=GA`

## 本題
本記事では、Mac AdminsでIsaac-sanが解決した方法を紹介します。
https://macadmins.slack.com/archives/C013HFTFQ13/p1716324782514089

## 免責
スクリプトの修正および使用は、すべて自己責任でお願いします。

## 結論
installomater.shに以下の記述を追記します。`{yoursubdomain}`を契約しているOktaのサブドメインに置換してください。
本スクリプトはアプリケーションのラベルがアルファベット順に並んでいるので、順番に合う場所に記述をしましょう。

```shell
oktaverify)
    name="Okta Verify"
    type="pkg"
    oktaTenant={yoursubdomain}.okta.com
    BLOCKING_PROCESS_ACTION=ignore
    if [[ -z $oktaTenant ]]; then
        cleanupAndExit "Okta Tenant not set" ERROR
    fi
    downloadURL="https://$oktaTenant/api/v1/artifacts/OKTA_VERIFY_MACOS/download?releaseChannel=GA"
    appNewVersion=$(curl -is "$downloadURL" | grep ocation: | grep -o "OktaVerify.*pkg" | cut -d "-" -f 2)
    expectedTeamID="B7F62B65BN"
    ;;
```

:::message
~~Setup Managerでは利用できません。Setup Manager内部にInstallomatorが内蔵されているためです。~~
Setup ManagerでもJamfのポリシーを使えば利用できるとのことでした。 @hikkyさんご指摘ありがとうございました！
:::
