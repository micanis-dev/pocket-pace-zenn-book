---
title: "React Routerプロジェクトの作成"
---

前章でNode.js 24とpnpmを使えるようになったので、今度はReact Router v7のひな形を作ります。Tailwind CSSとshadcn/uiも追加し、まずは開発用の画面が開くところと、ビルドが通るところまで進めます。知らない道具が一気に増えるので、何のために入れるのかも確認しながら作業しました。

## 使用するツール

コマンドをそのまま実行するだけでは後で分からなくなりそうなので、今回追加する三つの役割を先に整理します。

### React Router

React Routerは、Reactで作った画面とURLを結び付け、画面遷移を管理するためのツールです。ダッシュボード、支出管理、収入管理、履歴管理などの画面を持つこのアプリケーションでは、どのURLでどの画面を表示するかを管理する必要があります。

これをReactだけで実装すると、URLの変更、ブラウザの戻る・進む操作、URLへの直接アクセスなどを自分で考慮しなければなりません。こうしたルーティングの実装にかかる手間を減らすため、このプロジェクトではReact Routerを採用します。

### Tailwind CSS

Tailwind CSSは、`flex`、`p-4`、`text-lg`などの小さなユーティリティクラスをHTMLやJSXに組み合わせて、レイアウトや色、文字サイズなどを指定するCSSフレームワークです。このプロジェクトでは、Figmaで作成したUIをReactのコンポーネントへ落とし込むために使用します。

### shadcn/ui

shadcn/uiは、ボタン、入力欄、ダイアログ、カレンダーなどのUIコンポーネントをプロジェクトへ追加するための仕組みです。一般的なコンポーネントライブラリのように、パッケージ内の完成品をそのまま呼び出すのではなく、選んだコンポーネントのソースコードをプロジェクトへ追加します。

追加されたコードは自分のプロジェクト内で管理されるため、内容を確認したり、Figmaで作成したデザインに合わせて変更したりできます。このプロジェクトでは最初からすべてのコンポーネントを追加せず、必要になった段階で一つずつ導入します。

## 1. React Routerプロジェクトを作成する

React Router公式のプロジェクト作成ツールを使ってひな形を作ります。前章で用意したプロジェクトルートで次のコマンドを実行します。この本ではReact Router v7を使用するため、パッケージのメジャーバージョンを`@7`で固定します。

```bash:ターミナル
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

```bash:ターミナル
cd frontend
```

初期化後は、`package.json`、`app`ディレクトリ、`react-router.config.ts`、TypeScriptの設定ファイルなどが作成されます。

```bash:ターミナル
ls
```

`package.json`と`app`が存在していれば、React Routerプロジェクトのひな形は作成されています。

すでに`frontend`ディレクトリが存在し、ファイルが入っている場合は、そこで初期化をやり直さないようにします。既存ファイルがある場合は、内容を確認したうえで必要な依存関係だけを追加します。

## 2. Tailwind CSSを追加する

Tailwind CSSと、Tailwind CSSをViteで使用するためのプラグインを追加します。`frontend`ディレクトリで次のコマンドを実行します。

```bash:ターミナル
pnpm add tailwindcss @tailwindcss/vite
```

`tailwindcss`がTailwind CSS本体、`@tailwindcss/vite`がViteと連携するためのプラグインです。React Routerのデフォルトテンプレートにすでに含まれている場合は、必要なパッケージがインストール済みであることが表示されます。

続いて、`vite.config.ts`にTailwind CSSのViteプラグインが設定されているか確認します。次のインポートと`tailwindcss()`がすでにある場合は、変更する必要はありません。なければ、既存の設定を残したまま追加します。

```diff ts:vite.config.ts
 import { reactRouter } from "@react-router/dev/vite";
+import tailwindcss from "@tailwindcss/vite";
 import { defineConfig } from "vite";

 export default defineConfig({
-  plugins: [reactRouter()],
+  plugins: [tailwindcss(), reactRouter()],
 });
