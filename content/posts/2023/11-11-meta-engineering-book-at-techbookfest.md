---
title: "「メタエンジニアリング 理論と実践、その未来」という同人誌を技術書典15で頒布します"
date: 2023-11-11T06:00:00+09:00
url: /2023/11/11/meta-engineering-book-at-techbookfest
---
<a href="https://techbookfest.org/product/1Vu6ceKBjEWu6Bv4WcBhYs">{{< figure src="/images/publishing/meta-engineering.png" width="480" >}}</a>

[「メタエンジニアリング」という概念](/2022/12/22/meta-engineering/)についての同人誌を執筆し、[技術書典15](https://techbookfest.org/event/tbf15)で頒布します。オフライン会場にも「あ23 河童書房」として出展します。

書籍販売ページはこちらです。

- [メタエンジニアリング 理論と実践、そして未来：河童書房](https://techbookfest.org/product/1Vu6ceKBjEWu6Bv4WcBhYs)

<!--more-->

## いきさつ

昨年は「メタエンジニアリング」という概念について書いたり喋ったりしてきました。その内容は「[メタエンジニアリングについて考えた2022年](/2022/12/22/meta-engineering/)」という記事にまとめてあります。

「メタエンジニアリング」とは、ソフトウェア開発に携わるエンジニアとエンジニアリング組織の生産性を、エンジニアリングの技術・知識・経験を使って向上する取り組みのことです。技術広報・採用・組織開発の領域で、課題の発見・解決や組織と個人の成長支援を行います。

いろいろアウトプットの機会はいただいたのですが、全容が把握できるようなコンテンツは用意していなかったなそういえば、ということに気がついたので、今回は同人誌としてまとめてみました。[個人的な事情](https://kwappa.net/blog/20231102/status_update/)により準備期間が圧縮されてしまい、本としての体裁は最低限なんとか、物理本もコピー誌を少部数しか用意できていない、みたいな状態ですが、幸い[技術書典15](https://techbookfest.org/event/tbf15)の開幕には間に合いました。

## 執筆工程

理想的なものとは程遠いですが、急に思い立っても電子書籍（ePub）オンリーだったらこのぐらいでサクッと書ける、という意味で事例を紹介します。

### ツール

以下のツールを使っています。

#### [Visual Studio Code](https://azure.microsoft.com/ja-jp/products/visual-studio-code)

Markdownによる原稿執筆と、エクステンションによるePub生成に利用しました。

#### [Markdown Preview Enhanced](https://marketplace.visualstudio.com/items?itemName=shd101wyy.markdown-preview-enhanced)

その名の通り、Markdownのプレビューを強化したエクステンションです。プレビュー表示して右クリックでeBookが出力できるのはよい体験でした。

ドキュメントは[GitHub Pages](https://shd101wyy.github.io/markdown-preview-enhanced/#/)にあります。

使い方はこちらの記事をめっちゃ参考にしました。

- [VSCode + Markdownでお手軽電子出版 #Python - Qiita](https://qiita.com/ysugimura_it/items/21d1d39b643a0763e6d9)

#### [calibre](https://calibre-ebook.com/download)

電子書籍のリーダー / ジェネレーターです。MPE（Markdown Preview Enhanced）がeBookを出力する時に使います。

インストールしたらMPEが利用する `ebook-convert` にパスを通す必要があります。パスの通った場所にsymlinkを作るコマンドが公式にも紹介されているのですが、macOSのバージョンによっていくつか罠があります。

macOS 14.0 Sonomaにおいては、以下のように行いました。

{{< highlight bash >}}

sudo ln -s /Applications/calibre.app/Contents/MacOS/ebook-convert /usr/local/bin

{{< / highlight >}}

- calibreのパスが `~/Applications/` になってるサンプルがある（公式とか）  
    …現状では普通にインストールすると `/Applications` です。
- symlinkを作るパスが `/usr/bin` になってるサンプルがある  
    …macOSはEl Capitanから[システム整合性保護](https://support.apple.com/ja-jp/102149)によって `/usr/bin` には `sudo` でも書き込めなくなったのでした。`/usr/local/bin` にパスが通っていたので、そこにsymlinkを貼りました。存在しなければ作ってパスを通しましょう。

### ビルド

原稿はMarkdownで執筆しています。数式もコードも出てこないので単にもりもり書いただけです。以下の手順で書籍にしました。

#### ePub

MPEの `Open Preview` メニューでプレビューを生成したら、右クリック→eBook→ePubで出力します。

見た目はCSSで変更できます。MPEの `Customize CSS (Workspace)` を実行し、生成された `style.less` を修正します。

#### PDF

MPEの `Open Preview` メニューでプレビューを生成したら、右クリック→eBook→HTMLで、まずHTMLを出力します。直接PDF生成はちょっと大変そうだったので…。

作成されたHTMLをChromeで開き、PDFとして印刷するというアドホックな方法でも、いちおう読めるPDFには仕上がりました。

`style.less` に `@media print` というセクションを作ると、印刷時だけ適用されるスタイルを指定することができます。

{{< highlight css >}}
@media print {
    h1, h2 {
        break-before: page;
    }
}
{{< / highlight >}}

のようにすると、h1（部）とh2（章）では改ページ、などの指定ができます。ちょっと本としてそれっぽくなりました。

#### 表紙

[MyEdit](https://myedit.online/jp/photo-editor)のAI画像生成が作ってくれたやつに、[Pixelmator Pro](https://www.pixelmator.com/pro/)でテキストを入れて雑に作りました。フォントは[墨東レラ](https://www.type-labo.jp/Hanpubokutohrera.html)です。

#### 物理書籍

オフライン出展なら物理本もあったほうがいいので、急遽キンコーズに駆け込んで作ってきました。中綴じの同人誌をそれっぽく作る[ノウハウ](https://www.kinkos.co.jp/column/saddle-stitching-function-copy-machine/)も道具も全部あって、非常に助かりました。

##### 原稿印刷

ChromeのPDF出力はB5サイズが指定できません。なので、A4で作ったものをプリンターの機能でB5に縮小印刷しました。ついでにページ番号も付与しています（これは見た目がイマイチだったので残念）。表紙は600dpiでB5になるpng画像をカラープリントします。

いずれもUSBメモリでキンコーズに持ち込み、[セルフPCレンタル](https://www.kinkos.co.jp/service/self-pc)というサービスで自分で印刷します。

##### コピーと製本

表紙用にちょっといい紙を購入しました。見本が展示してあるので「10番をB4で**枚」みたいにオーダーすると、その場でカットしてくれます。その紙に表紙と背表紙を1枚にプリントして用意しておきます。

あとは複合機が全部やってくれました。B5に印刷した原稿をページ順に並べて、中綴じ印刷表紙付きを指定すれば、中綴じ用にページを割り付けて印刷し、手差しトレイから印刷済みの表紙をフィードして、真ん中で折ってホチキスで綴じたものが出てきます。すごい！

### 完成

{{< figure src="/images/2023/1111/kinkos_is_nice.jpg" width="480" >}}

キンコーズの複合機が中綴じ済みの物理本をぺっと吐き出してくれる体験は、なるほどこれが同人誌の醍醐味だな！と感じさせてくれるものでした（たぶんちがう）。印刷を想定してないところから急いで作ったのでクオリティはお察しですが、よかったらお求めください。

## 11/12 あ23 河童書房をよろしくお願いします

ということで、すでにオンラインでは販売が始まった「[メタエンジニアリング 理論と実践、そして未来](https://techbookfest.org/product/1Vu6ceKBjEWu6Bv4WcBhYs)」、明日11/12はサンシャインシティの展示ホールDのオフライン会場でも頒布します。「[あ23 河童書房](https://techbookfest.org/organization/50060059)」でお待ちしています！
