---
title: "開発環境の構築"
---

この章では、Webアプリケーションの開発に必要な環境を構築します。Devboxでプロジェクトごとの環境を用意し、Node.js 24とpnpmを導入します。また、direnvを使い、プロジェクトのディレクトリへ移動したときに環境が自動で切り替わるようにします。最後にReact Router v7のプロジェクトを作成し、ブラウザで初期画面を表示できるところまで確認します。

動作確認にはmacOSを使用しています。WindowsではWSL2上で同様に進められるはずですが、この本では検証していません。Linuxについても、ディストリビューションやシェルによって一部の手順が異なる可能性があります。

## 使用するツール

この本の開発環境で使用するDevbox、direnv、Node.js、pnpm、React Routerについて、それぞれの役割を簡単に紹介します。

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

### React Router

React Routerは、Reactで作った画面とURLを結び付け、画面遷移を管理するためのツールです。ダッシュボード、支出管理、収入管理、履歴管理などの画面を持つこのアプリケーションでは、どのURLでどの画面を表示するかを管理する必要があります。

これをReactだけで実装すると、URLの変更、ブラウザの戻る・進む操作、URLへの直接アクセスなどを自分で考慮しなければなりません。こうしたルーティングの実装にかかる手間を減らすため、このプロジェクトではReact Routerを採用します。

## 1. プロジェクト用のディレクトリを作る

最初に、開発用のディレクトリを作ります。私はホームディレクトリの下に`Develop`を用意し、プロジェクトごとのディレクトリをその中へ置いています。

```bash
mkdir -p ~/Develop/pocket-pace-noai
cd ~/Develop/pocket-pace-noai
```

`pocket-pace-noai`という名前には、財布のペースを管理するアプリケーションであることと、生成AIへコーディングを任せずに開発するという意味を込めています。ディレクトリ名は任意ですが、以降の例ではこの名前を使用します。

現在位置は`pwd`で確認できます。

```bash
pwd
```

末尾が`Develop/pocket-pace-noai`になっていれば準備完了です。以降、特に説明がない限り、このディレクトリをプロジェクトルートとして扱います。

## 2. Devboxをインストールする

