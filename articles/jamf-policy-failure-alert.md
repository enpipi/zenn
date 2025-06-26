---
title: "Jamf Proのポリシー実行に失敗したらSlackに通知する"
emoji: "😺"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Jamf Pro]
published: false
publication_name: "notahotel"
---

NOT A HOTEL Biz Techチームの@enpipiです。

Jamf Proでポリシーを設定した後、放置されがちですが、イベントフックを活用することでポリシーの失敗を検知できるようになります。
この記事では、通知によって得られた成果と、Zapierを使ってJamf Proのポリシー実行失敗をSlackに通知する方法を紹介します。

## この記事でわかること

- Jamf Proのポリシー失敗を自動検知する方法
- Zapierを使ったSlack通知の実装手順

## 対象読者

- Jamf Proを運用している管理者

## なぜ失敗を通知するようにしたか？

NOT A HOTELは各拠点で働くメンバーを除いて大半がフルリモートで働いており、入社日も固定で決めずにいつでも入社できるようにしています。
そのためキッティングはセルフサービスで対応いただくようにしており、人によってはアプリケーションがうまく配布されていなかったり、想定と異なる挙動をしていたりすることがありました。
例えばアプリケーションの配布に失敗してもユーザーは自分でインストールして終わらせてしまうので、管理者である我々も気がつきにくい状況でした。

## 成果

2月中旬からポリシーの失敗をSlackに通知するようにしました。
結果、配布方法の見直しや設定ミスの修正が行われ、容量の大きい特定のアプリケーションの配布だけが通知されるようになりました。

具体的な成果は以下の通りです。

- ポリシーが意図しないデバイスに適用されている、SmartComputerGroupでやるべきScopeをStaticGroupで実行していたといった設定ミスを検知できた
- 使用していないポリシーが動作していることが判明し、ポリシーの棚卸しにつながった
- Installomatorのバージョンが古いことが原因で、特定のアプリケーションの配布に失敗していたことを検知できた。
- 頻繁に失敗するアプリケーションがあり、配布方法を見直すことで失敗の頻度を大幅に削減できた

## 実装

ここからは実装方法の一部を紹介します。
Zapierを使って実装しており、Slackからすぐに情報にアクセスできるように、対象のポリシー・デバイス・実行者のリンクを取得できるようにしました。

### 全体の流れ

1. Jamf Proでポリシーの実行完了をWebhookで通知する
2. 失敗以外の通知をFilterする
3. Webhookに含まれるPolicy IDからポリシーの名称を取得
4. ポリシー、デバイス、実行者のURLをSlackに通知

### 1. Jamf Proでポリシーの実行完了をWebhookで通知する

1. 設定 \> グローバル \> Webhookを開く
2. Webhookイベントを「ComputerPolicyFinished」とする
3. その他の設定は、Webhookの通知先の仕様に合わせてください

### 2. 失敗以外の通知をFilterする

以下は実際にポリシー実行完了時のWebhookのペイロードの例です。
ポリシーの成功可否は `event` の `successful` で判定します。

```json
{
  "webhook": {
    "id": 4,
    "name": "ComputerPolicyFinished",
    "webhookEvent": "ComputerPolicyFinished",
    "eventTimestamp": 1741597077290
  },
  "event": {
    "policyId": 37,
    "successful": false,
    "computer": {
      "udid": "XXXXXXX-XXXX-XXXX-XXXX-XXXXXXXX",
      "deviceName": "M-XXXXXX",
      "model": "MacBook Air (M2, 2022)",
      "macAddress": "XX:XX:XX:XX:XX:XX",
      "alternateMacAddress": "XX:XX:XX:XX:XX:XX",
      "serialNumber": "XXXXXXX",
      "osVersion": "15.3.1",
      "osBuild": "24D70",
      "userDirectoryID": "-1",
      "username": "test@example.com",
      "realName": "Test User",
      "emailAddress": "test@example.com",
      "phone": "",
      "position": "",
      "department": "",
      "building": "",
      "room": "",
      "ipAddress": "xx.xx.x.xx",
      "reportedIpAddress": "xxx.xxx.xx.xxx",
      "jssID": 51,
      "managementId": "XXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXX"
    }
  }
}
```

### 3\. Webhookに含まれるPolicy IDからポリシーの名称を取得

ポリシーの名称を取得するには、以下のAPIエンドポイントを使用します。
`https://{yourdomain}.jamfcloud.com/JSSResource/policies/id/{id}`
[Finds policies by ID](https://developer.jamf.com/jamf-pro/reference/findpoliciesbyid)

### 4\. ポリシー、デバイス、実行者のURLをSlackに通知

Slackのフォーマットに合わせて次のようにしています。

```markdown
次のポリシーの実行に失敗しました。

> - *ポリシー:* <https://{yourdomain}.jamfcloud.com/policies.html?id={id}|{Policy Name}>
> - *実行デバイス:* <https://{yourdomain}.jamfcloud.com/computers.html?id={id}|{Computer Device Name}>
> - *実行者:* {Computer Email Address}
```

## まとめ

Jamf Proのポリシー失敗を通知することで、潜在的な問題を検知できるようになりました。
MDMは一度設定した項目は基本的に触らなくなるため、通知駆動での課題検知は相性がとても良いです。

入社したての頃にこのタスクを実施したので、NOT A HOTELのJamf環境を効率よくキャッチアップすることができました。
皆さんもポリシーの失敗を通知するようにすると、想定外の失敗を見つけることができると思います。
ぜひ参考にしてみてください。
