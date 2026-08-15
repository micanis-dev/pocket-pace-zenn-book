---
title: "開発環境の構築"
---

いよいよ開発を始めます。まずはWebアプリを作るための道具をそろえます。今回はDevboxでプロジェクト専用の環境を作り、Node.js 24とpnpmを入れました。さらにdirenvも使って、ディレクトリへ移動すると環境が自動で切り替わるようにします。

動作確認にはmacOSを使用しています。WindowsではWSL2上で同様に進められるはずですが、この本では検証していません。Linuxについても、ディストリビューションやシェルによって一部の手順が異なる可能性があります。

## 使用するツール

最初は似た名前が一度に出てきて混乱したので、作業の前にそれぞれの役割を自分なりに整理しておきます。

### Devbox

Devboxは、プロジェクトごとに開発環境を管理するツールです。必要なツールとそのバージョンを`devbox.json`に記録し、Nixを使って他のプロジェクトから分けた環境を作ります。

Devboxは内部でNixを使用していますが、今回の環境を作るだけならNix言語の知識は必要ありません。Nixから理解したい場合は[小さく始めるNix](https://zenn.dev/trifolium/books/1c0373f3570334)、Devboxを中心に知りたい場合は[Devboxについての紹介記事](https://zenn.dev/arkbig/articles/devbox_0b6b39cd097e3288fe58baa5a49c7d39a28c5b46849)が参考になります。

### direnv

direnvは、ディレクトリごとに環境変数を切り替えるためのツールです。今回はDevboxと連携し、プロジェクトのディレクトリへ入ったときにDevbox環境を自動で有効にするために使います。direnvは必須ではなく、使わない場合は開発を始めるたびに`devbox shell`を実行します。

### Node.js

Node.jsは、JavaScriptをWebブラウザの外でも実行できるようにする実行環境です。今回はReact Routerの開発サーバーやビルドツールを動かすために使用します。

Node.jsには複数のバージョンがあります。このプロジェクトで使用するのは、Node.jsのメジャーバージョン24です。以降のコマンドに出てくる`nodejs@24`は、「Node.jsというパッケージのバージョン24を使う」という指定です。

### pnpm

Webアプリケーションの開発では、Reactなどの外部パッケージを利用します。pnpmは、`package.json`に書かれたパッケージをインストールしたり、開発サーバーの起動やビルドのコマンドを実行したりするパッケージマネージャーです。

インストールしたパッケージの正確なバージョンは`pnpm-lock.yaml`に記録されます。このファイルもGitで管理することで、別の環境でも同じ依存関係を再現しやすくなります。

## 1. プロジェクト用のディレクトリを作る

最初に、開発用のディレクトリを作ります。私はホームディレクトリの下に`Develop`を用意し、プロジェクトごとのディレクトリをその中へ置いています。

```bash:ターミナル
mkdir -p ~/Develop/pocket-pace-noai
cd ~/Develop/pocket-pace-noai
```

`pocket-pace-noai`という名前には、財布のペースを管理するアプリケーションであることと、生成AIへコーディングを任せずに開発するという意味を込めています。ディレクトリ名は任意ですが、以降の例ではこの名前を使用します。

現在位置は`pwd`で確認できます。

```bash:ターミナル
pwd
```

末尾が`Develop/pocket-pace-noai`になっていれば準備完了です。以降、特に説明がない限り、このディレクトリをプロジェクトルートとして扱います。

## 2. Devboxをインストールする

[Devbox公式のインストール手順](https://www.jetify.com/docs/devbox/installing-devbox/)に従い、インストーラーを実行します。

```bash:ターミナル
curl -fsSL https://get.jetify.com/devbox | bash
```

Devboxが必要とするNixが見つからない場合は、Nixも同時にインストールされます。処理が完了したらターミナルを開き直し、バージョンを確認します。

```bash:ターミナル
devbox version
```

バージョン番号が表示されればインストールできています。`command not found`になる場合は、まずターミナルを開き直し、それでも解決しなければインストール時に表示されたPATHの案内を確認します。

## 3. Devboxプロジェクトを初期化する

プロジェクトルートで次のコマンドを実行します。

```bash:ターミナル
devbox init
```

実行後、`devbox.json`が作成されます。このファイルに、プロジェクトで使用するパッケージや環境変数などを定義します。

```bash:ターミナル
ls
```

一覧に`devbox.json`があれば初期化は完了です。`devbox.json`は開発環境を再現するために必要なので、Gitの管理対象に含めます。

## 4. Node.js 24とpnpmを追加する

Node.js 24とpnpmをDevboxのパッケージとして追加します。

```bash:ターミナル
devbox add nodejs@24 pnpm
```

このコマンドでは、`devbox add`に続けて、追加したい二つのパッケージを指定しています。`nodejs@24`の`nodejs`がNode.jsのパッケージ名、`@24`が使用するメジャーバージョンです。`pnpm`にはバージョンを指定せず、Devboxが利用可能なバージョンを解決するようにしています。解決されたバージョンは`devbox.lock`に記録されます。

初回はNixのパッケージ情報や各ツールを取得するため、完了まで少し時間がかかる場合があります。実行後の`devbox.json`には、次のようにパッケージが追加されます。

```json:devbox.json
{
  "packages": ["nodejs@24", "pnpm"]
}
```

実際の`devbox.json`には、スキーマやシェル設定など、ほかの項目が含まれることもあります。また、解決したパッケージ情報を記録する`devbox.lock`も作成されます。`devbox.json`と`devbox.lock`の両方をGitで管理します。

Devbox環境へ入ります。

```bash:ターミナル
devbox shell
```

その状態でNode.jsとpnpmを確認します。

```bash:ターミナル
node --version
pnpm --version
which node
which pnpm
```

Node.jsのバージョンが`v24`から始まり、`which`の結果に`.devbox`やNixに関係するパスが含まれていれば、Devbox内のツールを参照できています。Devbox環境から出る場合は`exit`を実行します。

## 5. direnvを導入する

ここまでの状態でも開発できますが、ターミナルを開くたびに`devbox shell`を実行する必要があります。そこでdirenvを使い、プロジェクトルートへ移動したときにDevbox環境が自動で読み込まれるようにします。

[direnv公式のインストール手順](https://direnv.net/docs/installation.html)では、パッケージマネージャーを使う方法と、インストールスクリプトを使う方法が案内されています。ここではスクリプトを使用します。

```bash:ターミナル
curl -sfL https://direnv.net/install.sh | bash
```

ターミナルを開き直し、インストールを確認します。

```bash:ターミナル
direnv version
```

direnvは、インストールしただけではディレクトリの移動を検知できません。使用しているシェルに応じたフックを設定する必要があります。設定方法はシェルごとに異なるため、[direnv公式のシェル別セットアップ](https://direnv.net/docs/hook.html)から、自分が使用しているシェルの手順を選びます。

zsh、Bash、Fish、PowerShellなど、自分が使用しているシェルに対応する設定を選んでください。この本では特定のシェルを前提にせず、フック設定の詳細は公式ドキュメントに任せます。

フックを追加したあとは、ターミナルを開き直して設定を反映します。

## 6. Devboxとdirenvを連携する

プロジェクトルートへ戻り、Devbox用の`.envrc`を生成します。

```bash:ターミナル
cd ~/Develop/pocket-pace-noai
devbox generate direnv
```

`.envrc`には、現在のシェルへDevbox環境を読み込む処理が記述されます。direnvは安全のため、未確認の`.envrc`を自動では実行しません。内容を確認してから許可します。

```bash:ターミナル
direnv allow
```

`devbox generate direnv`の実行時に自動で許可され、改めて`direnv allow`を実行する必要がない場合もあります。

動作を確認するため、一度プロジェクトの外へ移動してから戻ります。

```bash:ターミナル
cd ..
cd pocket-pace-noai
```

移動時に`direnv: loading`や`direnv: using devbox`のようなメッセージが表示されれば、`.envrc`が読み込まれています。`devbox shell`を実行せずに、Node.jsとpnpmが利用できるか確認します。

```bash:ターミナル
node --version
pnpm --version
which node
which pnpm
```

Devbox内のNode.jsとpnpmが表示されれば連携は完了です。

`devbox.json`や`.envrc`を変更すると、direnvが環境の読み込みを停止することがあります。その場合は変更内容を確認し、もう一度`direnv allow`を実行します。

## Gitで管理するファイル

この章で作成した`devbox.json`、`devbox.lock`、`.envrc`は、環境を再現するためにGitで管理します。一方、`.devbox`は設定ファイルをもとに再生成できるため、Gitへ登録しません。`.gitignore`に次の内容が含まれていることを確認します。

```gitignore:.gitignore
.devbox/
```

## Devboxやdirenvを使わない場合

Devboxやdirenvを使わない場合は、[Node.js公式サイト](https://nodejs.org/ja/download)と[pnpm公式のインストール手順](https://pnpm.io/ja/installation)に従って、それぞれをOSへインストールします。Node.jsはメジャーバージョン24を選びます。インストール後に`node --version`と`pnpm --version`を確認し、次の章から同じように進めます。

OSへ直接インストールする方法では、プロジェクトごとのバージョン差を管理する工夫が別途必要です。この本では、開発ツールとそのバージョンをプロジェクトの設定に残すため、Devboxを採用しました。

## よくある問題

### コマンドが見つからない

Devboxやdirenvをインストールした直後であれば、ターミナルを開き直します。Node.jsやpnpmだけが見つからない場合は、プロジェクトルートへ移動し、direnvが読み込まれているか確認します。direnvを使っていなければ、`devbox shell`へ入ってから実行します。

### `.envrc is blocked`と表示される

direnvが未許可の`.envrc`を止めています。内容を確認し、問題がなければプロジェクトルートで`direnv allow`を実行します。

### どの環境のNode.jsとpnpmを使っているか分からない

`which node`と`which pnpm`で実行ファイルの場所を確認します。Devboxを使っている場合は、`.devbox`またはNixに関係するパスが表示されます。あわせて`node --version`と`pnpm --version`を記録しておくと、環境差による問題を切り分けやすくなります。

## まとめ

Devbox、Nix、direnvの関係が最初はよく分かりませんでしたが、コマンドを動かすうちに少しずつ役割が見えてきました。DevboxでNode.js 24とpnpmを用意し、direnvのおかげでプロジェクトへ移動するだけで使える状態になりました。

バージョン番号と`which`の結果も確認できたので、ひとまず環境構築は完了とします。次は、この環境でReact Routerのプロジェクトを作ります。
