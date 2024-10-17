---
title: "Installomatorを使ってOkta Verifyを配布する"
emoji: "📦️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Jamf Pro, Okta]
published: false
publication_name: "visasq"
---

InstallomatorではOkta Verifyが未対応ですが配布する方法をMacAdminsで紹介されていたのでこちらで紹介します。



## 背景
Okta Verifyは毎月更新されており最新バージョンを維持していないと挙動が不安定になる場合があります。
そのため、macOSへのOkta Verifyの配布について公式ではVPPによる配布が推奨されています。
しかし、VPPの仕組みを用いたアプリケーションの更新は安定しておらず、他の手段を構築するのがJamf Proにおいては主流となっています。

代替手段として最も有名なものはInstallomatorです。
Jamf Proでは多くのユーザーがInstallomatorを中心にアプリケーションの配布をしているのではないでしょうか。

しかし、Okta VerifyはInstallomatorに対応していません。
Okta VerifyのpkgをダウンロードするURLは契約しているOktaのサブドメインが含まれているため、OSSでは扱えないという事情です。
https://subdomain-admin.okta.com/artifacts/OKTA_VERIFY_MACOS/[version]/OktaVerify-[version].pkg

本記事ではMac AdminsでIsaac-sanが解決した方法を紹介します。
https://macadmins.slack.com/archives/C013HFTFQ13/p1716324782514089


## 結論
Installomater.shに以下の記述を追記します。`{yoursubdomain}`に契約しているoktaのサブドメインに置換してください。
本スクリプトはアプリケーションのラベルがアルファベット順に並んでいるので、順番に合う場所に記述をしましょう。

```
oktaverify)
    name="Okta Verify"
    type="pkg"
    oktaTenant={yoursubdomain}.okta.com
    if [[ -z $oktaTenant ]]; then
        cleanupAndExit "Okta Tenant not set" ERROR
    fi
    downloadURL="https://$oktaTenant/api/v1/artifacts/OKTA_VERIFY_MACOS/download?releaseChannel=GA"
    appNewVersion=$(curl -is "$downloadURL" | grep ocation: | grep -o "OktaVerify.*pkg" | cut -d "-" -f 2)
    expectedTeamID="B7F62B65BN"
    ;;
```