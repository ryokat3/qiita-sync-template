<!--
title:   GitHub連携でQiita記事を素敵な執筆環境で！
tags:    GitHub,GitHubActions,Python,Qiita,個人開発
id:      d054b95f68810f70b136
private: false
-->
# 「素敵な執筆環境」とは？

心地よいソファーだったり、甘えん坊だけどキーボードの上だけは避けてくれる猫のことではなく、vi とか emacs とか vscode とか、お気に入りのエディタを使った執筆環境を実現するために開発した [Qiita Sync](https://github.com/ryokat3/qiita-sync) の紹介です。

https://github.com/ryokat3/qiita-sync

## Qiita の記事を執筆する時の不満

個人的には以下のような Qiita 公式の Web アプリによる執筆時の不満を解消するため、この執筆環境を開発しました。

- Web アプリという性質上、マウスの使用を強要されたり、慣れたエディタのキーバインドが操作ミスになったり（Backspace 代わりの Ctrl-H で履歴画面を見せられる...）、個人的にイライラすることが多い。

- 図を更新の際に、Qiita のサイトに upload されたファイルを直接編集することはできず、図のファイルをローカルにコピーし、保存管理しておく必要がある。

- Markdown の Table は等幅フォントで編集したい。

## vi で記事を書いて GitHub に push するだけ

notepad でもいいですが、とにかくあとは Qiita Sync にお任せです。

1. Qiita の記事を vim で書いて、GitHub に push 
2. GitHub Actions が自動で Qiita に記事を upload （下図）

![Qiita Sync](https://raw.githubusercontent.com/ryokat3/qiita-articles/main/img/qiita_sync.drawio.png) [^1]

## 記事の同期も自動でチェック

Qiitaの記事をブラウザでチャチャっと作ったり、更新したり、そんな時は GitHub との同期が取れなくなることもあります。でも大丈夫、同期がとれないことは、GitHub の画面で確認できるし、GitHub からメールのお知らせが届きます。

記事の差分が確認できたら、クリックひとつで再び同期させることもできます。

1. Qiita の記事をブラウザで更新
2. GitHub Actions が定期的に記事の同期をチェック
3. 同期が取れていれば GitHub の GUI に緑のバッジ、そうでなければ赤のバッジを表示
4. 同期が取れていない時は GitHub からメールで通知

![Qiita Sync Check](https://raw.githubusercontent.com/ryokat3/qiita-articles/main/img/qiita_sync_check.drawio.png) [^1]

## インストールしなくていい、使い方も覚えなくていい

メインの機能を提供する Qiita Sync は python の CLI コマンドですが、GitHub Actions 上で動作するので、コマンドをインストールしたり、使い方や引数を覚えたりする必要はありません。もちろん python のインストールも不要です。

# 準備

## Qiita Access Token の生成

記事の投稿に [Qiita API v2](https://qiita.com/api/v2/docs) を使うので秘密鍵である Access Token が必要になります。Access Token は Qiita のユーザ画面から、

1. [Qiita Account Applications](https://qiita.com/settings/applications) を開く
2. "Generate new token" をクリック
3. "Desciption" は適当な説明を入力。
4. "Scopes" の "read_qiita" と "write_qiita" をチェック（下図）
5. "Generate token" をクリック
6. 生成された Access Token はコピーして保存しておく

![Qiita Access Token 生成画面](https://raw.githubusercontent.com/ryokat3/qiita-articles/main/img/generate_qiita_access_token.png)

## Qiita Access Token の登録

Qiita 同期をする GitHub の repository を一つ用意する。できれば専用の repository を用意することをお勧めします。

1. GitHub repository の GUI から Settings >> Secrets で "Actions secrets" の画面を表示
2. 右上の "New repository secret" のボタンをクリック
3. Name には `QIITA_ACCESS_TOKEN` と入力
4. Value には Qiita で生成した Access Token を入力（下図）
5. "Add secret"をクリックして登録完了

![GitHub Access Token 登録画面](https://raw.githubusercontent.com/ryokat3/qiita-articles/main/img/github_save_access_token.png)

## GitHub Actions の設定

以下の２つの YAML ファイルを作成します。

- [.github/workflows/qiita_sync.yml](https://raw.githubusercontent.com/ryokat3/qiita-sync/main/github_actions/qiita_sync.yml)
- [.github/workflows/qiita_sync_check.yml](https://raw.githubusercontent.com/ryokat3/qiita-sync/main/github_actions/qiita_sync_check.yml)

どちらのファイルも基本的にこのまま変更なしに使用できます。

ただし `qiita_sync_check.yml` の `cron: "29 17 * * *"` の部分は変更をお願いします。利用者全員が同じ時間をになると、GitHub にも Qiita にも一斉に負担がかかるので、それを避けるためです。

:::note warn
cron の時間設定は変更する
:::

下記の例 `29 17 * * *` は 17:29 UTC なので日本時間だと毎日 02:29 JST に起動することになります。週一の起動でも構いません。

```yaml:.github/workflows/qiita_sync_check.yml
name: Qiita Sync Check

on:
  schedule:
    - cron: "29 17 * * *"
  workflow_run:
    workflows: ["Qiita Sync"]
    types:
      - completed
  workflow_dispatch:

jobs:
  qiita_sync_check:
    name: qiita-sync check
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2
      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.9'
      - name: Install qiita-sync
        run: |
          python -m pip install qiita-sync
      - name: Run qiita-sync check
        run: |
          qiita_sync check . > ./qiita_sync_output.txt
          cat ./qiita_sync_output.txt
          [ ! -s "qiita_sync_output.txt" ] || exit 1
        env: 
          QIITA_ACCESS_TOKEN: ${{ secrets.QIITA_ACCESS_TOKEN }}
```

`qiita_sync.yml` は Qiita と GitHub の内容を比較して、内容に差異がある場合は最終更新時間が新しい方を正とします。Qiita が新しい場合には download、GitHub が新しい場合には upload を行います。

GitHub のデフォルトのブランチ名が `main` なので、この GitHub Actions は `main` に push された時起動します。もしブランチ名に `master` など他の名前を使われている方は `on.push.branches` の `main` を `master` に変更してください。

```yaml:.github/workflows/qiita_sync.yml
name: Qiita Sync

on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  qiita_sync_check:
    name: Run qiita-sync sync
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2
      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.9'
      - name: Install qiita-sync
        run: |
          python -m pip install qiita-sync
      - name: Run qiita-sync
        run: |
          qiita_sync sync .
        env: 
          QIITA_ACCESS_TOKEN: ${{ secrets.QIITA_ACCESS_TOKEN }}
      - name: Git
        run: |
          find . -name '*.md' -not -path './.*' | xargs git add
          if ! git diff --staged --exit-code --quiet
          then
            git config user.name github-actions
            git config user.email github-actions@github.com
            find . -name '*.md' -not -path './.*' | xargs git add
            git commit -m "updated by qiita-sync"
            git push
          fi
```

この２つのファイルを GitHub に push すると同期が始まります。最初にインストールした時のファイル名は __最初に記事を作成した日付 + タグ + 記事の ID + .md__ になります。ファイルの拡張子が `.md` である限りは、ファイル名の変更やディレクトリの移動は自由なので、`git pull` した後は分かりやすいファイル名に変更してください。ただし `README.md` というファイルは同期の対象から外されています。

:::note info
ファイル名の変更や移動は自由
:::

![Qiita-Sync initial download](https://raw.githubusercontent.com/ryokat3/qiita-articles/main/img/qiita_sync_initial_download.png)

## バッジの設定

README に以下の画像リンクを追加すると、同期の成否を示すバッジが表示されます。成功のバッジが表示されていると執筆の意欲も沸くのでおすすめです。

`<Your-ID>` と `<Your-Respository>` の部分はあなたのものに置き換えてください。

```markdown:バッジの画像リンク
![Qiita Sync](https://github.com/<Your-ID>/<Your-Repository>/actions/workflows/qiita_sync_check.yml/badge.svg)
```

以下のバッジが表示されます。

- 成功した場合:

  ![Passing Badge](https://raw.githubusercontent.com/ryokat3/qiita-articles/main/img/qiita_sync_badge_passing.png)

- 失敗した場合:

  ![Failing Badge](https://raw.githubusercontent.com/ryokat3/qiita-articles/main/img/qiita_sync_badge_failing.png)

# 同期

記事を git で push すると自動的に同期が始まるので、通常手動で同期を行うことはありません。

ただ、Qiita の Web アプリケーションで記事を更新すると、次の cron 起動じに上記の失敗した場合のバッジが表示されます。同時に GitHub に登録したメールアドレス宛にも通知が行きます。その他、複数の新しい記事を一度にダウンロードする場合などに失敗することがあります。

そのような場合には、以下の手順で GitHub Actions を実行し、記事を同期させるようにします。

1. GitHub repository を開く
2. "Actions"、"Qiita Sync" を開く
3. "Run workflow" をクリックする（下図）

![Qiita Sync manual execution](https://raw.githubusercontent.com/ryokat3/qiita-articles/main/img/qiita_sync_manual_execution.png)

# 記事の執筆

お気に入りのエディタで markdown を編集するのですが、記事の執筆時に幾つかの注意点があります。

## 記事のヘッダ

Qiita からダウンロードした記事には以下のようなヘッダがファイルの先頭に自動的に付加されます。

`title` や `tags` は自由に変更できますが、`id` を変更したり、消去したりすることはできません。一方 `id` は他の記事と共用はできないので、ファイルをコピーする時には `id` を消去してください。

:::note alert
ヘッダの id の取扱は注意する
:::

```markdown:通常のヘッダ
<!--
title: This header is automatically generated by Qiita-Sync when downloading Qiita articles
tags:  Qiita-Sync
id:    a5b5328c93bad615c5b2
-->
```

## 新しい記事の作成

新しい記事を作成する場合には、ヘッダに `id` は不要です。Qiita-Sync が、記事を Qiita にアップロードした後に `id` をファイルのヘッダに付加します。GitHub 上で Qiita-Sync がファイルの一部を書き換えることになるので、`git pull` などで local も最新に追従するようにしてください。

:::note warn
新しい記事を追加した後には同期後に git pull しておく
:::

```markdown:新規作成時のヘッダ
<!--
title: No id is necessary in the header when writing new articles
tags:  Qiita-Sync
-->
```


## 他の記事へのリンク

同じユーザの他の Qiita の記事へのリンクは、以下のようにファイルの相対パスで指定することができます。

```markdown:編集中のリンク
<!-- An example of link to another Qiita article when writing -->
[My Article](../my-article.md)
```

Qiita にアップロードされる際に自動的にURLに変換されます。

```markdown:アップロード時のリンク
<!-- An example of link to another Qiita article when published to Qiita site -->
[My Article](https://qiita.com/ryokat3/items/a5b5328c93bad615c5b2)
```

ダウンロード時には再び相対パスのリンクに変換されます。

## 画像ファイルへのリンク

画像ファイルへのリンクは、以下のように相対パスで指定することができます。

```markdown:編集中のリンク
<!-- An example of link to image file 'earth.png' when writing-->
![My Image](../image/earth.png)
```

Qiita にアップロードされる際に自動的にURLに変換されます。

```markdown:アップロード時のリンク
<!-- An example of link to image file 'earth.png' when published to Qiita site -->
![My Image](https://raw.githubusercontent.com/ryokat3/qiita-articles/main/image/earth.png)
```

ダウンロード時には再び相対パスのリンクに変換されます。


[^1]: [図で使用した画像素材](https://www.pinterest.com/pin/create/button/?url=https%3A%2F%2Fpngtree.com%2Ffreepng%2Fman-working-on-computer-at-home-isometric-vector_4000330.html?share=3&media=https://png.pngtree.com/png-vector/20190219/ourlarge/pngtree-man-working-on-computer-at-home-isometric-vector-png-image_321818.jpg&description=Man+working+on+computer+at+home+isometric+vector) は [Man png from pngtree.com/](https://pngtree.com/so/Man) のものを使用しています。