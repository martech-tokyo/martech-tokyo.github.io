---
layout: post
title: 「FingerPrint2」に見るブラウザフィンガープリントの最新事情 - その1 -
image: assets/images/2018-05-19-browser-finger-print-now-from-fingerprint2.png
categories: [ オープンソース, アクセス解析 ]
toc: true
comments: false
---
AppleのSafari11.0から搭載されたITP（Intelligent Tracking Prevention）では、サードパーティのCookieを使った広告配信やサイトトラッキングに大きな制限が設けられました。
プライバシー保護の観点からCookieの使用が制限される流れがあるなか、Cookieや端末IDに依存しないユーザー識別を行う技術として『ブラウザー・フィンガープリント』が最近改めて注目を浴びています。

## ブラウザー・フィンガープリントを可能にするJavaScriptライブラリ「FingerPrint2」

ブラウザー・フィンガープリントを可能にするライブラリ「[FingerPrint2](http://valve.github.io/fingerprintjs2/)」は、Github上でスター数が5,000件を超えるオープンソースの人気ライブラリです。

Twitter上でも称賛とともに言及が多く、オープンソースのブラウザフィンガープリントライブラリとしてはデファクトといえるのではないかと思います。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="en" dir="ltr">How to track a user with <a href="https://twitter.com/hashtag/javascript?src=hash&amp;ref_src=twsrc%5Etfw">#javascript</a> : <a href="https://twitter.com/hashtag/FingerprintJS2?src=hash&amp;ref_src=twsrc%5Etfw">#FingerprintJS2</a> <a href="https://t.co/ad19EJpWMh">https://t.co/ad19EJpWMh</a> ... Terribly efficient and unstoppable !</p>&mdash; Paul Guilbert (@GuilbertPaul) <a href="https://twitter.com/GuilbertPaul/status/694980754183450624?ref_src=twsrc%5Etfw">2016年2月3日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## デジタル・フィンガープリントとは

フィンガープリント（finger print）とは、日本語に直すと「拇印」や「指紋」という意味で、デジタル情報が改ざんされていないことを証明するために様々な場面で使われています。
例えば、電子メールに使用されるデジタル署名は、元のメッセージのフィンガープリントを暗号化したもので、受信者は暗号化したフィンガープリントを復元し、受け取ったメッセージから計算したフィンガープリントと照合することで、メッセージが改竄されていないか確認することができます。

ブラウザー・フィンガープリントにおいては、Webサイトを閲覧したブラウザに関してサイト側が把握できる情報（バージョンや画面サイズ、各種設定など）を掛け合わせることで、そのブラウザに特有のハッシュ値を作成します。
その違いに基づいて、Cookieなどに頼らなくても、どのユーザーかを特定するということです。

## 「FingerPrint2」における実現方法

「FingerPrint2」においては把握できる情報として、下記の27種類の情報を利用しているようです。

1. ユーザーエージェント（ブラウザ名・ブラウザバージョンなどの情報を含む）
2. ブラウザの使用言語
3. 画素数
4. スクリーンのサイズ
5. タイムゾーン
6. セッションストレージの有無
7. ローカルストレージの有無
8. IndexedDBの有無
9. 「AddBehavior」メソッドを持っているか
10. OpenDBの有無
11. CPU情報
12. OS
13. DoNotTrack機能の使用有無
14. Flashによりインストールされているフォントのリスト（デフォルトでは無効）
15. インストールされているフォントのリスト（JS/CSSにより検知）
16. Canvasフィンガープリント
17. WebGLフィンガープリント
18. ブラウザプラグイン
19. アドブロックのインストール有無
20. ユーザー自身による使用言語設定情報の改ざん有無
21. ユーザー自身によるスクリーンサイズ情報の改ざん有無
22. ユーザー自身によるOS情報の改ざん有無
23. ユーザー自身によるブラウザ情報の改ざん有無
24. タッチスクリーン機能の有無
25. ピクセル比
26. ユーザーエージェントが使用可能なプロセッサーの数
27. デバイスのメモリサイズ（W3Cで[Device memory](https://w3c.github.io/device-memory/)仕様が議論）

## 「FingerPrint2」を試してみる

### デモサイトから試してみる

デモサイトで「Get my fingerprint」をクリックしてみると自ブラウザのフィンガープリントが表示されます。

下記、Google Chrome/Firefox/Safari/Operaでそれぞれ実行してみた結果です。ブラウザによって、それぞれの画面中央に異なるフィンガープリントの値が出力されていることがわかります。

![Browsers](/assets/images/2018-05-19-browser-finger-print-now-from-fingerprint2.png)

Google Chromeでプライベートウィンドウと通常ウィンドウそれぞれで実行してみました。この場合も、それぞれ異なるフィンガープリントが出力されます。

![Chrome](/assets/images/2018-05-19-browser-finger-print-now-from-fingerprint2-chrome.png)

プライベートウィンドウと通常ウィンドウでフィンガープリントが異なるのは一見不思議に思えますが、処理の内容を確認したところでは「Canvasフィンガープリント」の結果が異なってくるようです。

### JavaScriptを実際に実行してみる

本体のJavaScriptファイルを読み込んだうえで、

```html
<script src="//cdnjs.cloudflare.com/ajax/libs/fingerprintjs2/1.8.0/fingerprint2.min.js"></script>
```

下記の通り実行すればブラウザー・フィンガープリントによるハッシュ値が取得できます。

```javascript
<script>
new Fingerprint2(options).get(function(result) {
  console.log(result); // "19fc468160e8ab20022fc83555ee7528"などとDeveloper Console上に表示される
})
</script>
```

下記のような処理をすれば、フィンガープリントの元になっている情報のそれぞれを確認することも可能です。

```javascript
<script>
new Fingerprint2(options).get(function(result,components) {
  console.log(components);
})
</script>
```

## ブラウザー・フィンガープリントの技術深化の背景

ブラウザー・フィンガープリントは長らく「実使用に耐えない技術」との印象が強かったように思います。
実際、ブラウザー・フィンガープリント技術自体のバリエーションの少なさもあり、ガラケーなどの貧弱なブラウザにおいては多くのユーザーの間でハッシュ値が重複してしまい、サイト訪問者の識別には使うことができませんでした。

しかし「FingerPrint2」の実装をみる通り、そのような状況も改善されているようです。背景には、

- ブラウザの高機能化により、識別の元に使える情報が増加したこと
- それを追従する形での『ブラウザー・フィンガープリント』技術が深化してきたこと（特に、インストールされているフォントのリストをJS/CSSにより検知するという発想は興味深いものがあります）

があるように思います。

## Cookieとの特徴比較

とはいえCookie（もしくはセッションストレージ／ローカルストレージへの情報保持。以下同じ）と比べると、一長一短あるのも事実です。下記に整理してみました。

|                                           | Cookie       | フィンガープリント |
|:------------------------------------------|:------------:|:---------------:|
| サイト訪問者識別の一意性                      | ◯            | △ 　　　         |
| サイト訪問者識別の持続性                      | ◯            | △ 　　 　        |
| 使用制約の厳しさ（クロスドメイン／アドブロック）  | △            | ◯ 　　　         |
| データ保護ポリシーへの配慮必要性（GDPR）        | △            | △ 　　　         |

サイト訪問者識別の持続性に関しては、例えば下記のように、元のデータが変更されてしまうと、当然ながらフィンガープリントを照合しても何の意味もなくなります。

- ブラウザのバージョン更新
- 出力ディスプレイの変更などによる画面サイズの更新
- ブラウザの設定の変更

最近のブラウザは自動更新により頻繁にバージョンが変更となります。そのため、ユーザーユニークなCookieの代替としての使用は難しく、セッションユニークな（訪問ごとの）Cookieの代替としての利用が現実的でしょう。

## まとめ：Cookieとの補完により、より正確なユーザートラッキングが可能に

2018年5月25日に迫るEUのGDPR（General Data Protection Regulationの頭字語で、EUの一般データ保護規則のこと）施行が目下、業界に震撼を与えています。 [^1]
今後も、

- アドブロックの使用普及
- ITPなどに見られるクッキーや端末ID利用の制限強化

など、ユーザー側（や政府）がプライバシーを意識するようになる、事業者側がそれへの配慮を求められるようになる流れは不可避と思われます。

このような流れのなかで、以上のようなCookieとの特徴の差異を踏まえ、ブラウザー・フィンガープリントをCookieと補完させながら使用することは有用ですし、今後のMarTechにおいて一般的になっていくように思います。

ちなみに、同様の考え方から、オープンソースのMAツール「[Mautic](https://github.com/mautic/mautic)」でもユーザー識別にまさに「FingerPrint2」が使用されています。こちらも記事を改めて触れてみたいと思います。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="en" dir="ltr">fingerprintjs2 integrations in Mautic 1.4 is my favourite feature. Don&#39;t forget set field as Unique Identifier <a href="https://t.co/uwzkEsiuJk">pic.twitter.com/uwzkEsiuJk</a></p>&mdash; Kuzmany.biz (@kuzmany_biz) <a href="https://twitter.com/kuzmany_biz/status/730474638153793536?ref_src=twsrc%5Etfw">2016年5月11日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

[^1]: EUのGDPRでは個人データとは個人を特定し得るすべての記録と定義され、デジタルフィンガープリントも対象。参考：[Art. 4 Nr. 1 GDPR](http://www.privacy-regulation.eu/en/article-4-definitions-GDPR.htm)

## 続きの記事はこちら

[http://martech-tokyo.github.io/blog/entry/2018/07/07/browser-finger-print-now-from-fingerprint2-2:embed]