```

`plugins`にすでにほかのプラグインが設定されている場合は、それらを削除せず、配列に`tailwindcss()`を追加します。

最後に、グローバルなスタイルを定義する`app/app.css`の先頭に次の一行があるか確認します。なければ追加します。

```diff css:app/app.css
+@import "tailwindcss";
```

`app/root.tsx`から`app.css`が読み込まれることで、各画面でTailwind CSSのユーティリティクラスを使用できるようになります。

## 3. shadcn/uiを初期化する

Tailwind CSSの準備ができたら、shadcn/uiを初期化します。今回はshadcn/createで次の構成を選びました。

- コンポーネントライブラリ: Base UI
- スタイル: Vega
- ベースカラー: Neutral
- テーマカラー: Neutral
- チャートカラー: Neutral
- アイコン: Lucide
- フォント: Noto Sans
- 見出しのフォント: 本文と同じ
- 角丸: Small
- メニューのアクセント: Subtle
- メニューカラー: Default

選択した内容はプリセットコード`bIkez4q`としてコマンドへ含めます。プリセットコードだけでは設定内容を読み取れないため、この本では上の一覧も記録しておきます。

コードに含まれる設定は、次のコマンドでも確認できます。

```bash:ターミナル
pnpm dlx shadcn@latest preset decode bIkez4q
```

shadcn/uiを初期化するため、`frontend`ディレクトリで次のコマンドを実行します。

```bash:ターミナル
pnpm dlx shadcn@latest init --preset bIkez4q --template react-router
```

`--template react-router`はReact Router向けの構成を使用する指定です。プリセットには、Base UI、Vega、Neutralのほか、アイコンや角丸などの設定も含まれています。Base UIを選ぶと、shadcn/uiが追加するコンポーネントの内部でBase UIのプリミティブが使われます。Vegaの見た目とテーマを土台として、最終的にはFigmaのデザインに合わせて調整します。

選択後は、React Router、Tailwind CSS、インポートエイリアスなどの確認と、必要なパッケージのインストールが自動で行われます。処理が完了すると、プロジェクトルートに`components.json`が作成されます。このファイルには、コンポーネントの追加先、プリセット、インポートに使用するエイリアス、Tailwind CSSの設定などが記録されます。

今回の初期化では、あわせて次のファイルが作成されました。

- `app/components/ui/button.tsx`: Buttonコンポーネント
- `app/lib/utils.ts`: クラス名を組み立てるための共通関数

また、テーマやshadcn/uiで使用するスタイルを追加するため、`app/app.css`も更新されます。

```bash:ターミナル
ls components.json app/components/ui/button.tsx app/lib/utils.ts
```

3ファイルが表示されれば初期化は完了です。Buttonは初期化時に作成されましたが、この章ではまだ画面へ配置しません。入力欄やカレンダーなど、ほかのコンポーネントは、その機能を実装する章で必要になったときに追加します。

## 4. 開発サーバーを起動する

`frontend`ディレクトリで開発サーバーを起動します。

```bash:ターミナル
pnpm dev
```

ターミナルに`http://localhost:...`という形式のURLが表示されます。そのURLをブラウザで開き、Reactの初期画面が表示されれば成功です。ポート番号は環境やテンプレートのバージョンによって異なる可能性があるため、実際に表示されたURLを使用します。

![React Routerの初期画面](/images/7464cf3f3b7b86/react-router-initial-screen.png =600x)

開発サーバーを終了する場合は、起動したターミナルでControl+Cを押します。

続いて、本番用のビルドが通ることも確認します。

```bash:ターミナル
pnpm build
```

エラーが表示されずに完了すれば、開発を始めるための最低限の準備は整っています。

## Gitで管理するファイル

React Routerプロジェクトの作成によって追加された`package.json`と`pnpm-lock.yaml`に加えて、shadcn/uiの設定を記録する`components.json`、生成された`app/components/ui/button.tsx`と`app/lib/utils.ts`、更新された`app/app.css`もGitで管理します。これらには、利用するパッケージや正確なバージョン、コンポーネントの設定と実装が記録されています。

一方、`node_modules`は`package.json`と`pnpm-lock.yaml`をもとに再生成できるため、Gitへ登録しません。`.gitignore`に次の内容が含まれていることを確認します。

```gitignore:.gitignore
node_modules/
```

今後、データベースの接続情報やAPIキーを`.env`へ保存する場合も、秘密情報を含むファイルはGitへ登録しないようにします。コミット前に`git status`を確認する習慣を付けておくと、意図しないファイルの公開を防げます。

## よくある問題

### 開発サーバーを開けない

ブラウザが自動で起動しなくても、ターミナルに表示されたURLを直接開けば確認できます。ポートが使用中の場合は、別のターミナルで以前の開発サーバーが動いていないか確認します。

### Tailwind CSSが反映されない

`vite.config.ts`の`plugins`に`tailwindcss()`が含まれているか、`app/app.css`に`@import "tailwindcss";`があるかを確認します。また、`app/root.tsx`から`app.css`が読み込まれていることも確認します。

### shadcn/uiを初期化できない

`frontend`ディレクトリでコマンドを実行しているか確認します。あわせて、`package.json`、Tailwind CSSの設定、React Routerのデフォルトで用意される`~/*`のインポートエイリアスが存在するかも確認します。

## Reactライブラリについて

Reactには、画面遷移、フォーム、データ取得、状態管理、テストなどを支援する多くのライブラリがあります。ただし、最初からまとめて導入すると、それぞれが何を解決しているのか分かりにくくなります。

今の私には、たくさん入れても使い分けられそうにありません。このプロジェクトでは、困りごとがはっきりした時点で必要なライブラリを追加します。そのときに「何が楽になるのか」を調べ、採用した理由も記録するつもりです。

## まとめ

React Router v7のプロジェクトを作り、Tailwind CSSとshadcn/uiも追加できました。ひな形の作成中にGitリポジトリが入れ子になる点など、コマンドを実行するだけでは気づきにくいところもありましたが、開発サーバーと本番用ビルドの両方を確認できました。

まだ自分で画面はほとんど書いていませんが、これでようやく土台ができました。次は、画面とURLを結び付けるルーティングに進みます。
