---
title: "lsyncdからfswatchへの移行で学んだコネタたち"
date: 2024-01-29T09:00:00+09:00
url: /2024/01/29/lsyncd-to-fswatch
---

パートタイムでお手伝いしているプロダクトで、開発用サーバーへの同期ツールが動かない、という問題がありました。解決の過程で、いくつかのちょっとした学びがあったのでまとめておきます。特に目新しいものはありませんが、狭い特定の用途では役立つかもしれません。

{{< figure src="/images/2024/0129/mimi-thian-ZKBzlifgkgw-unsplash.jpg" width="100%" >}}
<div class="photo-caption">
<a href="https://unsplash.com/ja/%E5%86%99%E7%9C%9F/macbook-pro%E3%82%92%E8%A7%A6%E3%81%A3%E3%81%9F%E3%82%8A%E5%90%91%E3%81%91%E3%81%9F%E3%82%8A%E3%81%99%E3%82%8B%E4%BA%BA-ZKBzlifgkgw?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Unsplash</a>の<a href="https://unsplash.com/ja/@mimithian?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Mimi Thian</a>が撮影した写真
</div>

## おしながき <!-- omit in toc -->
- [複数のシェルスクリプトで共通の設定値を扱う](#複数のシェルスクリプトで共通の設定値を扱う)
- [Luaスクリプトに設定値を埋め込む](#luaスクリプトに設定値を埋め込む)
- [正規表現のデリミタ（区切り文字）を変更する](#正規表現のデリミタ区切り文字を変更する)
- [`fswatch` で更新を監視して同期する](#fswatch-で更新を監視して同期する)
- [`rsync` の引数 `SRC` には末尾スラッシュをつける](#rsync-の引数-src-には末尾スラッシュをつける)
- [Markdown のコードブロックに `` ` `` （バッククオート）を書く](#markdown-のコードブロックに--バッククオートを書く)

## 前提
- 手元での修正を `EC2` 上の開発環境に同期する、という開発ワークロード
- 初回はどかっと `rsync` で同期する
- 開発中は変更があったファイルを監視し同期する
  - `lsyncd` を使用
- いくつか課題があった
  - 設定値が複数のシェルスクリプトにハードコードされていた
  - 環境による差分があるので別スクリプトに分割されていた
    - 手元の `rsync` のパスが環境によって違うとか
  - `lsyncd` が動かない環境が出てきた
    - [fsevents event type missing · Issue #204 · lsyncd/lsyncd](https://github.com/lsyncd/lsyncd/issues/204)

<!--more-->

## 複数のシェルスクリプトで共通の設定値を扱う

設定値が複数のスクリプトにハードコードされていたので共通化した。

- `config.txt` のような設定ファイルを用意し、それぞれのスクリプトで `source` する
  - 個別の環境によって設定値が変わる場合
    - `config.txt.sample` のようなファイルに設定例を記述しgitリポジトリに含める
    - `.gitignore` で `config.txt` をバージョン管理から外しておく
    - 各自の手元で `config.txt` という名前でコピーして使う
  - バッククオートによるコマンド実行の結果も取れる
    - 例 : ``RSYNC_COMMAND=`which rsync` ``
  - ホームディレクトリは `~` では取れないので環境変数 `HOME` を使う
    - 例 : `SRC_DIR="${HOME}/project"`
- 例 : `config.txt.sample`
```bash
RSYNC_COMMAND=`which rsync`
SRC_DIR="${HOME}/project"
DST_DIR="example.com:/path/to/target"
SSH_KEY="${HOME}/.ssh/id_rsa"
```

## Luaスクリプトに設定値を埋め込む

[lsyncd](https://github.com/lsyncd/lsyncd) の設定ファイルは [Lua](https://www.lua.org/) で記述されている。こちらも設定値がハードコードされていたので共通化した。

- Luaでは `os.getenv('KEY')` で環境変数が取得できる
  - https://www.lua.org/pil/22.2.html
- しかし `lsyncd` の設定ファイルとして動くときは取得できないようだ（要出典）
- しかたないので [envsubst](https://www.gnu.org/software/gettext/manual/html_node/envsubst-Invocation.html) を使った
  - `lsyncd_config.lua.template` を用意しておく
    - `.gitignore` で `config.lua` をバージョン管理から外しておく
    - 環境変数で置き換えたいところをシェルスクリプトと同様の記法で記述しておく
      - 例 : `source = "${SRC_DIR}"`
    - 例 : `lsyncd_config.lua.template`
```lua
sync {
    default.rsync,
    source    = "${SRC_DIR}",
    target    = "${DST_DIR}",
    delay     = 15,
    rsync     = {
        binary   = "${RSYNC_COMMAND}",
        archive  = true,
        compress = true
    }
}
```
  - 実行時に `config.lua` を生成する
    - 例 :
```bash
envsubst < lsyncd_config.lua.template > lsyncd_config.lua
```
  - できあがった `config.lua` を使って `lsyncd` を起動する
    - 例 : `lsyncd lsyncd_config.lua`

## 正規表現のデリミタ（区切り文字）を変更する

パスの文字列には `/` が多用されるが、正規表現のデリミタもデフォルトが `/` であるため、エスケープする必要がある。面倒だし読みにくくなるので、デリミタを変更すると便利。

referred to : [sedの正規表現のデリミタに、/ 以外を使用する #Ruby - Qiita](https://qiita.com/sekineh/items/333e0a577e476e3018b6) :sushi:

- 例 : ローカルで変更があったファイルパスをリモートの同期先パスに置換する
  - `target` には変更があったファイルのフルパスが格納されている
  - 以下はデリミタを `!` に変更している
```bash
target_file=`echo ${event} | sed "-e s!${SRC_DIR}!${DST_DIR}/!"`
```

## `fswatch` で更新を監視して同期する

`lsyncd` が手元のmacOSでうまく動かなかったため、ファイルの更新監視を [fswatch](https://emcrisostomo.github.io/fswatch/) で行うようにした。

referred to : [[macOS] Terminalでファイルを監視し更新されたら指定の処理をする - fswatch - ねこの足跡R](https://blog.katsubemakito.net/macos/fswatch) :sushi:

- 例 : `rsync_with_fswatch.sh`
```bash
#!/bin/bash
source ./config.txt
ssh-add "${SSH_KEY}"

# 監視するディレクトリの存在チェック
if [[ ! -d ${SRC_DIR} ]]; then
  echo "[error] Not Found target directory ${TARGET_DIR}" >&2
  exit
fi

fswatch -0 ${SRC_DIR} | while read -d "" event; do
  # ignoreパターンにマッチしたら処理をスキップ
  if [[ \
    ${event} =~ \.git\/ || \
    ${event} =~ \/logs\/ || \
    ${event} =~ \.DS_Store \
  ]];then
    continue
  fi

  # 変更があったローカルのファイルパスを同期先のパスに変更する
  target_file=`echo ${event} | sed "-e s!${SRC_DIR}!${DST_DIR}/!"`
  echo "sync ${event}"

  # ローカルの削除を反映するため `r` と `delete` オプションをつける
  # `rsync` する範囲が狭いほど速いので、変更があったファイルが置かれたディレクトリを同期する
  ${RSYNC_BINARY} \
    -rv \
    --delete \
    `dirname ${event}`/ `dirname ${target_file}`
done
```

- 真面目にdaemonizeするほどヘビーなユースケースでもなさそうなので、同期専用のターミナルを開いておく、で割り切った
- `[TODO]` 1ファイルずつsyncするのであれば、ローカルの存在をチェックして `scp` と `rm` を使い分ければより速くなりそう

## `rsync` の引数 `SRC` には末尾スラッシュをつける

`rsync` 関連のスクリプトを書くと毎回忘れてうわっとなるので自戒を込めて。

ディレクトリをsyncするとき、末尾スラッシュが…

- ある場合 : そのディレクトリ以下をsyncする
- ない場合 : そのディレクトリ **そのもの** をsyncする

今回のようにあるディレクトリ以下をsyncする場合は以下のように書くことで、 `/src/foo/bar` 以下の変更があったファイルが `/dst/foo/bar` にsyncされる。

```bash
rsync /src/foo/bar/ /dst/foo/bar
```

末尾スラッシュを忘れると、 `/src/foo/bar` **そのもの** が `/dst/foo/bar` にsyncされるので、結果として `/dst/foo/bar/bar` ができあがってしまう。

## Markdown のコードブロックに `` ` `` （バッククオート）を書く

この記事にはバッククオートが多用されているため調べた。

referred to : [Markdownでコードブロックにバッククオートを含める方法](https://zenn.dev/ttskch/articles/c26308bbcdb800) :sushi:

つまり、こう書くと

{{< figure src="/images/2024/0129/backquote_md.png" >}}

こうなる。

{{< figure src="/images/2024/0129/backquote_rendered.png" >}}

## まとめ <!-- omit in toc -->

ひとつひとつはコネタだし知ってる人には常識みたいなことですが、それらの蓄積が問題解決になるんだなあ、と改めて認識したタスクでした。需要があるかは気にせず記事にするのもひさしぶりで、アウトプットする楽しさを思い出しました。

ということでまとめとしては…

- 日々の仕事の問題解決をアウトプットするのは大事だし楽しい
- アウトプットは謙虚に、でも萎縮せず
- `rsync` では末尾スラッシュを忘れるな

です！

see also : [「日々のアウトプットが変える！あなたのエンジニア・ライフ」というイベントに登壇してきたよ #forkwell // Kwappa研究開発室](/2018/10/10/705/)
