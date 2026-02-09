---
title: "ハンズオン-Dockerfile編②"
free: false
---

## Dockerコンテナを作る上で重要なDockerfileについて理解しよう

今まで既存のイメージを使ってコンテナを起動してきました。
実はそのベースイメージもDockerfileから作られています。

自分でDockerfileを書くのは、独自のアプリケーションをコンテナで動かしたい時です。
例えば、Node.jsアプリなら、ベースイメージに自作アプリをコピーしてビルド、起動する流れをDockerfileに書きます。

Dockerfileは、コンテナ作成時に実行するコマンドを記述したファイルです。
シェルスクリプトのようなイメージですね。

## Dockerfileがあると何がいいのか

コンテナ作成の手順がファイルに書き起こされていることでそのファイルを渡しコンテナを作成するだけで
同じ環境が簡単に作成できるようになります。
この仕組みがあるのでコンテナで開発環境を作る流れが加速しました。

## Dockerfileを書いてWebサーバを作ってみよう

実際にDockerfileを書いて、自分だけのWebサーバコンテナを作ってみましょう。
今回は、HTMLファイルを表示するシンプルなWebサーバを作成します。
プログラミング言語の知識は不要です。Dockerfileの基本的な書き方を学ぶことに集中できます。

## 5-1. プロジェクトの準備

まず、作業用のディレクトリとHTMLファイルを作成します。
好きな場所に`my-website`というディレクトリを作成してください。

```bash
mkdir my-website
cd my-website
```

次に、表示するHTMLファイルを作成します。
以下の内容で`index.html`を作成してください。
body部を好きなように変えてみるのもただコピペするより面白いかもしれません。

```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My Docker Website</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            background-color: white;
        }
        .container {
            text-align: center;
            padding: 2rem;
            border: 2px solid black;
        }
        h1 {
            margin: 0 0 1rem 0;
        }
        p {
            margin: 0;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Welcome to Docker!</h1>
        <p>自分で作成したHTMLファイルが表示されていると思います！</p>
    </div>
</body>
</html>
```

## 5-2. Dockerfileの作成

同じ`my-website`ディレクトリ内に`Dockerfile`という名前のファイルを作成します。
注意：拡張子は不要です。ファイル名は必ず`Dockerfile`にしてください。

`Dockerfile`の内容は以下の通りです。

```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
```

たったの2行です。
これだけでWebサーバコンテナが作れます。

各行の意味を説明します。

### FROM命令

```dockerfile
FROM nginx:alpine
```

`FROM`は、ベースとなるイメージを指定する命令です。
Dockerfileは必ず`FROM`命令から始まります。

- `nginx`：Webサーバソフトウェア（nginxと読みます）
- `alpine`：軽量なLinuxディストリビューション

`nginx:alpine`は、nginxがインストールされた軽量なイメージです。
このイメージを元に、自分のWebサーバコンテナを作ります。

なぜalpineを使うのか？
通常の`nginx`イメージは約140MBですが、`nginx:alpine`は約40MBと非常に軽量です。
必要最小限のものだけが入っているため、セキュリティ的にも優れています。

### COPY命令

```dockerfile
COPY index.html /usr/share/nginx/html/index.html
```

`COPY`は、ホストのファイルをコンテナ内にコピーする命令です。

書式：`COPY [コピー元] [コピー先]`

- コピー元：`index.html`（ホストのファイル）
- コピー先：`/usr/share/nginx/html/index.html`（コンテナ内のパス）

`/usr/share/nginx/html/`は、nginxがHTMLファイルを配信するデフォルトのディレクトリです。
ここに自分のHTMLファイルを配置することで、nginxがそのファイルを表示してくれます。

## 5-3. イメージのビルド

Dockerfileが準備できたら、イメージをビルドします。
`my-website`ディレクトリ内で以下のコマンドを実行してください。

```bash
docker build -t my-website:1.0 .
```

実行すると、以下のような出力が表示されます。

```
[+] Building 2.3s (7/7) FINISHED
 => [internal] load build definition from Dockerfile
 ...(省略)
```

この出力から、Dockerfileの各ステップが実行されていることがわかります。

### tips