[Devbox公式のインストール手順](https://www.jetify.com/docs/devbox/installing-devbox/)に従い、インストーラーを実行します。

```bash
curl -fsSL https://get.jetify.com/devbox | bash
```

Devboxが必要とするNixが見つからない場合は、Nixも同時にインストールされます。処理が完了したらターミナルを開き直し、バージョンを確認します。

```bash
devbox version
```

バージョン番号が表示されればインストールできています。`command not found`になる場合は、まずターミナルを開き直し、それでも解決しなければインストール時に表示されたPATHの案内を確認します。

## 3. Devboxプロジェクトを初期化する

プロジェクトルートで次のコマンドを実行します。

```bash
devbox init
```

実行後、`devbox.json`が作成されます。このファイルに、プロジェクトで使用するパッケージや環境変数などを定義します。

```bash
ls
```

一覧に`devbox.json`があれば初期化は完了です。`devbox.json`は開発環境を再現するために必要なので、Gitの管理対象に含めます。

## 4. Node.js 24とpnpmを追加する

Node.js 24とpnpmをDevboxのパッケージとして追加します。

```bash
devbox add nodejs@24 pnpm
```

このコマンドでは、`devbox add`に続けて、追加したい二つのパッケージを指定しています。`nodejs@24`の`nodejs`がNode.jsのパッケージ名、`@24`が使用するメジャーバージョンです。`pnpm`にはバージョンを指定せず、Devboxが利用可能なバージョンを解決するようにしています。解決されたバージョンは`devbox.lock`に記録されます。

初回はNixのパッケージ情報や各ツールを取得するため、完了まで少し時間がかかる場合があります。実行後の`devbox.json`には、次のようにパッケージが追加されます。

```json
{
  "packages": ["nodejs@24", "pnpm"]
}
```

実際の`devbox.json`には、スキーマやシェル設定など、ほかの項目が含まれることもあります。また、解決したパッケージ情報を記録する`devbox.lock`も作成されます。`devbox.json`と`devbox.lock`の両方をGitで管理します。

Devbox環境へ入ります。

```bash
devbox shell
```

その状態でNode.jsとpnpmを確認します。

```bash
node --version
pnpm --version
which node
which pnpm
```

Node.jsのバージョンが`v24`から始まり、`which`の結果に`.devbox`やNixに関係するパスが含まれていれば、Devbox内のツールを参照できています。Devbox環境から出る場合は`exit`を実行します。

## 5. direnvを導入する

ここまでの状態でも開発できますが、ターミナルを開くたびに`devbox shell`を実行する必要があります。そこでdirenvを使い、プロジェクトルートへ移動したときにDevbox環境が自動で読み込まれるようにします。

[direnv公式のインストール手順](https://direnv.net/docs/installation.html)では、パッケージマネージャーを使う方法と、インストールスクリプトを使う方法が案内されています。ここではスクリプトを使用します。

```bash
curl -sfL https://direnv.net/install.sh | bash
```

ターミナルを開き直し、インストールを確認します。

```bash
direnv version
```

direnvは、インストールしただけではディレクトリの移動を検知できません。使用しているシェルに応じたフックを設定する必要があります。設定方法はシェルごとに異なるため、[direnv公式のシェル別セットアップ](https://direnv.net/docs/hook.html)から、自分が使用しているシェルの手順を選びます。

zsh、Bash、Fish、PowerShellなど、自分が使用しているシェルに対応する設定を選んでください。この本では特定のシェルを前提にせず、フック設定の詳細は公式ドキュメントに任せます。

フックを追加したあとは、ターミナルを開き直して設定を反映します。

## 6. Devboxとdirenvを連携する

プロジェクトルートへ戻り、Devbox用の`.envrc`を生成します。

```bash
cd ~/Develop/pocket-pace-noai
devbox generate direnv
```

`.envrc`には、現在のシェルへDevbox環境を読み込む処理が記述されます。direnvは安全のため、未確認の`.envrc`を自動では実行しません。内容を確認してから許可します。

```bash
direnv allow
```

`devbox generate direnv`の実行時に自動で許可され、改めて`direnv allow`を実行する必要がない場合もあります。

動作を確認するため、一度プロジェクトの外へ移動してから戻ります。

```bash
cd ..
cd pocket-pace-noai
```

移動時に`direnv: loading`や`direnv: using devbox`のようなメッセージが表示されれば、`.envrc`が読み込まれています。`devbox shell`を実行せずに、Node.jsとpnpmが利用できるか確認します。

```bash
node --version
pnpm --version
which node
which pnpm
```

Devbox内のNode.jsとpnpmが表示されれば連携は完了です。

`devbox.json`や`.envrc`を変更すると、direnvが環境の読み込みを停止することがあります。その場合は変更内容を確認し、もう一度`direnv allow`を実行します。

## 7. React Routerプロジェクトを作成する

上で説明したルーティングの管理をReact Routerに任せるため、React Router公式のプロジェクト作成ツールを使ってひな形を作ります。プロジェクトルートで次のコマンドを実行します。この本ではReact Router v7を使用するため、パッケージのメジャーバージョンを`@7`で固定します。

```bash
pnpm dlx create-react-router@7
```

`@latest`を指定するとReact Router v8の作成ツールが実行されるため、ここでは使用しません。

初回実行時は、`create-react-router`とその依存関係がダウンロードされます。今回の動作確認では、`create-react-router v7.18.2`が実行されました。`@7`はメジャーバージョンだけを固定するため、実行する時期によってv7系のより新しいバージョンが表示される場合があります。

`deprecated subdependencies found`という警告が表示されることがあります。これは作成ツールが間接的に使っているパッケージに関する警告です。その後も処理が続き、最後に`That's it!`と表示されれば、プロジェクトの作成は完了しています。

質問が表示されたら、次のように回答します。

- `Where should we create your new project?`: `./frontend`
- `Initialize a new git repository?`: `Yes`
- `Install dependencies with pnpm?`: `Yes`

テンプレートを別途指定していないため、デフォルトテンプレートが使用されます。作成先として`./frontend`を指定すると、プロジェクトルートの下に`frontend`ディレクトリが作成されます。その中でGitリポジトリが初期化され、pnpmによる依存関係のインストールも行われます。

処理が完了したら、表示された案内に従って作成されたディレクトリへ移動します。

```bash
cd frontend
```

初期化後は、`package.json`、`app`ディレクトリ、`react-router.config.ts`、TypeScriptの設定ファイルなどが作成されます。

```bash
ls
```

`package.json`と`app`が存在していれば、React Routerプロジェクトのひな形は作成されています。

すでに`frontend`ディレクトリが存在し、ファイルが入っている場合は、そこで初期化をやり直さないようにします。既存ファイルがある場合は、内容を確認したうえで必要な依存関係だけを追加します。

## 8. 開発サーバーを起動する

`frontend`ディレクトリで開発サーバーを起動します。

```bash
pnpm dev
```

ターミナルに`http://localhost:...`という形式のURLが表示されます。そのURLをブラウザで開き、Reactの初期画面が表示されれば成功です。ポート番号は環境やテンプレートのバージョンによって異なる可能性があるため、実際に表示されたURLを使用します。

![React Routerの初期画面](/images/7464cf3f3b7b86/react-router-initial-screen.png =600x)

開発サーバーを終了する場合は、起動したターミナルでControl+Cを押します。

続いて、本番用のビルドが通ることも確認します。

```bash
pnpm build
```

エラーが表示されずに完了すれば、開発を始めるための最低限の環境は整っています。

## Gitで管理するファイル

環境構築によって作成されるファイルのうち、`devbox.json`、`devbox.lock`、`.envrc`、`package.json`、`pnpm-lock.yaml`は基本的にGitで管理します。これらには、環境や依存関係を再現するための情報が記録されています。

一方、`.devbox`と`node_modules`は、設定ファイルをもとに再生成できるためGitへ登録しません。`.gitignore`に次の内容が含まれていることを確認します。

```gitignore
.devbox/
node_modules/
```

今後、データベースの接続情報やAPIキーを`.env`へ保存する場合も、秘密情報を含むファイルはGitへ登録しないようにします。コミット前に`git status`を確認する習慣を付けておくと、意図しないファイルの公開を防げます。

## Devboxやdirenvを使わない場合

Devboxやdirenvを使わない場合は、[Node.js公式サイト](https://nodejs.org/ja/download)と[pnpm公式のインストール手順](https://pnpm.io/ja/installation)に従って、それぞれをOSへインストールします。Node.jsはメジャーバージョン24を選びます。インストール後に`node --version`と`pnpm --version`を確認し、React Routerプロジェクトを作る手順から同じように進めます。

OSへ直接インストールする方法では、プロジェクトごとのバージョン差を管理する工夫が別途必要です。この本では、開発ツールとそのバージョンをプロジェクトの設定に残すため、Devboxを採用しました。

## よくある問題

### コマンドが見つからない

Devboxやdirenvをインストールした直後であれば、ターミナルを開き直します。Node.jsやpnpmだけが見つからない場合は、プロジェクトルートへ移動し、direnvが読み込まれているか確認します。direnvを使っていなければ、`devbox shell`へ入ってから実行します。

### `.envrc is blocked`と表示される

direnvが未許可の`.envrc`を止めています。内容を確認し、問題がなければプロジェクトルートで`direnv allow`を実行します。

### 開発サーバーを開けない

ブラウザが自動で起動しなくても、ターミナルに表示されたURLを直接開けば確認できます。ポートが使用中の場合は、別のターミナルで以前の開発サーバーが動いていないか確認します。

### どの環境のNode.jsとpnpmを使っているか分からない

`which node`と`which pnpm`で実行ファイルの場所を確認します。Devboxを使っている場合は、`.devbox`またはNixに関係するパスが表示されます。あわせて`node --version`と`pnpm --version`を記録しておくと、環境差による問題を切り分けやすくなります。

## Reactライブラリについて

Reactには、画面遷移、フォーム、データ取得、状態管理、テストなどを支援する多くのライブラリがあります。ただし、最初からまとめて導入すると、それぞれが何を解決しているのか分かりにくくなります。

このプロジェクトでは、必要な機能が明確になった段階でライブラリを追加します。導入時には、そのライブラリが解決する問題、Reactの標準機能だけでは不足する理由、保守状況などを確認し、採用理由とともに記録します。

## まとめ

この章では、Devboxでプロジェクト専用の環境を作り、Node.js 24とpnpmを導入しました。さらにdirenvと連携し、プロジェクトルートへ移動するだけで環境が有効になるようにしました。最後にpnpmでReact Routerプロジェクトを初期化し、開発サーバーの起動とビルドを確認しました。

これで、前章までに設計したUIをReactで実装する準備が整いました。
