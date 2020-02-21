---
layout: post
title: 「PlanOut」に見るFacebookのABテストへの取り組み - その1 -
image: assets/images/2018-07-29-how-facebook-implements-ab-testing-in-planout.png
categories: [ A/Bテスト, Facebook, オープンソース ]
toc: true
comments: false
---
A/Bテストその他の「オンラインフィールド実験」は、マーケティングの検証スピードと精度を高めるものとして、広くインターネットマーケティングで使用されてきました。
PlanOutは、この「オンラインフィールド実験」のために、Facebookが2014年にオープンソース化したフレームワークです。

Facebookはサービス開始初期の「10日以内に7人の友だちを作ると、そのユーザーは毎日使用してくれるようになる」という発見など、継続的なグロース戦略でも有名です。
本記事では、Facebookの「オンラインフィールド実験」への取り組みを、PlanOutを通じて見ていきます。

## PlanOutとは

[PlanOut](https://github.com/facebook/planout)はFacebookが2014年にオープンソース化したA/Bテストその他の「オンラインフィールド実験」のためのフレームワークです。
FacebookはPlanOutを使って毎日数千の実験を億単位のユーザーを対象にして実施しているとされています。

- リリース
  - 2014年4月
- Githubスター数
  - 1,269(2018年7月時点)
- 開発言語
  - 基本実験環境：[Python](https://github.com/facebook/planout)
  - 実験用コード：Python, [Java](https://github.com/Glassdoor/planout4j), [JavaScript](https://github.com/HubSpot/PlanOut.js), [PHP](https://github.com/vimeo/ABLincoln)(アルファ版としてGo, Julia, Ruby)

多言語への移植としてJava、JavaScript、PHPがありますが、それぞれ有名企業によるオープンソースへのコントリビューションとなっています。
JavaScript版は[HubSpot社](https://www.hubspot.com/)、PHPは[Vimeo社](https://vimeo.com/)、またJava版をコントリビュートした[Glassdoor社は2018年5月にリクルートにより買収](https://jp.techcrunch.com/2018/05/09/recruit-glassdoor/)された企業です。

### A/Bテストとは

日本では、広義のA/Bテストはインターネットマーケティングにおける施策の良否を判断するために、2つの施策同士を比較検討する行為全般を指します。（Facebookでは「オンラインフィールド実験」と呼称）

いっぽう、狭義のA/Bテストはそのなかでも、単一の変数を単一の変数を変更した場合の仮説をテストするための手法を指します。典型的にはウェブサイト内の一部分を変更することで、比較したウェブサイト内のパーツ「A」と「B」のどちらがよりユーザビリティの観点から優れているかを実験します。

![Abtesting](https://upload.wikimedia.org/wikipedia/commons/2/2e/A-B_testing_example.png)

出典：[A/B testing - Wikipedia](https://en.wikipedia.org/wiki/A/B_testing)

### PlanOut使用のメリット

#### 単一要素での「A/Bテスト」はもちろん、多変量テストにも対応する

狭義の「A/Bテスト」が単一の変数を変更した場合の仮説をテストするための手法なのに対し、「多変量テスト」は、複数の変数を変更した場合の仮説をテストするための手法です。

![Big experiments](/assets/images/2018-07-29-how-facebook-implements-ab-testing-in-planout-big-experiments.png)

出典：[Big experiments: Big data’s friend for making decisions](https://www.facebook.com/notes/facebook-data-science/big-experiments-big-datas-friend-for-making-decisions/10152160441298859)

例えば上記の多変量テストでは、サイコロの色(白or赤)と背景の色(紺or赤)の4パターンの仮説を試験することができます。PlanOutではもちろん、このような多変量テストの実装方法も提供しています。

さらにPlanOutは複雑なテストにも対応します。並行して複数のテストを走らせる場合、いっぽうのテストにもういっぽうのテストが影響を受けてしまうようでは、テストの意義がなくなります。PlanOutではこのような場合にもそれぞれのテストが独立して実施されるよう、テスト対象グループのランダム化に注意が払われています。

#### A/Bテストのかゆいところに手が届く

A/Bテストを真剣に検討・実施するにあたり、下記のような調整が必要となることがあります。

- 対象グループをユーザーベースで（セッションベースで）固定してテストしたい
- 表示比率に傾斜を付けたい
- 有意差検定の基準を調整したい

PlanOutでは上記のような場合にも対応しています。

#### ロギングを自動で実施する

PlanOutではテストで分類したグループに基づいて、以後の行動情報のログ収集を自動で実施します。

### PlanOutを試してみる

#### インストール方法

インストールは下記のみと簡単です。

```shell
pip install planout
```

#### A/Bテストの実験パターンを実装してみる

Googleではリンクの青色を決めるにあたって、[マリッサ・メイヤーの提唱により41種類の青色とウェブサイト利用者の行動データを分析した](https://toyokeizai.net/articles/-/171160?page=2)といいます。この実験をPlanOutで実装すると下記のような形になります。 [^1]

- "ColorExperiment"クラスの宣言

```python
class ColorExperiment(SimpleExperiment):
  def assign(self, params, userid):
    params.blue_value = RandomInteger(min=215, max=255, unit=userid)
    params.button_color = '#0000%s' % format(params.blue_value, '02x')
    params.button_text = 'Join now!'
```

- アクセスしたそれぞれのユーザー(下記の場合はuserid=10)に対する"ColorExperiment"の実行

```python
color_exp = ColorExperiment(userid=10).get_params()
button_color = color_exp.get('button_color')
```

- HTMLテンプレートへの挿入(下記の場合はCSS部分)

```HTML
<style type="text/css">
a:link, a:visited {
    color: {{ button_color }}
}
</style>
```

## まとめ

以上の通り、PlanOutは、Facebookの何億ものユーザーを対象にした、何年にも渡る「オンラインフィールド実験」の試練をくぐり抜けてきただけあって、「オンラインフィールド実験」に必要十分な機能が充実しています。

どのような設計によりこれらの機能を実現しているか、後日さらに詳しく見ていきたいと思います。

※[後編](/entry/2018/07/29/how-facebook-implements-ab-testing-in-planout-2)を書きました。

## 参考

^1: [planout/0-getting-started.ipynb at master · facebook/planout](https://github.com/facebook/planout/blob/master/contrib/pydata14_tutorial/0-getting-started.ipynb)
