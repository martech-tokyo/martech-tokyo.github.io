---
layout: post
title: マーケティングオートメーション・ツールの名寄せロジックを「Mautic」に見る - その1 -
categories: [ オープンソース, マーケティングオートメーション ]
toc: true
comments: false
---
匿名データを「リード」として管理・育成するマーケティングオートメーション・ツールでは、様々なチャネルから訪れるユーザーがどの過去訪問ユーザーと同一であるかを割り出す「タギング」の作業がとても重要となります。

## オープンソースのマーケティングオートメーション・ツール「Mautic」

オープンソースマーケティングオートメーションソフトウェア「[Mautic](https://jp.mautic.org/)」は、Github上でスター数が2,400件を超えています。

## 「Mautic」の「タギング」ロジック

一般に、マーケティングオートメーション・ソフトウェアは下記の2つの方法で見込み顧客情報とその見込み顧客が使用しているウェブブラウザ情報を紐づけます。

1. マーケティングオートメーション・ソフトウェアから配信されたメールのクリック
1. ウェブフォームの通過

ここでは「Mautic」が1の方法をどうやって提供しているかを見ていきます。

### 配信されたメールのクリックによるタギングの流れ

``ContactRequestHelper.php`` の ``getContactFromUrl`` 関数をみると、 ``ct`` というパラメータにクリックスルー用の情報を埋め込んでいます。
これを（GETリクエストの場合は）デコードしたうえで ``getContactFromEmailClickthrough`` 関数に引数として渡しています。

<script src="http://gist-it.appspot.com/github/mautic/mautic/blob/1f9ed5fd44a5b956e8ffe5684448c8d541d8954e/app/bundles/LeadBundle/Helper/ContactRequestHelper.php?slice=139:179" ></script>

``getContactFromEmailClickthrough`` 関数を見てみると、 ``emailStatRepository->findOneBy`` としてリポジトリから見込み顧客情報を取得してきていることがわかります。

<script src="http://gist-it.appspot.com/github/mautic/mautic/blob/1f9ed5fd44a5b956e8ffe5684448c8d541d8954e/app/bundles/LeadBundle/Helper/ContactRequestHelper.php?slice=181:218" ></script>

情報が存在する場合は ``mergeWithTrackedContact`` 関数を実行します。ここでは ``leadModel->mergeLeads`` 関数を実行します。

<script src="http://gist-it.appspot.com/github/mautic/mautic/blob/1f9ed5fd44a5b956e8ffe5684448c8d541d8954e/app/bundles/LeadBundle/Helper/ContactRequestHelper.php?slice=300:312" ></script>

``leadModel->mergeLeads`` 関数では、基本的に下記の方針のもとマージが実施されているようです。

- 上書きするリードと上書きされるリードの決定は、基本的にはデータの生成日（古いほうを新しいほうで上書き）
- 上書きにあたっては、新しいリードに存在する項目を上書き
- 但しポイントについては新しいリードのポイント加算（つまり新しいリードのほうはポイントとして増分ポイントだけ記載）

<script src="http://gist-it.appspot.com/github/mautic/mautic/blob/78b82c95dd28ae79df066e28fc376509cc9a2644/app/bundles/LeadBundle/Model/LeadModel.php?slice=1254:1348" ></script>

### デジタル・フィンガープリントも使用

先日も触れた通り、「[Mautic](https://github.com/mautic/mautic)」ではユーザー識別に「FingerPrint2」を使用してのデジタル・フィンガープリントも実施しています。

[http://martech-tokyo.github.io/entry/2018/05/19/browser-finger-print-now-from-fingerprint2:embed:cite]

``onBuildJs`` 関数で真っ先に（Hackしたバージョンの）Fingerprint2を記載しており、これが「Mautic」導入において貼り付けられたタグ内で発火します。

<script src="http://gist-it.appspot.com/github/mautic/mautic/blob/322964b3578f63cf40c88d4a67922684720388ba/app/bundles/CoreBundle/EventListener/BuildJsSubscriber.php?slice=37:62" ></script>

## まとめ

「Mautic」はタギングのロジックなどにおいてオーソドックスな方法を示してくれています。このあたりはオープンソースならではかと思います。

## 続きの記事はこちら

[http://martech-tokyo.github.io/entry/2018/06/06/mautic-marketing-automation-tagging-logic-2:embed:cite]
