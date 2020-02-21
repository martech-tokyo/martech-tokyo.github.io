---
layout: post
title: オープンソースライブラリ「Ouibounce」の実装から「Exit Intent Technology」を垣間見る
image: assets/images/2018-05-06-ouibounce-to-exit-intent-technology.gif
categories: [ Web接客, オープンソース ]
toc: true
comments: false
---
LPO（ランディングページ最適化）を実施するうえでは、マウス操作などからユーザーの志向や心理を読み取り、離脱しそうなユーザーに対してのみコミュニケーションの追加実施することが有用です。
この領域はUSでは「Exit-Intent Market」などとして既に市場化が進んでいる領域でもあります。

この「Exit Intent Technology」[^1]の領域を知るにあたっては、同種のことをオープンソースで提供している人気ライブラリ「Ouibounce」について理解するのが早いでしょう。

[^1]: [What is exit intent technology? - Quora](https://www.quora.com/What-is-exit-intent-technology)

## タブを閉じる前にモーダルを出すライブラリ「Ouibounce」

タブを閉じる前にモーダルを出すライブラリ「[Ouibounce](https://github.com/carlsednaoui/ouibounce)」は、Github上でスター数が2,000件を超えるオープンソースの人気ライブラリです。

国内でも2014年に[Moongift](https://www.moongift.jp/2014/06/ouibounce-%E9%9D%A2%E7%99%BD%E3%81%84%EF%BC%81%E3%82%BF%E3%83%96%E3%82%92%E9%96%89%E3%81%98%E3%82%8B%E5%89%8D%E3%81%AB%E3%83%A2%E3%83%BC%E3%83%80%E3%83%AB%E3%82%92%E5%87%BA%E3%81%99%E3%83%A9/)が紹介したことで有名になりました。

内容としては、下記の通り、ウェブサイトをユーザーが離れようとしたときにモーダルを表示する、というものです。

![Ouibounce利用イメージ](https://camo.githubusercontent.com/6d050948b5ba97a0b925e0d0b101a5964b431ba6/687474703a2f2f636c2e6c792f696d6167652f32433270306c3357314d30302f6f7569626f756e63652e676966)

## 実現方法

この「Ouibounce」、中身は実はとてもシンプルな作りになっています。（そのためもあってかソース自体ここ数年更新されていないようです）

### ユーザーが離れようとする動きの検知

ユーザーが離れようとしたときのマウス操作の検知処理は下記の通りです。

- マウスのmouseleaveイベントに対するリスナーを作成
- リスナー上では、画面領域の上部から20ピクセル以上だった場合にのみ、(モーダル表示の)処理を実行。つまり画面の下や右、左側から画面領域外に出た場合は処理は実行されない

```javascript
var (省略)
    sensitivity  = setDefault(config.sensitivity, 20),
    _html        = document.documentElement;

...省略...

function handleMouseleave(e) {
  if (e.clientY > sensitivity) { return; }

  _delayTimer = setTimeout(fire, delay);
}

...省略...

_html.addEventListener('mouseleave', handleMouseleave);
```

[Mouseleaveイベント](https://developer.mozilla.org/en-US/docs/Web/Events/mouseleave)はマウスが要素の外に出た場合に一度のみ発火するイベントですが、ここではdocument.documentElement要素のMouseleaveイベントに対してリスナーを設定することで、ブラウザ画面領域の外に出たときにモーダル表示処理を実行する仕様となっています。

### マウスの離脱操作がほんの一瞬だったときのためのサスペンド処理

以上の通り、基本的にはとてもシンプルな作りなのですが、setTimeoutによるサスペンド処理にはOuibounceの処理の配慮を感じます。

```javascript
var (省略)
    timer        = setDefault(config.timer, 1000),
    delay        = setDefault(config.delay, 0),

...省略...

_html.addEventListener('mouseenter', handleMouseenter);

...省略...

function handleMouseenter() {
  if (_delayTimer) {
    clearTimeout(_delayTimer);
    _delayTimer = null;
  }
}
```

[Mouseenterイベント](https://developer.mozilla.org/en-US/docs/Web/Events/mouseenter)はマウスが要素内に外から入ってきた場合に一度のみ発火するイベントです。ここではdocument.documentElement要素に対してリスナーを設定することで、一度mouseleaveイベントが発生しても1秒(＝1000ミリ秒、デフォルト設定)以内に画面領域内にマウスが戻ってきた場合はイベント実行処理はサスペンドされ、モーダル表示は実施されません。

マウスのちょっとしたチラツキなどによってマウスポインタが一時的にブラウザ画面領域外に出てしまうことはままあることです。そうしたノイズに反応しないようにすることでユーザーにとって邪魔にならないようにしようとする配慮を感じます。

### オープンソースライブラリのある種の割り切り

以上より、マウスの離脱試行に対するモーダル表示処理を「Ouibounce」が非常にシンプルに実装できることがわかったかと思います。
そこにはユーザーの「離脱しようとする」心理を読み取るにおいて「マウスをタブのほうに移動させる」という操作のみをターゲットにしようとする、ある種の割り切りが感じられます。

## 商用ツールで提供される「Exit Intent Technology」

「Ouibounce」のような方法だと「ユーザーが本当はブックマークするつもりだったのにモーダルが表示されてしまった」などのノイズを取り除くことはできません。
これでもマーケティング現場での活用に耐えうるとするかは考え方次第ですが、理想的にはユーザーそれぞれのマウス操作傾向を考察し、それに応じたコミュニケーションを設計していくことが必要でしょう。
そのためには機械学習を利用して本当に離脱したユーザーの直前のマウスの動きを教師あり学習していくことが必要で、これはオープンソースのJavaScriptのみ提供のライブラリでは実現し得ない領域となってきます。

USだとこの領域は「Exit Intent Technology」の領域として市場化が進んでいます。有名どころだと下記あたりがあり、いずれも自社独自の「Exit Intent Technology」の力を謳っています。

- [BounceX](https://www.bouncex.com/)（旧Bounce Exchange）
- [Optimonk](https://www.optimonk.com/)
- [OptinMonster](https://optinmonster.com/)

ただ実際にツール選定などを行うにあたっては、まだまだ形成段階の市場であり、有用な比較検証の指標自体が存在しない印象です。
結局はどこまで効果的に（ノイズなく）離脱しそうなユーザーのみをターゲットとできるのか、その正確性・効率性がキモな訳で、いずれそういった比較検証の情報が出て来ることを期待したいところです。
