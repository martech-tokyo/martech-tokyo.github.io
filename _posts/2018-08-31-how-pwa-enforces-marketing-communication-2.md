---
layout: post
title: 「Progressive Web Apps」はエンゲージメントを加速するか - その2 -
image: assets/images/2018-08-31-how-pwa-enforces-marketing-communication-2.jpg
categories: [ PWA, Webトレンド ]
toc: true
comments: false
---
[前回記事](/entry/2018/08/05/how-pwa-enforces-marketing-communication)で触れた「Progressive Web Apps」、略して「PWA」。

今回記事では、「基本的なPWAの仕組み」と「実際に本格的に実装・提供するための技術スタック」について触れたいと思います。

## 基本的な「PWA」の仕組み

PWAの実現に最低限必要なものは下記の3つのみです。

- Web App Manifest
- Service Workers
- SSL

1. Web App Manifest（manifest.json）

これは、Webアプリケーションに関するメタ情報を提供するjsonファイルです。

```json
{
  "name": "サンプルPWA",
  "short_name": "SPWA",
  "theme_color": "#2196f3",
  "background_color": "#2196f3",
  "display": "standalone",
  "icons": [
      {
          "src": "/img/icon/144.png",
          "sizes": "144x144",
          "type": "image/png"
      }
  ],
  "scope": "/",
  "start_url": "/"
}
```

上記のようなJSONファイルにより、下記などの情報を設定することができます。 [^1]

- アプリのアイコン画像（icons）
- アプリの背景色（background_color）
- アプリの名前、短縮名（name, short_name）
- 表示タイプのカスタマイズ（display）
![display](https://developers.google.com/web/fundamentals/web-app-manifest/images/manifest-display-options.png?hl=ja)
- ページの最初の向きが縦表示か横表示か（orientation）
![orientation](https://developers.google.com/web/fundamentals/web-app-manifest/images/manifest-orientation-options.png?hl=ja)

このマニフェストファイルは自分で書くことも、[ツールを使って画面上で生成する](https://app-manifest.firebaseapp.com/)こともできます。
これをウェブサイトのトップディレクトリに配置し、index.html等で下記の通り指定します。

```html
<link rel="manifest" href="/manifest.json">
```

2. Service Workers（serviceworker.js）

サービスワーカーは、アプリケーションのバックグラウンドで実行され、ネットワークとアプリケーションの間のプロキシとして機能する、イベントドリブンのワーカーです。
ネットワーク要求を傍受することで、バックグラウンドで私たちの情報をキャッシュすることができ、オフラインでの使用時にはそこからデータがロードされます。

例えば下記のようなjavascriptスクリプトにより、フェッチやインストールのようなイベントを待ち受け、タスクを実行します。 [^2]

```javascript
self.addEventListener('fetch', event => {
    //caching for offline viewing
    const {request} = event;
    const url = new URL(request.url);
    if(url.origin === location.origin) {
        event.respondWith(cacheData(request));
    } else {
        event.respondWith(networkFirst(request));
    }
});
async function cacheData(request) {
    const cachedResponse = await caches.match(request);
    return cachedResponse || fetch(request);
}
```

これをHTMLから呼び出します。 [^3]

```html
<script>
  // service workerが有効なら、service-worker.js を登録します
  if ('serviceWorker' in navigator) {
    navigator.serviceWorker.register('./serviceworker.js').then(function() { console.log('Service Worker Registered'); });
 }
</script>
```

3. SSL

PWAになるためには、Webアプリケーションは安全なネットワークを介して提供されなければなりません。
https, localhostでしかService Workerは動かないことで常に安全な状態を保つためです。

Cloudfareや[LetsEncrypt](https://letsencrypt.org/)のようなサービスでは、SSL証明書を取得するのは簡単です。

## 本格的なPWA実装・提供に向けて

基本的なPWAの仕組みは上記の3点で整います。が、プッシュ通知を配信するといった豊富な機能を持つウェブアプリをPWAで実装・提供においては、上記の要素をプリミティブに用意し、PWAの諸機能を1つ1つ作成していくのでは非効率です。

このような場合には、下記のような技術スタックを利用することが一般的なようです。

![ionic2-firebase](/assets/images/2018-08-31-how-pwa-enforces-marketing-communication-2.jpg)

### [Ionic2](https://ionicframework.com/pwa)

Google社のAngularというアプリケーションフレームワークをベースに、HTML5アプリの開発に特化して作られたJavaScriptフレームワークです。
Web/iOS/Android アプリのいわゆるハイブリッド開発ができるフレームワークとして提供されていますが、PWAにも対応されています。

Ionic2でどんなウェブアプリの提供が可能なのかについては下記のIonic2のカンファレンスアプリを見るのが良いでしょう。

<iframe width="560" height="315" frameborder="0" allowfullscreen="" src="//www.youtube.com/embed/qCW2n17OHEk"></iframe><br><a href="https://youtube.com/watch?v=qCW2n17OHEk">Ionic v4 Conference App</a>

### [Firebase](https://firebase.google.com/?hl=ja)

Google社が提供するmBaaS（mobile backend as a Serviceの略で、モバイルアプリ開発のバックエンド側のインフラを提供するサービス）です。

下記の記事では、ユーザーを認証し、ユーザー情報を保存し、プッシュ配信機能を提供するウェブアプリの構成について紹介しています。

[How I built a PWA with Angular and Firebase Part 1: Introduction - CODE WITH STYLE](https://codewithstyle.info/how-i-built-a-progressive-web-app-with-angular-and-firebase-part-1/)

この中で紹介されている構成図は、ネイティブアプリのようなウェブアプリにおけるIonic2とFirebaseのわかりやすい関係図となっているかと思います。

![PWA Architecture](https://i1.wp.com/codewithstyle.info/wp-content/uploads/2017/04/Architecture.png?w=362)

## まとめ

以上、PWAを提供するための要素と技術スタックについて紹介してきました。

にわかに大きな注目を集めるようになった「PWA」ですが、エンゲージメントに活用しうるプラットフォームとして従来のウェブサイトやネイティブアプリをリプレイスしていくためには、「PWA」自体のさらなる普及と相まって、変化する技術トレンドにキャッチアップすることもハードルとなります。「その投資を自社かけられることができるか」は、マーケティングや商品開発の意思決定者にとって、今後、複雑でチャレンジングな検討課題になっていきそうです。

# 参考

- [^1]: [ウェブアプリ マニフェスト - Google Developers](https://developers.google.com/web/fundamentals/web-app-manifest/?hl=ja)
- [^2]: [Progressive Web Apps 101: the What, Why and How - freeCodeCamp](https://medium.freecodecamp.org/progressive-web-apps-101-the-what-why-and-how-4aa5e9065ac2)
- [^3]: [PWA 入門 〜iOS SafariでPWAを体験するまで〜 2018年6月版](https://qiita.com/umamichi/items/0e2b4b1c578e7335ba20)