dockerfileを作りながらコンテナを起動するとdockerfileで書いた記述が原因でコンテナの起動に失敗することがあります。
その場合は、ビルド中の出力の中でどの記述でエラーになったかが出力されるので原因の特定ができます。
ターミナル上は、素早く出力されていきますがdockerfileを作成しているときのエラー出力の場でもあるのでうまくビルドできない時はよくみるようにしましょう。

### docker buildコマンドの詳細

```bash
docker build -t my-website:1.0 .
```

- `docker build`：イメージをビルドするコマンド
- `-t my-website:1.0`：イメージに名前（タグ）を付ける
  - `my-website`：イメージ名
  - `1.0`：バージョン（タグ）
  - `-t`オプションがないと、IDだけのイメージになり管理しづらくなります
- `.`：ビルドコンテキスト（Dockerfileがある場所）
  - 現在のディレクトリ（`.`）を指定しています
  - Dockerはここにある全てのファイルを使ってビルドします

イメージが作成されたか確認してみましょう。

```bash
docker images
```

`my-website`というイメージが表示されていれば成功です。

```
REPOSITORY    TAG       IMAGE ID       CREATED          SIZE
my-website    1.0       xxxxx          10 seconds ago   43MB
```

## 5-4. コンテナの起動

イメージができたので、コンテナを起動します。

```bash
docker run -d -p 8080:80 --name my-site my-website:1.0
```

コマンドを実行すると、コンテナIDが表示されます。
これは、コンテナが正常に起動した証拠です。

### docker runコマンドの詳細

```bash
docker run -d -p 8080:80 --name my-site my-website:1.0
```

- `-d`：バックグラウンドで実行（デタッチモード）
  - これがないと、ターミナルがコンテナに占有されます
- `-p 8080:80`：ポートマッピング（超重要！）
  - `8080`：ホスト（自分のPC）のポート
  - `80`：コンテナ内のポート
  - 「ホストの8080番ポートをコンテナの80番ポートに転送する」という意味
- `--name my-site`：コンテナに名前を付ける
  - 名前がないと、ランダムな名前が付けられます
  - 名前があると管理しやすくなります
- `my-website:1.0`：使用するイメージ

### ポートマッピングとは

コンテナは隔離された環境です。
コンテナ内でWebサーバが80番ポートで起動していても、そのままでは外からアクセスできません。

ポートマッピング（`-p`オプション）を使うことで、
「ホストの8080番ポートにアクセス → コンテナの80番ポートに転送」
という仕組みを作ります。

```
あなたのブラウザ → localhost:8080 → コンテナ内の80番ポート（nginx）
```

これがDockerの基本的な仕組みです。
めちゃくちゃ重要なので覚えておきましょう。

## 5-5. 動作確認

コンテナが起動しているか確認します。

```bash
docker ps
```

以下のように表示されていれば成功です。

```
CONTAINER ID   IMAGE            COMMAND                  CREATED          STATUS          PORTS                  NAMES
xxxxx          my-website:1.0   "/docker-entrypoint.…"   10 seconds ago   Up 9 seconds    0.0.0.0:8080->80/tcp   my-site
```

`PORTS`の列に`0.0.0.0:8080->80/tcp`と表示されていることを確認してください。
これは、ポートマッピングが正しく設定されている証拠です。

それでは、ブラウザでアクセスしてみましょう。
以下のURLをブラウザに入力してください。

```
http://localhost:8080
```

HTMLファイルで作成したページが表示されれば成功です。
自分だけのDockerコンテナでWebサーバを立ち上げることができました。

### nginxのログを確認する

コンテナが何をしているか、ログを見てみましょう。

```bash
docker logs my-site
```

以下のような出力が表示されます。

```
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
...
2024/01/01 00:00:00 [notice] 1#1: start worker processes
```

ブラウザでアクセスした記録も確認できます。

```
172.17.0.1 - - [01/Jan/2024:00:00:00 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 ..."
```

これは、あなたのブラウザからのアクセスログです。
Webサーバが正常にリクエストを処理していることがわかります。

### リアルタイムでログを監視する

コンテナのログをリアルタイムで監視したい場合は、以下のコマンドを使います。

