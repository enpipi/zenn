---
title: "Oktaのプロビジョニング失敗を検知する"
emoji: "🔔"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
publication_name: "NOT A HOTEL"
---

## Oktaのプロビジョニングがイベントフックに対応した

2025年12月頃にアプリのプロビジョニングとインポートに関するイベントが、イベントフックの対象になりました。
- `application.provision.user.push_profile`
- `application.provision.user.push`
- `application.provision.user.reactivate`
- `application.provision.user.import_profile`
- `app.user_management.user_group_import.upsert_success`

https://developer.okta.com/docs/release-notes/2025-okta-identity-engine/#events-for-app-provisioning-and-import-changes-are-now-event-hook-eligible

:::message
現在利用可能なEvent Hookは以下より確認できます
https://developer.okta.com/docs/reference/api/event-types/?q=event-hook-eligible
:::

## Oktaのプロビジョニング失敗を検知したい

Oktaのプロビジョニングはいくつかの理由で失敗しますが、検知がしにくい問題がありました。
今回はアカウント作成時と、プロファイル属性の更新時を検知する Okta Workflowを作ります。

## Okta Workflowsで、API Endpoint トリガー を作る

詳しい説明は、 クラウドネイティブさんの記事が参考にしてください。
本記事でも参考にしております。ありがとうございます。

https://blog.cloudnative.co.jp/28292/

API Endpoint トリガーの `Invoke URL` を次の工程で使います。

## イベントフックを作る

:::message
事前にイベントフックのカスタムフィルターを有効にしてください
https://help.okta.com/oie/ja-jp/content/topics/automation-hooks/event-hooks-filters.htm
:::

Workflow > Event Hooks に移動し、「Create Event Hook」を選択します。

- URL: 先ほどコピーしたWorkflowsの `Invoke URL` 
- Subscribe to events: `Push user's profile to external application.`, `User synced from external application` を選択

フィルターは、どちらも `event.outcome.result eq 'FAILURE'` とします

### 余談： イベントフックでトリガーされるイベントについて

今回、一番取得したいログは `serverty eq "INFO"  && event.outcome.result eq 'FAILURE'` ですが、フィルターをオフにしてもトリガーされませんでした。

このログにしか、失敗した理由が記載されていないためです。

結局以下、二つしかトリガーせずでした。

- `serverty eq "INFO"  && event.outcome.result eq 'success'`
- `serverty eq "ERROR"  && event.outcome.result eq 'FAILURE'`

よって、今回は `event.outcome.result eq 'FAILURE'`のイベントフックをトリガーにOkta Workflowsでログを取得し直すという方針になっています。

## Okta Workflowsを作る

全体像はこちらです

### 1. API Connector - Close

前述のクラウドネイティブさんの記事を参考にいれています

- Status Code : `200`

### 2. JSON - Parse

API Connector の Bodyを変数として扱えるようにします

- Input : トリガー：API Endpoint の `body`
- Output : `output`

### 3. Object - Get Multiple

- Input : 2の `output`
- Output :
 - `data.events.0.transaction.id`
 - `data.events.0.eventType`


### 4. Text - Compose

- Input : `transaction.id eq {data.events.0.transaction.id} and outcome.result eq "FAILURE"`
- Output : `custom filter` と名称変更

### 5. 