---
title: "ハンズオンの準備"
free: true
---

## 3-1. インストール

まずは、いかからDocker Desktopのインストーラーをダウンロードしてインストールします。
Windows、Macそれぞれ自分の環境にあったインストーラを使用しましょう。
インストール後ライセンスの注意事項を読んで次へ進みます。（前の章で話した企業の規模の話です）
使用目的やログインを求められますがすべてスキップして問題ありません。

https://www.docker.com/ja-jp/products/docker-desktop/

## 3-2. インストール確認

:::message
今回のハンズオンでは、Docker DesktopのGUIはほぼ使いません。
基本は、コマンドでの操作を行います。
Windows, MacともにVSCodeのターミナルを使用して操作することを想定しています。
ファイルを作成しながらコマンドを打つことになるからです。
:::

以下をターミナルに入力しEnterを押下します。

```bash
$ docker ps
```

すると以下のような表示が出ると思います。
これは、今のコンテナの状態を表しています。
起動しているコンテナがあればここに表示されます。
今は、こんなふうに表示されるんだなぐらいで良いです。

```
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

## 3-3. コンテナを立ち上げてみよう

Dockerの公式イメージ（コンテナを作る元）を使用してコンテナを立ち上げます。
以下コマンドをターミナルに入力しEnterを押下します。

```bash
$ docker run hello-world
```

すると以下のような表示が出てきたと思います。
出てきていれば成功です。
このイメージは、Dockerが動いているかを試すのにうってつけな最小限のイメージです。

```
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (arm64v8)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

ここまでで実は、以下が行われていました。

1. Docker Hubから`Hello-World`イメージをPull（ダウンロード）する
2. `Hello-World`イメージを使用してコンテナを立ち上げる
3. 標準出力（ターミナル）にHello World!の文字が出力される

`docker run`コマンドがここまで一気に行なってくれています。
コンテナを立ち上げるのには、いくつか処理のステージがあるのですが
このコマンドは、それらをまとめて実行してくれます。

Hello-Worldイメージの説明はこちら
https://hub.docker.com/_/hello-world

## 3-4. コンテナが終了するタイミングはいつか

前章の`Hello-World`イメージは、標準出力に文字を出力するとコンテナが終了します。
なぜか、、
コンテナは、コンテナ内で起動しているプロセスが０になると終了する仕組みになっています。
今回は、文字を出力するプログラム（プロセス）が文字を出力して終了したためコンテナも終了しました。
立ち上げたままにするためにはWebサーバのような常駐するプロセスを動かしていれば立ち上げっぱなしにできます。
簡単に試してみましょう。
以下コマンドを入力しEnterを押下してください。

```bash
$ docker run -it ubuntu bash
``` 

すると以下のような表示になると思います。
この表示になっていれば成功です。

```
root@{ほにゃらら}:/#
```

今の状態は、ubuntuイメージをもとにコンテナを立ち上げ、さらにターミナルでログインしている状態です。
平たく言うと、ubuntuコンテナの中に入っていることになります。※常駐するプロセスを再現している状態です。
`ls -l`などするとコンテナのディレクトリ構造が見れるかと思います。

さらにこの状態で、別のターミナルを開いてください。
そこで以下コマンドを入力しEnterを押下してください。

```bash
$ docker ps
```

`hello-world`イメージを使用した時とは違い何かしらコンテナの情報が出てきていると思います。
ubuntuコンテナにbashというシェルを使用してログインしているのでプロセスが一つコンテナの中で動いています。
つまりプロセスが０ではないのでコンテナが立ち上がりっぱなしになっています。
立ち上がっているコンテナは、上記のコマンドで確認が可能です。

左から以下のようになっています。
- コンテナID：自動採番
- コンテナ起動に使用したイメージ：イメージ名
- コンテナ起動時の初期コマンド：起動時のコマンド文字列
- コンテナがいつ作成されたか：どれくらい前に作られたか
- 現在のコンテナステータス：立ち上がっているのか終了しているのかなど
- 使用しているポート：コンテナが使用しているポートが表示されます。ポートの話は、後述します。
- コンテナ名：何も決めなければデフォルトの名前が使われます。

では、元のターミナルに戻ってログアウトしましょう。
`exit`と入力してEnterを押下するか。
WindowsならCtrl＋D。Macならcontrol＋Dでログアウトします。

このターミナルのままもう一度`docker ps`コマンドを入力しEnterを押下します。
そうすると今度は、ヘッダーだけ(CONTAINER ID...)の表示になっていると思います。
bashのプロセスが終了しubuntuコンテナのプロセスが０になったためコンテナが終了しました。

## 3-5. 終了したコンテナはどう確認する？

終了したコンテナは、以下コマンドで確認ができます。
以下を入力しEnterを押下します。

```bash
$ docker ps -a
```

すると、いくつかコンテナの情報が出てきたと思います。
１行が１コンテナの情報です。
何回か`docker run`コマンドを叩いているといくつかコンテナが作られていると思います。
`docker run`コマンドがすでにあるコンテナを立ち上げるわけではなく１からコンテナを作り直すからです。

`docker ps -a`は、実はよく使います。
コンテナを立ち上げたけど何かエラーが起きて立ち上がらなかった時は、`docker ps`だけでは、一覧に出てこないのでぱっと見何が起きたかわかりません。
どうなっているか確認したいときは、`docker ps -a`で作成されているコンテナの一覧を見るのが早いです。
