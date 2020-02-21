---
layout: post
title: Google Analyticsのコマンドキューの実装を覗いてみる
image: assets/images/2018-05-31-google-analytics-command-queue-implementation.png
categories: [ アクセス解析 ]
toc: true
comments: false
---
みんな大好き「Google Analytics」では、非同期によるJavaScriptの読み込み・実行によりアクセスログを逃さずサーバに送信し解析結果を正確に保つため、以前より「コマンドキュー」という実装方式が取られ、これは現行バージョンの「gtag.js」にも継承されています。ここでは「Google Analytics」のJavaScriptのコマンドキューに関する内部実装を見ていきます。

## コマンドキューとは

コマンドキューとは、コマンドを待ち行列（キュー）に入れ、ターゲットの意図する順番で処理する形式のことです。

「Google Analytics」のJavaScriptスニペットでは、本体の「gtag.js」ライブラリが読み込まれるまでは関数の実行をすることができませんが、ライブラリが完全に読み込まれる前でもトラッキング対象となるイベント（スクロール、クリックなど）が発生する可能性があるため、ライブラリが完全に読み込まれる前でもコマンドを使用できる状態にしておく必要があります。

そのため、gtag.jsライブラリが読み込まれる前は gtag() コマンドはコマンドキュー関数として定義され、gtag.js ライブラリが完全に読み込まれる前は実行したいコマンドをキューとして追加・保持します。
コマンドキューのコマンドは、gtag.jsライブラリが読み込まれるとすぐに、追加された順に実行され、これが完了すると、キューに新しくプッシュされたコマンドがすぐに実行されます。

それでは実際の実装を見ていきましょう。

## コマンドキューの実装

### 1. 配列への登録

「gtag.js」のスニペットは下記のような表記となっています。

```javascript
<!-- Global Site Tag (gtag.js) - Google Analytics -->
<script async src="https://www.googletagmanager.com/gtag/js?id=GA_TRACKING_ID"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());

  gtag('config', 'GA_TRACKING_ID');
</script>
```

1行目のスクリプト読み込みにより、「gtag.js」をgoogletagmanager.comから読み込み開始します。
しかしこのスクリプト読み込みのタグには `async` 属性が付与されているため、読み込みと平行して次以降のスクリプトも実行が始まります。

これ以降の `gtag` 関数では、`window` 配下のグローバルな配列オブジェクト `dataLayer` に、後に「Google Analytics」初期化関数呼び出し時の引数となる情報を、順に追加していきます。
1つめが `js`、2つめが`config` です。

通常は1行目のスクリプト読み込みよりも先に後のscriptの実行が完了するため、ブラウザは `dataLayer` が2行の配列となった状態で gtag.js ライブラリの読み込みを待ちます。

![dataLayer](/assets/images/2018-05-31-google-analytics-command-queue-implementation.png)

### 2. ライブラリが読み込まれるとすぐに、追加された順に実行

次にライブラリが読み込まれたあとを見ていきましょう。

まず、下記スクリプトにて初期化が実施されます。

```javascript
ug.ld();
```

この関数のなかで呼び出される `be` 関数は下記の通りです。

```javascript
be = function() {
    var a = Fa("dataLayer", []),
        b = Fa("google_tag_manager", {});
    b = b["dataLayer"] = b["dataLayer"] || {};
    Rc.push(function() {
        b.gtmDom || (b.gtmDom = !0, a.push({ // google_tag_manager.gtmLoad == true だったらそのまま終了、falseだったらtrueにすると同時に dataLayerに `event: "gtm.load"`` を追加
            event: "gtm.dom"
        }))
    });
    Vd.push(function() {
        b.gtmLoad || (b.gtmLoad = !0, a.push({
            event: "gtm.load"
        }))
    });
    var c = a.push;
    a.push = function() {
        var b = [].slice.call(arguments, 0);
        c.apply(a, b);
        for (Xd.push.apply(Xd, b); 300 < this.length;) this.shift();
        return $d()
    };
    Xd.push.apply(Xd, a.slice(0));
    P(ae)
};
```

上記のなかで `dataLayer` 配列の中身を随時削除するとともに、ここから呼び出される `$d` のなかで下記の通り `js`、`config`といった関数を実行していきます。

```javascript
if (l.length && kc(l[0])) {
    var m = Md[l[0]];
    if (m) {
        b = m(l);
        break a
    }
}
```

`Md` オブジェクトの中に `js`、`config`といった関数がオブジェクトとして格納されています。

```javascript
var Md = {
        event: function(a) {
          (内容省略)
        },
        set: function(a) {
          (内容省略)
        },
        js: function(a) {
          (内容省略)
        },
        config: function(a) {
          (内容省略)
        }
    },
```

### 3. 初期化時の処理書き換え

そのなかで下記により、DOM構築が終わっていたら即座に、終わっていなかったらwindowのload後に `Wd` 関数が実行されます。

```javascript
"complete" === document.readyState ? Wd() : Ia(window, "load", Wd);
```

`Wd` 関数の中身は下記の通り。

```javascript
var Ud = !1,
    Vd = [];

function Wd() {
    if (!Ud) {
        Ud = !0;
        for (var a = 0; a < Vd.length; a++) P(Vd[a])
    }
};
```

ここで `P` という関数は0秒のみ待つsetTimeout関数なので、

```javascript
P = function(a) {
    window.setTimeout(a, 0)
},
```

0秒後に `Vd[a]` を実行することとなります。

そのなかで、下記にて `gtag` 関数を無効化しているようです。

```javascript
var Bc = function(a) {
        if (a) {
            var b = Cc(["gtag", "targets", a]);
            return qa(b) ? b : void 0
        }
    },
```

## まとめ

以上の通り実装を覗いてみました。

コマンドキュー形式は、クライアントブラウザやそのネットワーク環境といった不確定要因に対してリスクヘッジを行う、優れた実装方法と言えそうです。
