---
layout: post
title: ブロックチェーン技術はアドテクをディスラプトするか
image: assets/images/2018-09-17-blockchain-to-distupt-ad-technology-or-not.png
categories: [ ブロックチェーン, Brave, Web3.0, Substratum, Oyster, Protocol, Martechトレンド ]
toc: true
comments: false
---
Mediumを読んでいると、「Web 3.0」というキーワードがよく目につきます。[^1] 元はといえばブロックチェーンを利用した分散アプリケーションの開発・提供プラットフォームである「Ethereum」の共同創業者であるGavin Woodによって提唱された [^2] ようですが、ブロックチェーン技術の浸透、分散型アプリケーションの現実化によって「公平で透明なウェブ」「人間中心インターネット」といったコンセプトの「Web 3.0」への期待が日に日に高まりつつあるのを感じます。

マーケティングテクノロジーの領域でも、ブロックチェーン技術を利用したディスラプトに向けた取り組みが進んでいます。目下、そのメインのターゲットとなっているのがアドテク、もしくはその根拠となっているインターネットビジネスモデルの領域です。

![The State of Ad blocking](https://cdn.dribbble.com/users/1278296/screenshots/3351596/adblocker_-_dribbble.png)

出典：[The State of Ad blocking by Maxy Lotherington - Dribbble](https://dribbble.com/shots/3351596-The-State-of-Ad-blocking?utm_source=Clipboard_Shot&utm_campaign=maxy&utm_content=The%20State%20of%20Ad%20blocking&utm_medium=Social_Share)

## 現在のアドテクの課題

アドテクにおける上記のような取り組みの多くは下記の認識に基づいています。

- 現在使用されているオンライン広告モデルは正しく機能していない
- 例えば、多くのメディアサイトでバナー広告などの広告が掲載されているが、決してユーザーの興味関心に合ったものが掲載されているとは言いがたい。結果、ほとんどの広告はユーザーから無視される結果に終わる
- ユーザーの側は必要悪として広告を単に無視するか、広告を表示しないようにするアドブロッカーをインストールれてしまう。広告から最大の収益を得ているGoogleのChromeブラウザですら、そのための拡張機能を提供している
- 一方、メディア側としては、ユーザーに有料課金をオファーしても、むしろユーザーを離れてしまう結果に終わってしまう。そのため（ユーザーの多くに有害とわかっていても）サイトに広告を掲載するしかない
- 広告主側にとっても、アドテクによってその費用の大半は失われているのではないかとの疑念がある。（ガーディアンのハミッシュ・ニックリン氏が2016年、投じられた広告費の30％しか同社に入らないケースがあったと明かしている [^3] ）

これに対して起こしうる変化は、

- ユーザー／メディア／広告主それぞれのインセンティブ設計の変更
- ステークホルダーとそれによる「中抜き」の削減

となるでしょう。既にこれに取り組んでいるブロックチェーン企業をいくつか紹介します。

### アドテクから「中抜き」排除を狙う[Brave](https://brave.com/)

こういった企業のなかで最も有名と思われるのが[Brave](https://brave.com/)です。

![Brave](/assets/images/2018-09-17-blockchain-to-distupt-ad-technology-or-not-brave.png)

同社は、「Brave」という独自のブラウザを通じて、大企業が間にはいることなしに、メディア、広告主、そしてエンドユーザーの3者のトリプルウィンな関係を作ろうと取り組んでいます。

2015年、JavaScriptの開発者として知られるブレンダン・アイクが設立した同社は、2017年6月には「BAT（Basic Attention Token）」という仮想通貨によるICO（Initial Coin Offering）を実施し、わずか30秒の間に3500万ドルを調達しています。

このブラウザの最大の特徴は、デフォルトで広告をブロックすることです。ただ、「Brave」は決して広告をなくそうとしているわけではありません。なくそうとしているのはあくまで「中抜き」です。

そのビジネスモデルは下記の通りです。

- 広告主はBATトークンを使って「Brave」内に広告を出すことができる
- ユーザーにとっては、不要な広告を目にしなくて済むだけではなく、読み込みが早く、しかも（「控えめな」広告を見ることに同意した場合は）BATトークンが付与される
- またユーザーはアクセスするメディアサイトの運営者にBATトークンを直接支払うことができる。これはコンテンツに対する対価、応援という位置付け。そして運営者は中抜きされることなく広告収入を得ることができる

![Brave Business Model](/assets/images/2018-09-17-blockchain-to-distupt-ad-technology-or-not-brave2.png)

出典：[Basic Attention Token](https://basicattentiontoken.org/)

このビジネスモデルが成り立つためには、数千万人程度の「Brave」のユーザーがいなければならないと考えられています。 [^4] 一方、様々な調査機関の発表するブラウザシェアランキングにおいて、「Brave」はまだランクインする利用規模になってはおらず、本ビジネスモデルの実現可能性は未知数です。

ただ、「Brave」のビジネスモデルには、ユーザーとメディア、そして広告主がBATを媒介にして3者にメリットのあるつながりを作る可能性があるといえるでしょう。

### [Oyster Protocol](https://oysterprotocol.com/)

Oyster Protocolは、ウェブサイトを収益化させたいサイト運営者と簡単に安全なストレージを探しているユーザとを結びつけてお互いが求めているものを提供するプラットフォームです。

![Oyster Protocol](/assets/images/2018-09-17-blockchain-to-distupt-ad-technology-or-not-oyster-protocol.jpg)

Webサイトの運営者は、Webサイトの内部に1行のコードを設置するだけで収入が入るようになります。

Webサイトの訪問者は、自分のPCのCPUとGPUの電力の一部を提供する代わりに、自身のファイルを分散型ストレージに保存できます。

具体的には下記の通りです。 [^5]

- 「Oyster Protocol」を介してIOTA（IoTへの利用に特化して開発されている仮想通貨）のTangle（IOTAのコア技術で、有向非巡回グラフと呼ばれるデータ構造を応用した分散台帳技術）にファイルをアップロードする際、ユーザーはノードとしてプルーフ・オブ・ワーク（ブロックチェーンのコンセンサス・アルゴリズムの1つで、多大な計算量を要する問題を最初に解いたものに発言権を与えるというもの。またはそれに基づく計算作業）を実施
- 「Oyster Protocol」を実装したウェブサイト運営者やモバイル開発者はPearls（PRL）というOysterのトークンを稼ぐことができる

これにより、ウェブサイトの所有者は擬似的にストレージユーザーから報酬が支払いを受けていることになります。結果、ウェブサイトの訪問者はウェブサイトの所有者が広告に依存しにくくするのに貢献し、広告なしのブラウジング体験を楽しめます。

### [Substratum](https://substratum.net/)

「Oyster Protocol」と同様に、デフォルトのブラウザ（Safari、Firefox、Chrome、Internet Explorerなど）を活用してユーザーがコンピューティングリソースを提供し合うことで、インターネット上の規制を超え、情報発信と取得を可能にすることを狙っているのが「[Substratum](https://substratum.net/)」です。

![Substratum](/assets/images/2018-09-17-blockchain-to-distupt-ad-technology-or-not-substratum.png)

仕組みは下記の通りです。

- ユーザーにサブストラタムネットワーククライアントを実行するようにインセンティブを与えるために仮想通貨を提供する
- アプリケーションのように簡単に導入することが可能で、技術知識のないユーザーでも参加することができる
- Substratumネットワークは、「SUBSTRATUM HOST」（サイト運営・コンテンツ提供者）と
「SUBSTRATUM NODE」（コンテンツを運ぶ役割を担い、その仕事の報酬としてSUBトークンを受け取る）、「END WEB USER」（SUBSTRATUMネットワークを通じて検閲されていないコンテンツを通常のブラウザで閲覧できる）と3つの役割のノードが構成される

![Substratum](/assets/images/2018-09-17-blockchain-to-distupt-ad-technology-or-not-substratum2.png)

各PCやスマートフォンがノードとして公平な立場で、分散型のネットワークを構築しているため、規制にかかっている地域からでも、ネットワークにつながった規制にかかっていない地域のノードから、情報が入手ができるのです。

## まとめ

「Web 3.0」というお題目が示すとおり、ブロックチェーン技術が私たちに約束しようとする未来は、現時点では極端なものに聞こえます。
例えば、単なる好奇心や使命感などを脇においたときに、「Substratum」のビジネスモデルを知ったときに1エンドユーザーとしてそれに参加しようというリスク・リターンの感覚を持つ人はどれくらいいるでしょうか。

しかし、その感覚自体、数十年後の「常識」からすると的外れなものになるかもしれません。
例えば、1990年代に「インターネット」上で発言などの何らかのコミットをすることに対して一般エンドユーザーが抱いてに違いない「ハイリスク」という感覚と同様のものを、現在ブロックチェーンに感じていたとします。とすると、30年後に全くその感覚自体が変わっている可能性を誰が否定できるでしょうか。（その際には「ネットリテラシー」ならぬ「仮想通貨リテラシー」なる言葉が存在するようになっているのかもしれません）

少なくとも、デジタルにおけるこれまでの他の革命と無関係でいられたマーケッターがいなかったように、ブロックチェーン技術は、そう遠くないうちに、他の産業と同じくマーケティングの領域に、大きな変化を引きおこすことは確かでしょう。今後もその進化を見守っていきたいと思います。

## 参考

[^1]: [The Web 3.0: The Web Transition Is Coming – Hacker Noon](https://hackernoon.com/the-web-3-0-the-web-transition-is-coming-892108fd0d)
[^2]: [Why We Need Web 3.0 – The Crypto Collection – Medium](https://medium.com/s/the-crypto-collection/why-we-need-web-3-0-5da4f2bf95ab)
[^3]: [ブロックチェーン技術が、広告から「中間業者」を排除する | Agenda note (アジェンダノート)](https://agenda-note.com/technology/detail/id=84&pno=0)
[^4]: <a href="https://ja.wikipedia.org/wiki/Brave_(%E3%82%A6%E3%82%A7%E3%83%96%E3%83%96%E3%83%A9%E3%82%A6%E3%82%B6)">Brave - Wikipedia</a>
[^5]: [Exclusive interview to the Oyster protocol team before Mainnet release - IOTA HISPANO](http://iotahispano.com/2018/05/28/exclusive-interview-to-the-oyster-protocol-team-by-iota-hispano-before-mainnet-release/#comment-12)
