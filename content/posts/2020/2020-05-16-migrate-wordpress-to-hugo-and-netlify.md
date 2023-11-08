---
title: "WordPressのBlogをHugoとNetlifyに移行する"
date: 2020-05-17T10:00:00+09:00
url: /2020/05/17/migrate-wordpress-to-hugo-and-netlify
aliases:
    - /posts/migrate-wordpress-to-hugo-and-netlify
---

この「Kwappa研究開発室」は2008年に[ココログ](https://www.cocolog-nifty.com/)で書き始めたものです。2011年の正月にVPS上のWordPressに移転し、以降はずっとそこでホスティングしてきました。

技術以外のブログや静的サイトも全部ここでホスティングして、さらにはSlack Botや定時タスクを動かしたり…と長年お世話になっていたのですが、さすがにこのクラウド時代に自前でホスティングを維持するのもしんどいものです。ましてや、[今の職場](https://kwappa.net/blog/archives/2189/)は[AWSでおなじみの会社](https://classmethod.jp/)ですし、もうちょっとモダンにしたいという気持ちも日に日に強まっていました。

そんなタイミングでVPSから年間契約更新の連絡が来たので、これをきっかけに引っ越すことにしました。

{{< figure src="/images/2020/05/hugo-and-netlify.png" >}}

## 移転の要件

- `www.kwappa.net`
  - Apache + PHP + MySQL
  - `/blog` 以下にWordPressを設置
- `randd.kwappa.net`
  - 同じVPSに別のWordPressを設置
- 不満だったこと
  - メンテナンスが大変、手が回ってないことがストレス
    - ApacheもPHPもMySQLも古い
    - PHPが古いからWordPressも古い
    - バックアップは日次でDropboxに投げている
      - これもちょいちょい失敗しててストレス
  - まれによく落ちる
    - ちょっと記事がバズると大変
    - それ以外にも気がつくとおなじみ「データベース接続エラー」
      - 午前3時に `REPAIR TABLE` 打ったり
  - 記事の掲載が大変
    - Markdownで書く→手元でHTML作成→WordPressのエディタで仕上げ、という工程がほんとにめんどい
- 移転で実現したいこと
  - 静的にビルドしてマネージドなホスティングに置きたい
  - HTTPS接続にしたい
  - サブドメイン `www` を外したい
  - 既存記事のURLはそのまま維持したい
  - Markdownで書いたものをそのままデプロイしたい

これらを検討した結果、WordPressのコンテンツを[Hugo](https://gohugo.io/)にエクスポートし、[Netlify](https://www.netlify.com/)でホスティングすることにしました。

<!--more-->

S3 + CloudFrontじゃないの？というツッコミも聞こえてきそうですが、今回のような用途では[Netlify](https://www.netlify.com/)の便利さが圧倒的だったので…。

とはいえぼちぼち苦労したので、ログを残しておくことにします。

## 転出作業

### WordPress → Hugo

WordPressからHugoで動く形にエクスポートするには、[WordPress to Hugo Exporter](https://github.com/SchumacherFM/wordpress-to-hugo-exporter)というWordPress用のプラグインを使うのが一番簡単でした。

[WordPressの記事をHugoに移行する - Qiita](https://qiita.com/Tebasaki314/items/ec50bbbcc4a76a95c5cf)の通りに

* WordPressにプラグインを追加してエクスポート
  * [https://github.com/SchumacherFM/wordpress-to-hugo-exporter](https://github.com/SchumacherFM/wordpress-to-hugo-exporter)
    * Clone or donwload
    * Download ZIP
  * WordPress ダッシュボード
    * プラグイン→新規追加
    * さっきダウンロードしたzipファイルをアップロード→インストール→有効化

これでとりあえず、記事がmarkdownになったものと画像がダウンロードできます。

#### WordPress on Docker → Hugo

ところが、記事や画像の数が多いと、zipに固める途中で「メモリが足りない」とかエラーが出ることがあります。一時的にVPSのメモリを増やせればいいのですが、古い契約なので動的にスケールアップさせられないインスタンスだったらしく、手詰まりになってしまいました。

仕方ないので手元のMacBookでWordPressを立て、そこでインポート→エクスポートしてみました。

* ローカルにDockerを導入する
  * Docker Desktop for Macがインストール済みだった
    * `brew cask install docker`
* DockerでWordPressを動かす
  * [公式ドキュメント](http://docs.docker.jp/compose/wordpress.html)でだいたいok
  * `http://localhost:8000` のレスポンスが空だったら、 `/wp-login.php` あたりを叩いてみる
* 既存のWordPressからエクスポート → インポート
  * 既存のWordPress ダッシュボード →ツール → エクスポート
    * でかいXMLファイルが落ちてくる
  * Docker上のWordPress ダッシュボード → ツール → インポート
    * さっき落ちてきたXMLファイルをアップロードする

あとはさっきの[WordPress to Hugo Exporter](https://github.com/SchumacherFM/wordpress-to-hugo-exporter)を使ってHugo形式でエクスポートします。

ところがXMLファイルがでかすぎてアップロードできない、なんてyakが発生したので倒したメモを残しておきます。

* PHPの設定 `upload_max_filesize` を増やしてやればいい
  * [DockerでPHPのファイルアップロード上限を上げる方法 - memo](https://web-memo-s.hatenablog.com/entry/2019/02/21/182815)
    * WordPressのコンテナに入る
      * `docker exec -it #{CONTAINER_NAME} bash`
    * `/usr/local/etc/php/conf.d/#{CONF_NAME}.ini` を作る
      * `upload_max_filesize = 10M` ぐらいにしておく
    * Apacheを再起動
      * `/etc/init.d/apache2 reload`
  * コンテナに入ったけどテキストエディタがない
    * [Docker — docker コンテナの中で vim が使えない場合 - Qiita](https://qiita.com/YumaInaura/items/3432cc3f8a8553e05a6e)
      * `apt-get update; apt-get install vim` とか

### Hugo形式にととのえる

エクスポートできたら落ちてきたzipの中身を使ってHugo形式にととのえていきます。zipの中身から構築するよりは、`hugo new site [SITE_NAME]` したディレクトリに必要なファイルを移動して行った方がラクでした。

* 記事ごとのファイル : `/posts/*.md`
  * `/contents/posts` あたりに移動しておく
  * ファイル名が `YYYY-MM-DD-#{記事タイトル}.md` になってるので気になるなら直しておく
    * 日本語ファイル名だいたいロクなことにならないので…
* 画像ファイル : `/wp-content/uploads/`
  * `/static/images` あたりに移動しておく
  * 記事中のパスも修正する必要があるのでsedなりなんらかのスクリプトで頑張る
* Makrdown中のHTMLタグをレンダリングする設定にしておく
  * エクスポートしてきたMarkdownにはHTMLタグがたっぷり含まれてるが、そのままだと `<!-- raw HTML omitted -->` とレンダリングされない
  * `config.toml` でHTMLタグをレンダリングするオプションを指定する
  * [Hugo v0.60以上を使うと、Markdown中のHTMLタグが「raw HTML omitted」となって消えてしまう - My External Storage](https://budougumi0617.github.io/2020/03/10/hugo-render-raw-html/)
{{< highlight toml >}}
[markup]
  [markup.goldmark]
    [markup.goldmark.renderer]
      unsafe = true
{{< / highlight >}}
* gitで管理しておく
  * `.gitignore` に `public/` と `resources/` を追加しておく
    * Netlifyがビルドしてくれるので
  * `themes/` 以下には直接cloneせず `submodule add` する
* `netlify.toml` を作る
  * 公式ドキュメントの[Host on Netlify](https://gohugo.io/hosting-and-deployment/hosting-on-netlify/)のサンプルほぼそのままでよい
  * テーマが決まっていたら `command = "hugo --gc --minify --theme=#{THEME_NAME}"` に直しておく

手元のHugoで動くようになったら、各記事の確認も含めていろいろカスタマイズしてみましょう。よさげだったらGitHubにリポジトリを作ってpushしておきます。

## 転入作業

公式の[Host on Netlify](https://gohugo.io/hosting-and-deployment/hosting-on-netlify/)がとても詳細なのでここを見れば特に問題ないと思います。
今回はGitHubからデプロイするの前提なので、アカウントもGitHub連携で作ればクリックぽちぽちだけで完了します。

* [Netlify](https://app.netlify.com/)でアカウント作成
* `Sites` → `New site from Git` → GitHub上のリポジトリを選択
* しばし待つ

初回ビルドはけっこう時間がかかるようです。Netlifyがつけてくれたサブドメインでコンテンツが表示されていたら、ひとまずは完了。

### 自分のドメインで動かす

各コンテンツは引き続き `kwappa.net` に置きたいので、いろいろ設定をしておきます。手順はやっぱり[公式ドキュメント](https://docs.netlify.com/domains-https/custom-domains/)に従えばよいでしょう。

* Netlify上の各サイトで `Site name` を設定する
  * `#{SITE_NAME}.netlify.app` が割り当てられる
* 使いたいドメインを `Domain alias` として設定する
  * `Domain management` → `Add domain alias` → `#{YOUR_DOMAIN}.example.com`
  * 「ほんとにお前のドメインじゃろな？」と確認される
* `#{YOUR_DOMAIN}.example.com` を `CNAME` レコードとして `#{SITE_NAME}.netlify.app` に向ける
  * Netlify DNSを使っているとエイリアスを作ったら自動的に設定してくれる

今回はサブドメインなしのネイキッドドメイン(Zone Apex / Apex Domain) `kwappa.net` もNetlifyに向けたかったのですが、DNSのサービスてはネイキッドドメインにCNAMEを設定することができないものが多いようです（お名前.comのDNSはできなかった）。なので、DNSもまるっとNetlifyに移行してしまいました。

例によって[公式ドキュメント](https://docs.netlify.com/domains-https/custom-domains/configure-external-dns/#configure-an-apex-domain)をよく読んでみると、「AレコードとしてNetlifyのロードバランサーのIPアドレスいれてええで」と案内されています。こっちでやる手もあったかなー。

### HTTPS接続を強制する

`http://` への風当たりが日に日に強まっています。せっかく重い腰を上げて移行するんだし、HTTPS接続も強制化してしまいましょう。

HTTPS対応そのものは非常に簡単で、Netlifyにサイトを作成すれば自動的にLet's Encryptの証明書を使ってHTTPSを有効にしてくれます。更新も自動でやってくれるっぽいのでありがたい限りです。

HTTPS接続の「強制化」、つまり `http://` へのアクセスを `https://` へとリダイレクトする設定については、実現したもののちょっと手順が不定でした。

### Force TLS Connection

Netlifyの `Site settings` → `Domain management` → `HTTPS` で証明書の状態を確認・更新したり、自前の証明書を使ったりすることができます。ここに「Force TLS Connection」というボタン（やチェックボックス）があれば、それを有効にすることによって自動的にリダイレクトされます。

ところがこのボタンが出ることがあったり、出なかったり、出てないのにリダイレクトされたり、とサイトやタイミングごとに状態が違っていました。サイトが自分のドメインで動いている状態で…

* `Force TLS Connection` が出ている
  * ボタンを押して有効化する
* `Force TLS Connection` が出ていない
  * `http://` でアクセスしたとき `https://` にリダイレクトされる
    * 設定不要
  * `http://` でアクセスしたとき `https://` にリダイレクトされない
    * `_redirects` リダイレクトを設定する

### _redirects

Netlifyでは、ドキュメントルートに `_redirects` というファイルを置くことで細かくリダイレクトを制御することができます。`mod_rewrite` に四苦八苦したのを思い出しますね（老人会）。

とはいえあれより全然簡単です。例によって[公式ドキュメント](https://docs.netlify.com/routing/redirects/#syntax-for-the-redirects-file)がありますし、内容をチェックする[Playground](https://play.netlify.com/redirects)もあります。

Hugoの場合は `/static` 直下に `_redirects` を作成し、そこに内容を書いていきます。今回は以下の通りで、 `www` サブドメインつきのアクセスもネイキッドドメインにリダイレクトしています。はてブが分散しちゃうけど、過去記事にそんなにたくさんつくこともあるまい、と割り切ることにしました。

{{< highlight text  >}}
http://www.kwappa.net/* https://kwappa.net/:splat 301!
http://kwappa.net/* https://kwappa.net/:splat 301!
https://www.kwappa.net/* https://kwappa.net/:splat 301!
{{< / highlight >}}

書けたら中身を[Playground](https://play.netlify.com/redirects)に貼り付け、 `Test rules` を押して確認しましょう。エラーが出ていなければ、デプロイして実際の動作を確認します。

## ということで

だいたい移転は完了しました。これで「データベース接続エラー」ともバックアップともおさらばだ！過去記事はちょいちょい体裁が崩れているので（特にSyntax Highlightとか全滅）、ちょっとずつ直していくことにします。

けっこう大変で面倒だったけど、それなりに楽しい作業でした。記事を書くのもラクになったし、もっと頻繁に書くようにしたいところです。