```bash
docker logs -f -t -n 100 my-site
```

- `-f`オプション：ログをフォロー（follow）し、リアルタイムで表示します
  - tailコマンドの`-f`と同じ動作です
  - 新しいログが出力されると、自動的に画面に表示されます
  - 停止するには`Ctrl + C`を押します
- `-t`オプション：各ログエントリにタイムスタンプを追加します
  - いつログが出力されたかが分かるので、デバッグ時に便利です
- `-n 100`オプション：表示するログの行数を指定します
  - 過去100行分のログを表示してから、リアルタイム監視を開始します
  - 指定しない場合は、全てのログが表示されます

この状態でブラウザを更新すると、アクセスログがリアルタイムで追加されるのが確認できます。
アプリケーションのデバッグや動作確認に非常に便利なので、よく使います。

## 5-6. クリーンアップ

動作確認ができたら、コンテナを停止して削除しましょう。

### コンテナの停止

```bash
docker stop my-site
```

コンテナが停止します。
`docker ps`で確認すると、何も表示されなくなります。

停止したコンテナも含めて確認したい場合は以下を実行します。

```bash
docker ps -a
```

`STATUS`列に`Exited`と表示されていれば、停止しています。

### コンテナの削除

```bash
docker rm my-site
```

コンテナが削除されます。
`docker ps -a`で確認すると、完全に消えています。

### イメージの確認

コンテナは削除しましたが、イメージはまだ残っています。

```bash
docker images
```

`my-website`イメージが表示されているはずです。
イメージがあれば、いつでも同じコンテナを再び作成できます。

もしイメージも削除したい場合は以下のコマンドを実行します。

```bash
docker rmi my-website:1.0
```

## 5-7. よくあるエラーと対処法

### ポートが既に使用されている

```
Error response from daemon: driver failed programming external connectivity on endpoint my-site:
Bind for 0.0.0.0:8080 failed: port is already allocated.
```

これは、8080番ポートが既に別のプログラムで使われている場合に発生します。

**対処法1：別のポートを使う**

```bash
docker run -d -p 8081:80 --name my-site my-website:1.0
```

ブラウザでは`http://localhost:8081`でアクセスします。

**対処法2：使用中のポートを確認する**

Macの場合：
```bash
lsof -i :8080
```

Windowsの場合：
```bash
netstat -ano | findstr :8080
```

### 同じ名前のコンテナが既に存在する

```
Error response from daemon: Conflict. The container name "/my-site" is already in use by container "xxxxx".
```

既に`my-site`という名前のコンテナが存在する場合に発生します。

**対処法：既存のコンテナを削除する**

```bash
docker rm my-site
```

もし起動中の場合は、先に停止してから削除します。

```bash
docker stop my-site
docker rm my-site
```

または、強制的に削除することもできます。

```bash
docker rm -f my-site
```

### イメージがビルドできない

```
ERROR: failed to solve: failed to resolve source metadata for docker.io/library/nginx:alpine
```

Docker Hubに接続できない場合に発生します。

**対処法：ネットワーク接続を確認する**

1. インターネット接続を確認
2. Docker Desktopが起動しているか確認
3. 再度ビルドを実行

## 5-8. まとめ

ここまでで、以下のことを学びました。

1. Dockerfileの基本構文
   - `FROM`命令：ベースイメージの指定
   - `COPY`命令：ファイルのコピー

2. docker buildコマンド
   - イメージのビルド方法
   - `-t`オプションでタグ付け
   - ビルドコンテキストの指定

3. docker runコマンドのオプション
   - `-d`：バックグラウンド実行
   - `-p`：ポートマッピング（超重要！）
   - `--name`：コンテナに名前を付ける

4. Dockerの基本的な流れ
   - Dockerfileを書く → イメージをビルド → コンテナを起動 → 動作確認

Dockerfileは、たった2行でも実用的なWebサーバを作ることができます。
これがDockerの便利なところです。

次のセクションでは、より実用的なDockerfileの書き方を学びます。
パッケージのインストール（`RUN`命令）、環境変数の設定（`ENV`命令）、
作業ディレクトリの指定（`WORKDIR`命令）など、実際の開発でよく使う命令を紹介します。