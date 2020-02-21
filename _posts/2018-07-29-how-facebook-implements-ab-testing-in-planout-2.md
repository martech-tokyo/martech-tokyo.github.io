---
layout: post
title: 「PlanOut」に見るFacebookのABテストへの取り組み - その2 -
image: assets/images/2018-07-29-how-facebook-implements-ab-testing-in-planout-2.png
categories: [ A/Bテスト, Facebook, オープンソース ]
toc: true
comments: false
---
前回記事で触れた、A/Bテストその他の「オンラインフィールド実験」のためのオープンソース「[PlanOut](https://github.com/facebook/planout)」。

[先日の記事](/entry/2018/07/22/how-facebook-implements-ab-testing-in-planout.html)ではA/Bテストの仕組みとPlanOutの特長、インストール方法から使い方までを概観しました。

今回は「PlanOut」のキーコンセプトである「ネームスペース」に着目して、その設計ポイントを紹介していきます。

「ネームスペース」とは、
Googleの 「レイヤー」[^1] 、Microsoftの「ラインナンバー」 [^2] 、Facebookの「ユニバース」に似た概念で、オンラインフィールド実験同士の重複による問題を解決するためののものです。

## 「ネームスペース」のモチベーション [^3]

- 反復的な検証への対応

「オンラインフィールド実験」は1回の実験では決定的ではないことが多く、同じパラメータを操作する実験によるフォローアップが必要です。2回目の実験では、成果をより正確に推定するためにほぼ同一の設計の実験を再度行ったり、または新しいバリエーションを試したりします。
このような継続的な再設計と開発により、パラメータの影響が変化する可能性があります。

- 同じ部分を同時に検証する並行実験への対応

同じサービスの同じ部分に複数の実験が並行して実行されることがあります。（たとえば、同じ商品一覧ページの、フォントサイズとページあたりのアイテム数など）
これらを矛盾なく実行するためには、各実験によってどのパラメータが設定されているかを追跡・制限することができるシステムを持つのが有用です。

このためには、実験ツールをクロスプラットフォームなものにし、異なるチームによって異なる時間に開始された複数の実験に対してそれぞれ実験対象数の割り当てができるようにする必要があります。

## 「ネームスペース」の仕組み

「ネームスペース」の仕組みにより、オンラインフィールド実験の実施時には、ユーザーのアクセスごとに「フォントサイズは12pxか14pxか」「ページあたりのアイテム数は10か20か」などのパラメータを、直接要求するのではなく、ネームスペースから要求するようになります。
このパラメータは、そのプライマリユニット（例：ユーザーID）が属する実験がある場合、そのユニットが含まれている実験についてどの検証プランを表示すべきかを処理処理します。
開発者の視点からは、実験システムから直接パラメータを要求していた時と同じように、「ネームスペース」にパラメータをリクエストし、記録することができます。

「ネームスペース」の仕組みの裏側では、プライマリユニットは多数のセグメント（例えば、10,000）のうちの1つにマッピングされます。
新しいテストが作成されると、セグメントはランダムに割り当てられます。
セグメントが実験に割り当てられている場合、入力データは実験システムに渡され、対応する「Experiment」オブジェクトのロジック通りにランダム割り当てが行われます。

![namespace_diagram](https://facebook.github.io/planout/static/namespace_diagram.png)

出典：[PlanOut - Namespaces](https://facebook.github.io/planout/docs/namespaces.html)

プライマリユニットが実験に割り当てられていない場合や、実験で定義されていないパラメータが要求された場合は、デフォルトの実験値を使用できます。これにより、実験者は、現在実行中の実験に干渉しないように、即座にパラメータのデフォルト値を設定することができます。

## 「ネームスペース」を使ってA/Bテストを行う

例えば下記のような実装により、「ネームスペース」を使ってA/Bテストを行うことができます。

```python
from planout.namespace import SimpleNamespace
from planout.experiment import SimpleExperiment, DefaultExperiment
from planout.ops.random import *


class V1(SimpleExperiment):
  def assign(self, params, userid):
    params.banner_text = UniformChoice(
      choices=['Hello there!', 'Welcome!'],
      unit=userid)

class V2(SimpleExperiment):
  def assign(self, params, userid):
    params.banner_text = WeightedChoice(
      choices=['Hello there!', 'Welcome!'],
      weights=[0.8, 0.2],
      unit=userid)

class V3(SimpleExperiment):
  def assign(self, params, userid):
    params.banner_text = WeightedChoice(
      choices=['Nice to see you!', 'Welcome back!'],
      weights=[0.8, 0.2],
      unit=userid)


class DefaultButtonExperiment(DefaultExperiment):
  def get_default_params(self):
    return {'banner_text': 'Generic greetings!'}

class ButtonNamespace(SimpleNamespace):
  def setup(self):
    self.name = 'my_demo'
    self.primary_unit = 'userid'
    self.num_segments = 100
    self.default_experiment_class = DefaultButtonExperiment

  def setup_experiments(self):
    self.add_experiment('first version phase 1', V1, 10)
    self.add_experiment('first version phase 2', V1, 30)
    self.add_experiment('second version phase 1', V2, 40)
    self.remove_experiment('second version phase 1')
    self.add_experiment('third version phase 1', V3, 30)

if __name__ == '__main__':
  for i in xrange(100):
    e = ButtonNamespace(userid=i)
    print 'user %s: %s' % (i, e.get('banner_text'))
```

※[Github内のdemoより引用](https://github.com/facebook/planout/blob/master/demos/demo_namespaces.py)

実験の経過は下記のような流れとなっています。

1. V1のテストを最初10%のセグメントにのみ開始
2. V1のテストを30%のセグメントに拡大
3. V2のテストを40%のセグメントに実施
4. V2のテストを中止
5. V3のテストを30%のセグメントに対して開始

## まとめ

以上の通り、「PlanOut」は、個々のユーザーに対してランダムに検証プランを割り振る仕組みの管理層として「ネームスペース」を用意しています。

この層が存在することにより、実験を実施する側は、実験プランをコードに書き加えることで機動的に実験を拡大したり、新たな実験に変更したりすることができるわけです。

「PlanOut」はA/Bテストの下記のようなユースケースを踏まえた現実的なフレームワークといえます。まさにFacebookの毎日数千の実験のなかから生まれてきた所以でしょう。

- A/Bテストをまずは少量のユーザーから開始し、反応がよければ徐々に増やしていきたい
- 複数のA/Bテストを同時に走らせたい。どっちもやっていない層との比較もしたい

開発者と実験者が同じチームの場合には、「PlanOut」はオンラインフィールド実験のツールとして魅力的な選択肢になってくるのではないでしょうか。

一方、コードを書けるメンバーが検証チーム内にいない場合にはなかなか運用が難しいツールであることも確かです。
また都度実験プランをコードに書いて反映しなければならないのを負担に感じる場合もあるでしょう。
そういった場合には「Optimizely」などのA/Bテストツールが選択肢に入ってくるのだと思います。

## 参考

[^1]: [Online Controlled Experiments at Large Scale](https://www.exp-platform.com/Documents/2013%20controlledExperimentsAtScale.pdf)
[^2]: [Overlapping Experiment Infrastructure:More, Better, Faster Experimentation](https://storage.googleapis.com/pub-tools-public-publication-data/pdf/36500.pdf)
[^3]: [Designing and Deploying Online Field Experiments](https://arxiv.org/pdf/1409.3174v1.pdf)
