---
layout: post
title: 「FingerPrint2」に見るフィンガープリンティングの最新事情 - その2 -
image: assets/images/2018-05-19-browser-finger-print-now-from-fingerprint2.png
categories: [ オープンソース, アクセス解析 ]
toc: true
comments: false
---
以前も触れた「[Fingerprint2](https://github.com/valve/fingerprintjs2)」によるブラウザフィンガープリンティング。

[先日の記事](/entry/2018/05/19/browser-finger-print-now-from-fingerprint2)ではブラウザフィンガープリンティングの仕組みとその実現に利用している情報群を概観し、各ブラウザでの検証結果について紹介しましたが、今回は実装の詳細を見てみます。

## 「FingerPrint2」で使われている技術群

「FingerPrint2」でハッシュ値作成に使われている技術群のなかで興味深かったものを紹介していきます。

### 1. インストールされているフォントリスト

![Fonts](/assets/images/2018-07-07-browser-finger-print-now-from-fingerprint2-fonts.png)

「JS/CSSによりインストールされているフォントを検知する」という技術、個人的に興味深かったのですがこれ自体は枯れた技術のようで、少なくとも2008年頃には開発やそれにまつわる議論が存在していたようです。
なかでも有名なのは下記のJavaScriptライブラリです。（残念ながら本家サイトは閉じてしまったよう）

<script src="https://gist.github.com/szepeviktor/d28dfcfc889fe61763f3.js"></script>

※[引用元](https://gist.github.com/szepeviktor/d28dfcfc889fe61763f3)

大まかな処理の流れは下記の通りです。

1. デフォルトのフォントを用いてテスト文字列"mmmmmmmmmmlli"の幅と高さを取得（このコードではデフォルトのフォントとして、'monospace', 'sans-serif', 'serif'の3つを利用）
2. 次に、指定したフォントを用いてテスト文字列"mmmmmmmmmmlli"の幅・高さを取得し、1のデフォルトフォントの幅・高さと比較
3. 取得した文字列の幅と高さが、1のデフォルトフォントでの幅と高さのどれにも一致しなかった場合、指定したフォントはデフォルトのフォントではないため、フォントはインストールされていると判定
4. デフォルトフォントのいずれかの幅と高さに一致した場合は、フォントが存在しなかった＝デフォルトのフォントが代用されたとみなし、フォントはインストールされていないと判定

「FingerPrint2」のソースコード内でも「kudos to http://www.lalit.org/lab/javascript-css-font-detect/」というコメントの通り、上記ライブラリを参照して実装されています。（デフォルトのフォント、テスト文字列まで一緒）

<script src="http://gist-it.appspot.com/github/Valve/fingerprintjs2/blob/3c3920f5183ec96b914cda7d06c8ab28ace59d23/fingerprint2.js?slice=367:380" ></script>

### 2. Canvasフィンガープリント

次はCanvasというJavaScriptを使って動的に図を描くために策定された仕様を使用したフィンガープリンティングです。
CanvasはGIFやJPEG、Flashなどを利用せずに図を表示するものですが、実際にその図が表示される寸法や位置、色は、OSの環境によって細かく異なります。
それがいかにブラウザやOSの環境によって異なるのかについては、35のブラウザ環境におけるCanvas画像の表示の違いを示した下記アニメーションがわかりやすいのではと思います。

![アニメーション画像](https://browserleaks.com/img/canvas-fingerprinting.gif.png)

出典：[https://browserleaks.com/canvas#how-does-it-work](https://browserleaks.com/canvas#how-does-it-work)

これを利用しているのがCanvasフィンガープリントです。

<script src="http://gist-it.appspot.com/github/Valve/fingerprintjs2/blob/3c3920f5183ec96b914cda7d06c8ab28ace59d23/fingerprint2.js?slice=741:801" ></script>

ここでは下記のような流れでハッシュ値を作成しています。

1. Canvas上で様々なパターンの図を作成。文字のほかに、長方形や円弧、その塗りつぶしなど。
2. ``toDataURL()``関数を利用して1の図をそれらを画像バイナリファイルのbase64エンコード文字列に変換
3. ここではピクセル単位の違いが文字列の差となって表れるため、その環境特有のデータとなる

## 「FingerPrint2」の利用推奨オプション

このような興味深い技術から構成される「FingerPrint2」ですが、実用にはいくつか制約があります。

### OSやブラウザのバージョン自動更新により容易にUserAgentが変更になる

最近のOSやブラウザは起動時にバックグラウンドでバージョンを自動で更新します。
これにより、UserAgentのバージョン部分が変更になることによってフィンガープリントも変わってしまいます。

- "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/**66.0.3359.181** Safari/537.36"
- "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.13; rv:**61.0**) Gecko/20100101 Firefox/**61.0**"

これを考えると、「FingerPrint2」で取得したハッシュ値を一定期間に渡りユーザーを識別する値として使いたい場合には、UserAgent値をフィンガープリントに含まないよう ``excludeUserAgent: true`` とするのが望ましいでしょう。

ちなみにバージョンを削除してUserAgentのOS名とブラウザ名だけでハッシュ値を作成するカスタマイズ方法もGithub上で紹介されています。

```
new Fingerprint2({
  preprocessor: function(key, value) {
    if (key == "user_agent") {
      var parser = new UAParser(value); // https://github.com/faisalman/ua-parser-js
      var userAgentMinusVersion = parser.getOS().name + ' ' + parser.getBrowser().name
      return userAgentMinusVersion
    } else {
      return value
    }
  }
}).get(function(result, components) {
  // user_agent component will contain string processed with our function. For example: Windows Chrome
  console.log(result, components)
});
```

### SSL環境外ではデバイスのメモリサイズが正常に取得できない

これについては下記の[Github issue](https://github.com/Valve/fingerprintjs2/issues/328)に詳しいですが、「Device-Memory Client Hint header and JS API」にはSSLページでしか利用できないという制約があるため、ユーザーがSSLページなのか非SSLページなのかでフィンガープリント値も変わってしまいます。（また、非SSLページでは必ず``-1``が戻り値として返ってくるだけなので、フィンガープリント値としては意味のないものとなってしまいます）

非SSLページが一部混ざっているようなサイトの場合、これを避けるために ``excludeDeviceMemory: true`` とするのが良いでしょう。
（そして、これ一つとっても「FingerPrint2」を実用に乗せるための開発者達の苦労が伺えます…。）

## まとめ：「ポストGDPR」時代のオープンソースによるユーザートラッキング

一つ一つの取得情報に、技術的にすごく難しいわけではない、だけどなかなか普通には思いつかないようなアイデアが詰まっているブラウザフィンガープリンティング。
このようなユーザートラッキング手法が

- GDPRの施行
- アドブロックの使用普及
- ITPなどに見られるクッキーや端末ID利用の制限強化

といった流れのなかで、オープンソースでたくさんの協力者間で知恵を出し合いながら開発・利用が進んでいるのは大変興味深い事象です。
いつかウェブ上で個人を識別する方法として、CookieやLocalStorageなどに変わる、より便利でセキュアな手法となる日が来ることを期待しながら、引き続き注目していければと思います。
