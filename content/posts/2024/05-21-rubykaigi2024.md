---
title: "RubyKaigi 2024に参加してきた #rubykaigi"
date: 2024-05-21T10:00:00+09:00
url: /2024/05/21/rubykaigi2024
images: ["/images/2024/0521/rubykaigi2024.png"]
---

{{< figure src="/images/2024/0521/rubykaigi2024.png" >}}

ハイサイ！

2024/05/15 - 17、沖縄県那覇市で開催された[RubyKaigi 2024](https://rubykaigi.org/2024/)に参加していました。この記事は那覇空港のビジネスラウンジで書いています（書き終わったら[Helios](https://www.naha-airport.co.jp/news/11734/)だ！）

Day 0はあいにくの雨、Day 7は梅雨入りで雨でしたが、それ以外は好天に恵まれ、Kaigiも沖縄も満喫することができました。

## 印象に残ったセッション

### Writing Weird Code / tomoya ishida

- [Description](https://rubykaigi.org/2024/presentations/tompng.html#day1)
- [Slide](https://drive.google.com/file/d/1Dkx15u_5UAGoFqJHCeAuj2FXS-z_U7EE/view?usp=sharing)

RubyKaigiで開催されるたびに衝撃と爆笑が巻き起こる[TRICK](https://github.com/tric)の2022 チャンピオンがひとりTRICKをたっぷり6本見せてくれる、という最高のキーノートによって、RubyKaigiは面白いよ、Tech Conferenceだよという強烈なメッセージで開幕しました。

[実際のコード](https://github.com/tompng/selftrick2024)を見ても、なるほどわからん…。それぞれのリポジトリにはGit Logが添付してあるので、それを見ながら読み込んでみようと思います。

[Most Floral](https://github.com/tompng/selftrick2024/tree/main/floral)だけは昔取った杵柄でちょっとだけわかりみがありました。なんでバイナリにコードを埋め込むと左下の画素が影響されるかというと、[BMPフォーマット](https://www.setsuki.com/hsp/ext/bmp.htm)の画素データは左下から右上に、つまり上下反転して記録されているのです。コードを読むときはヘッダの構造も参照しながらがよいでしょう。

<!--more-->

### Cross-platform mruby on Sega Dreamcast and Nintendo Wii / Yuji Yokoo

- [Desccription](https://rubykaigi.org/2024/presentations/yujiyokoo.html#day1)
- [Slide](https://github.com/yujiyokoo/dreampresent-wii/releases/tag/RubyKaigi-2024)

家庭用ゲーム機でmrubyを動かすシリーズ第3弾、[Dreamcast](https://www.sega.jp/history/hard/dreamcast/)用プレゼンテーションソフト「[Dreampresent](https://github.com/yujiyokoo/dreampresent)」を[Wii](https://www.nintendo.co.jp/wii/index.html)で動かす、という話です。

[RubyKaigi 2022](https://rubykaigi.org/2022/)の[Megarubyの話](https://rubykaigi.org/2022/presentations/yujiyokoo.html#day3)を聴いて以来Yuji Yokooさんの一方的なファンになってしまい、今回も楽しみにしていました。

だいたいDreamcastとWiiは世代もアーキテクチャも周辺デバイスもなにもかも違うじゃん！とか思ってたのですが（Wiiは[コントローラー](https://www.nintendo.co.jp/wii/controllers/index.html)が特徴的）、それがまさかオチになってるとは…。

プレゼンの元データは[このへん](https://github.com/yujiyokoo/dreampresent-wii/tree/main/romdisk)で見ることができます。画面に表示されてたテキストは[content.txt](https://github.com/yujiyokoo/dreampresent-wii/blob/main/romdisk/content.txt)に記述されてます。このエスケープシーケンスみたいな独自の記法が含まれるテキストデータは、ゲーム開発の頃を思い出して感慨深い気持ちになりました（キャラやテキストを制御するシナリオスクリプトというものがあってな）。

Day4にひとりでビールを飲んでたらその店に[その店にYokooさんが入ってくる](https://x.com/kwappa/status/1791807863838683532)というミラクルがあるのもRubyKaigiのすごいところ。一方的なファンだったのにすっかり仲よくしていただき、そのあとも何度か一緒に飲みに行く機会がありました。Yokooさん、ありがとうございました！

### Namespace, What and Why / Satoshi Tagomori

- [Description](https://rubykaigi.org/2024/presentations/tagomoris.html#day1)
- [Slide](https://speakerdeck.com/tagomoris/namespace-what-and-why)

RubyにNamespaceの機能を導入しよう、という試みの現状報告でした。ライブデモ、動いてるすごーい！

個人的には昔[Padrino](https://padrinorb.com/)で[開発をしていた](https://speakerdeck.com/kwappa/padrino-in-production)ので、Namespeceの具体的なご利益はすぐ思い当たりました。Padrinoには[Sub Application](https://padrinorb.com/guides/generators/sub-applications/)という仕組みがあり、複数のアプリケーションをパスでルーティングしながら動かすことができるのです。大変便利ですが、マウントしたアプリケーション同士で名前衝突が起こるため、ご利用は計画的に、という機能です。

このセッションはMatzのクロージングキーノートの伏線になっていた、というのがDay 3にわかったのもエモくて最高でした。また、夜のPartyやそのあとの飲み会でスピーカーのtagmorisさんが「Namespaceどうだった？」と感想やユースケースの聞き込みをしているのも、RubyKaigiのオンサイト開催ならではの素敵な光景でした。

### It's about time to pack Ruby and Ruby scripts in one binary / ahogappa

- [Description](https://rubykaigi.org/2024/presentations/ahogappa0613.html#day2)
- [Slide](https://speakerdeck.com/ahogappa0613/its-about-time-to-pack-ruby-and-ruby-scripts-in-one-binary)

Rubyのプログラムと関連ミドルウェアをワンバイナリにして配布したいという欲求を、自分でバリバリ実装して実現した、というかっこいい内容です。[Rustで仮想ファイルシステムを書いてる](https://github.com/ahogappa0613/kompo-vfs/tree/main/kompo-cli)？すごくない？（Rustなんもわからんがあとでもう少し読む）

そういえば昔、なにかの自動化ツールが[Ruby2exe](https://github.com/matti/ruby2exe)によって提供されてて、便利だけど壊れると作者のメンテ待ちになった記憶が蘇ってきました。今ならわかるんだけど、[ないものは作ればいい](https://speakerdeck.com/ahogappa0613/its-about-time-to-pack-ruby-and-ruby-scripts-in-one-binary?slide=19)んですよね。それを思い出させてくれるセッションでした。

スピーカーのahogappaさんは以前の同僚です。新卒入社のタイミングを知ってるエンジニアがRubyKaigiのスピーカーになる、という点でも感慨深いセッションでした。

### Porting mruby/c for the SNES (Super Famicom) / Ryota Egusa

- [Description](https://rubykaigi.org/2024/presentations/gedorinku.html#day3)
- [Slide](https://speakerdeck.com/gedorinku/c-for-the-snes-super-famicom-rubykaigi-2024)

mruby/cを[SNES](https://ja.wikipedia.org/wiki/Super_Nintendo_Entertainment_System)（日本では[スーパーファミコン](https://www.nintendo.co.jp/n02/shvc/index.html)）で動かす、という話。アーキテクチャから調べて実装していてすごい…。拡張カートリッジとか、存在は知ってたけど仕組みは全然だったので、めっちゃ勉強になりました。

ちょっとちらつきがありながらも実機で動くデモが成功したのもポイントです。スマートフォンアプリを開発したことがある方はちょっとわかるかもなのですが、エミュレータではなく実機で動くことには、エンジニアとしての強い快楽があります。

さきほどのYokooさんの[Megarubyの話](https://rubykaigi.org/2022/presentations/yujiyokoo.html#day3)にインスパイアされてやってみた、というのもエモがありますね。ホールでは偶然Yokooさんのすぐ近くに座っていたのですが、デモが動いた瞬間誰よりも早く大きく拍手をされていたのが印象的でした。

## See you in next RubyKaigi in Matz-Yama！ :mountain:

来年は愛媛県松山市での開催です。四国は去年初めて行ったんだけど、徳島と香川をちょっと回っただけで愛媛は未体験。楽しみです。

{{< figure src="/images/2024/0521/see_you_in_matsuyama.jpg" width="100%" >}}

