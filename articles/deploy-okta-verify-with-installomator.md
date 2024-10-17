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
Okta Verifyは毎月更新されておりアプリの更新を自動的に強制することを推奨しています。
そのため、macOSへのOkta Verifyの配布についてApple Business Managerを用いた配布（以下、VPP）が紹介されています
https://help.okta.com/oie/ja-jp/content/topics/identity-engine/devices/ov-install-options-macos.htm

しかし、VPPの仕組みを用いたアプリケーションの更新は安定しておらず、他の手段を構築するのがJamf Proにおいては主流となっています。

代替手段として最も有名なものはInstallomatorです。
Jamf Proでは多くのユーザーがInstallomatorを中心にアプリケーションの配布をしているのではないでしょうか。
https://github.com/Installomator/Installomator

しかし、Okta VerifyはInstallomatorに対応していません。
Okta VerifyのpkgをダウンロードするURLは契約しているOktaのサブドメインが含まれているため、OSSでは扱えないという事情です。

>https://subdomain-admin.okta.com/artifacts/OKTA_VERIFY_MACOS/[version]/OktaVerify-[version].pkg

## 本題
本記事ではMac AdminsでIsaac-sanが解決した方法を紹介します。
https://macadmins.slack.com/archives/C013HFTFQ13/p1716324782514089


## 結論
installomater.shに以下の記述を追記します。`{yoursubdomain}`に契約しているoktaのサブドメインに置換してください。
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
