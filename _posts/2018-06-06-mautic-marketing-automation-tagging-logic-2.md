---
layout: post
title: マーケティングオートメーション・ツールの名寄せロジックを「Mautic」に見る - その2 -
categories: [ オープンソース, マーケティングオートメーション ]
toc: true
comments: false
---
多くのマーケティングオートメーション・ツールでは、『個人情報の登録を終えているコンタクト（見込み顧客）』と『匿名コンタクト（見込み顧客）』を区分けしています。

## 「Mautic」の「Visitors」と「Standard contacts」

以前も触れたオープンソースマーケティングオートメーションソフトウェア「[Mautic](https://jp.mautic.org/)」では、下記の通りContactを区分けしています。 [^1]

- Visitors（以前は匿名のリード「Anonymous Leads」）
  - フォームや他の接触などでまだ特定されていないサイトへの訪問者。「Mautic」ではトラッキングはしますが通常は管理画面に表示されません。
- Standard contacts
  - フォームや他のソースなどで身元が特定できたコンタクトです。結果、名前、メールアドレスなどが典型ですが、個人を特定できるフィールドを持っています。

[先日の記事](/entry/2018/05/28/mautic-marketing-automation-tagging-logic-1.html)ではメールのクリック時にどういった実装になっているかを見ました。

今回はその他のコンタクト登録時の「名寄せ」ロジックについてどういった実装をしているかを見ていきます。

## 「Mautic」の「名寄せ」ロジック

「Mautic」でAPIにより名寄せを実施したいのは、典型的には「企業が保持している「Standard contacts」情報と同じメールアドレスでのフォーム送信があった」といった場合です。

このような場合について「Mautic」は「ユニークなフィールドが存在する場合にそれに基づいてコンタクト情報の重複がないかを確認し、重複するコンタクト情報について同一人物としてマージする」という方針を取っています。
このユニークなフィールドは複数指定も可能です。

実際にどのような実装によりこれを実現しているかを見ていきましょう。

### APIリクエストに対する名寄せ

APIリクエストに対しては、 ``LeadApiController.php`` の ``getExistingLead`` 関数で ``leadModel->checkForDuplicateContact`` 関数を呼び出しています。

<script src="http://gist-it.appspot.com/github/mautic/mautic/blob/blob/adc723d3a0bac3836ec37ccc6c94b0c4f3935ec4/app/bundles/LeadBundle/Controller/Api/LeadApiController.php?slice=54:70" ></script>

``checkForDuplicateContact`` 関数では、 ``uniqueFields`` を呼び出してそれに基づき ``existingLeads`` を読み込み、存在する場合には ``mergeLeads`` 関数を実行しています。

<script src="http://gist-it.appspot.com/github/mautic/mautic/blob/182a711ceae845f63343af04411c8883b04949b2/app/bundles/LeadBundle/Model/LeadModel.php?slice=923:985" ></script>

### フォーム送信に対する名寄せ

フォーム送信に対しては、 ``LeadModel.php`` の ``getContactFromRequest`` 関数で ``checkForDuplicateContact`` 関数を呼び出しています。

<script src="http://gist-it.appspot.com/github/mautic/mautic/blob/182a711ceae845f63343af04411c8883b04949b2/app/bundles/LeadBundle/Model/LeadModel.php?slice=875:906" ></script>

### 一括インポートに対する名寄せ

一括インポートにおいては、 ``LeadModel.php`` の ``import`` 関数で ``checkForDuplicateContact`` 関数を呼び出しています。

<script src="http://gist-it.appspot.com/github/mautic/mautic/blob/182a711ceae845f63343af04411c8883b04949b2/app/bundles/LeadBundle/Model/LeadModel.php?slice=1417:1461" ></script>

### ``checkForDuplicateContact`` 関数の処理

``checkForDuplicateContact`` 関数の内部では、下記の通りに処理が進みます。

1. コンタクト情報の「固有の識別子」欄が「Yes」になっているフィールドを取得。（このフィールドが同じコンタクトは、クッキーやIPアドレスなどが異なっても同一人物として扱われ、履歴情報などがマージされる仕様になっている）
1. これらユニークフィールドをキーに存在する既存リード情報がないかを検索
1. 既存リード情報が存在する場合は、フィールドごとに新しいリード情報優先で上書き

<script src="http://gist-it.appspot.com/github/mautic/mautic/blob/182a711ceae845f63343af04411c8883b04949b2/app/bundles/LeadBundle/Model/LeadModel.php?slice=976:985" ></script>

## まとめ

前回の記事で見た「メール文内のリンククリックによる名寄せ」はメールアドレスが名寄せのキーになるという点でシンプルでしたが、今回見てきた名寄せパターンでは、必ずしもメールアドレスがキーになるとは限りません。
このような場合に、「Mautic」はユニークなフィールドを企業側に指定させることで名寄せを実施していることを見てきました。
「名寄せ」に関する企業側の予測可能性を担保するという点で、これは非常に合理的な仕様のように思います。

[^1]: [Contacts | Mautic Documentation](https://www.mautic.org/docs/en/contacts/index.html)
