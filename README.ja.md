# NAT ラボ: Ubuntu VM で nftables によるマスカレードとポートフォワーディングを学ぶ

[English](README.md) | [Français](README.fr.md) | [日本語](README.ja.md) | [Español](README.es.md)

[![check](https://github.com/mitsurutech/lab-nat-gateway/actions/workflows/check.yml/badge.svg)](https://github.com/mitsurutech/lab-nat-gateway/actions/workflows/check.yml)

授業でそのまま使える **NAT ラボ**です。[MitsuruTech ラーニングプラットフォーム](https://mitsurutech.co.uk/ja/learning)の一部で、学校は生徒一人ひとりに専用のマシンとブラウザー上のターミナルを渡せます。生徒は Ubuntu VM を 1 台受け取り、その中に小さなネットワークが組まれています。プライベートな LAN のホスト、**ゲートウェイ**、外部のサーバーの 3 つです。生徒はゲートウェイに **nftables** で**マスカレード（送信元 NAT）**と**ポートフォワーディング（宛先 NAT）**を設定し、再起動しても残るようにルールを保存して、反対側から何が見えるかを自分の目で確かめます。生徒のパソコンには何もインストールしません。

VM は 1 台のマシンなので、3 つはそれぞれ**ネットワーク名前空間**（network namespace）で、各リンクは仮想ケーブルのペアです。本物のコマンドがそのまま使えるのが狙いです: `nft`、`conntrack -L`、`tcpdump`、`curl`、`ip route`。

## ラボ

| ラボ | ファイル | 生徒の作業 | ヘルプページ |
| --- | --- | --- | --- |
| NAT ゲートウェイ | [`nat-gateway.lab.yaml`](nat-gateway.lab.yaml) | プライベートなホストが外部から応答を受け取れない理由を見つけ、ゲートウェイでそのアドレスを変換し、外部からの 1 つのポートを LAN の Web ページに転送し、その通信だけを許可してそれ以上は許可せず、ルールセットを保存する | [NAT ゲートウェイラボ](https://mitsurutech.co.uk/ja/help/lab-nat-gateway) |

ネットワークの構成: `10.0.0.0/24` の LAN ホスト、それと外部ネットワークの間にあるゲートウェイ、`203.0.113.0/24`（ドキュメント用に確保された範囲）、そしてリクエストごとに、そのリクエストが来たアドレスを返す外部サーバーです。インターネットと同じく、外部から LAN への経路はありません。ラボは生徒のホームフォルダーに README を置きます。何をするかは書かれていますが、答えは書かれていません。

ラボには**チェック**があります。提出された VM を、教員が中身を読まなくても採点するスクリプトです。保存されたマシンのコピーを新たに起動した状態で実行するので、生徒が `/etc/nftables.conf` に保存した内容だけが見えます。また、**ルールの書き方ではなく、ネットワークの実際の動作を採点します**。LAN ホストからのリクエストは外部サーバーに届き、外部サーバーはゲートウェイの外側のアドレスを見ていること。外部から転送ポートへのリクエストは LAN のページに届き、そのページはクライアント自身のアドレスを見ていること（つまりゲートウェイ上のプロキシは不合格です）。もう一方の LAN のページとその他すべてのポートは閉じたままであること。そしてゲートウェイの forward チェーンが、許可するよう指示されていない通信を引き続き破棄していること。

## ファイルの中身

`.lab.yaml` は、プラットフォームがそのまま実行する **VM ラボ定義**の全体です。

| キー | 内容 |
| --- | --- |
| `cpus`、`memory`、`disk` | 各生徒の VM のサイズ: 1 CPU、1024 MB、10 GB |
| `open` | ブラウザーで開くポート: なし。ラボはターミナルです |
| `setup` | 各生徒の VM が最初に起動したときに root で一度だけ実行されるスクリプト。Ubuntu のアーカイブから `nftables`、`conntrack`、`curl` といくつかのツールを約 10 秒でインストールし、`netlab` コマンド、トポロジー、開始時の `/etc/nftables.conf`、ラボの README を書き込みます |
| `check` | 採点スクリプト。保存されたディスクのコピーを新たに起動した状態で root として実行します。終了ステータスが結果で、`0` なら合格です |

インストールされる `netlab` コマンド: `netlab status` はマシンの一覧を表示し、`netlab shell gw` はそのマシンの中でシェルを開き、`netlab reset` は開始時のルールに戻してラボを再起動します。

## プラットフォームでの使い方

**クラスの課題として配布するには:**

1. モジュールで課題を追加し、**Ubuntu VM ラボ**のカードを選びます。
2. NAT ゲートウェイのサンプルを押すか、このリポジトリのファイルの内容を貼り付けます。サンプルを使うとテキストが置き換わりますが、元に戻すで自分の内容に戻せます。
3. 公開して、まず自分用のコピーで試します。生徒にはそれぞれ専用の VM が渡され、セッションをまたいで保持されます。
4. 通常の課題と同じように配布します。提出されたら、クラス一覧の生徒の行でチェックを実行します。保存されたマシンのコピーを起動し（保存し戻されることはありません）、合格したかどうかと出力内容が表示されます。

残りはヘルプマニュアルの [Ubuntu VM ラボのガイド](https://mitsurutech.co.uk/ja/help/labs)と[サンプルラボのページ](https://mitsurutech.co.uk/ja/help/sample-labs)をご覧ください。

学校のアカウントはまだありませんか？ 50 人までなら[最初の 1 年間は無料](https://mitsurutech.co.uk/ja/pricing)で、[アカウント作成](https://mitsurutech.co.uk/ja/signup)にはメールアドレスだけが必要です。

## ファイルのチェック

プッシュのたびに、`.github/workflows/check.yml` が、ファイルが上記の 5 つのキーを持つ有効な YAML であること、そして `setup` と `check` がそれぞれプラットフォームの上限 16 KB に収まっていることを確認します。VM は起動しません。それはプラットフォーム自身のマシンで行われます。

## ラボでできないこと

VM から到達できるのは Ubuntu のパッケージアーカイブ、PyPI、npm、GitHub だけです。そのため、インターネット上のホストにラボを向けることはできません。VM は 1 台なので、「外部」は VM の中の名前空間であり、本物のインターネットではありません。

## このリポジトリについて

[Mitsuru Technologies](https://mitsurutech.co.uk/ja) が [MitsuruTech ラーニングプラットフォーム](https://mitsurutech.co.uk/ja/learning)のために作りました。学校向けに、Python、Java、Go、Node、SQL、Linux をブラウザーで学べます。コンテナーラボ、VM ラボ、GitHub からデプロイする Web アプリ、自動採点されるクイズも備えています。アカウントなしでも[プレイグラウンド](https://mitsurutech.co.uk/ja/playground)でコードを試せます。

同じ提供元のほかのラボ: [lab-netlab](https://github.com/mitsurutech/lab-netlab)（静的ルーティング、OSPF、BGP）、[lab-juice-shop](https://github.com/mitsurutech/lab-juice-shop)、[lab-dvwa](https://github.com/mitsurutech/lab-dvwa)、[lab-webgoat](https://github.com/mitsurutech/lab-webgoat)、[lab-mysql-phpmyadmin](https://github.com/mitsurutech/lab-mysql-phpmyadmin)、[lab-git-gitea](https://github.com/mitsurutech/lab-git-gitea)、[lab-sample](https://github.com/mitsurutech/lab-sample)。

## ライセンス

この作品は [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/) でライセンスされています。Mitsuru Technologies Ltd のクレジットを表示し、商用に使わず、これをもとに作ったものを同じライセンスで共有するなら、学習と授業のために共有・改変できます。全文は `LICENSE` ファイルにあります。

nftables と conntrack-tools は GPL-2.0 でライセンスされており、VM の起動時に Ubuntu のアーカイブからインストールされます。それらのコードはここでは配布していません。
