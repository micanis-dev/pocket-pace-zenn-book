---
title: "ルーティング"
---

前章では、React Routerプロジェクトを作成し、開発サーバーの起動とビルドを確認しました。ここからは、これまでに設計した画面の実装へ進みます。

最初に取り組むのはルーティングです。Webアプリケーションでダッシュボードや支出管理などのページを移動するときは、表示する内容に対応したURLへ切り替える必要があります。URLとページの対応を自分で一から管理することもできますが、ブラウザの戻る・進む操作やURLへの直接アクセスも考慮しなければなりません。

React Routerは、こうしたページとURLの対応やページ遷移を管理するためのライブラリです。この章では、ダッシュボード、支出管理、収入管理、履歴管理の4ページを用意し、すべてのページへ移動できるところまで確認します。まだ画面の見た目は作り込まず、ルーティングが正しく動くことだけを目標にします。

## ファイルベースルーティングを導入する

今回は、ファイル名をもとにURLを決めるファイルベースルーティングを使用します。そのために、`@react-router/fs-routes`を追加します。

以降のコマンドは、前章で作成したReact Routerプロジェクトの`frontend`ディレクトリで実行します。現在位置がプロジェクトルートの場合は、次のコマンドで移動します。

```bash:ターミナル
cd ~/Develop/pocket-pace-noai/frontend
```

`package.json`があることを確認します。

```bash:ターミナル
ls package.json
```

ファイルが表示されたら、pnpmで`@react-router/fs-routes`を追加します。

```bash:ターミナル
pnpm add @react-router/fs-routes
```

React Routerの公式ドキュメントではnpmを使ったコマンドが掲載されていますが、このプロジェクトでは前章でpnpmを採用したため、ここでもpnpmを使用します。

## 4ページ分のファイルを作る

ファイルベースルーティングでは、`app/routes`ディレクトリ内のファイルがページとして扱われます。今回必要な4ページのファイルを作成します。

```bash:ターミナル
touch app/routes/dashboard.tsx \
  app/routes/expense.tsx \
  app/routes/income.tsx \
  app/routes/history.tsx
```

CLIを使わず、エディターやファイルマネージャーから一つずつ作成しても問題ありません。作成後の主な構成は次のようになります。

```text:ディレクトリ構成
frontend/
└── app/
    ├── routes.ts
    └── routes/
        ├── dashboard.tsx
        ├── expense.tsx
        ├── history.tsx
        └── income.tsx
```

`flatRoutes`では、ファイル名がそのままURLのパスになります。たとえば、`dashboard.tsx`は`/dashboard`、`expense.tsx`は`/expense`に対応します。

## `routes.ts`を編集する

次に、`app/routes.ts`を編集し、ファイルベースルーティングを有効にします。既存の内容を次のコードへ置き換えます。

```tsx:app/routes.ts
import { type RouteConfig } from "@react-router/dev/routes";
import { flatRoutes } from "@react-router/fs-routes";

export default flatRoutes() satisfies RouteConfig;
```

`RouteConfig`は、ルート設定の型を確認するために使用します。`flatRoutes()`は`app/routes`内のファイルを読み取り、ファイル名に対応するルート設定を作成します。

この設定により、4つのファイルとURLは次のように対応します。

| ファイル | URL |
| --- | --- |
| `app/routes/dashboard.tsx` | `/dashboard` |
| `app/routes/expense.tsx` | `/expense` |
| `app/routes/income.tsx` | `/income` |
| `app/routes/history.tsx` | `/history` |

## ページ遷移用のリンクを作る

4つのページを移動できるように、それぞれのファイルへナビゲーションを追加します。今回はルーティングの確認が目的なので、4ファイルとも同じ内容にします。

`app/routes/dashboard.tsx`、`app/routes/expense.tsx`、`app/routes/income.tsx`、`app/routes/history.tsx`へ、次のコードを記述します。

```tsx:app/routes/*.tsx
import { NavLink } from "react-router";

export default function Page() {
  return (
    <nav>
      <NavLink to="/dashboard" end>
        Home
      </NavLink>
      <NavLink to="/expense" end>
        Expense
      </NavLink>
      <NavLink to="/income" end>
        Income
      </NavLink>
      <NavLink to="/history" end>
        History
      </NavLink>
    </nav>
  );
}
```

`NavLink`は、React Routerが提供するページ遷移用のコンポーネントです。通常の`a`要素と同じようにリンクを表示できますが、リンクを選択したときにページ全体を読み込み直さず、React Routerの管理下でURLと表示内容を切り替えられます。

`to`には移動先のURLを指定します。今回はファイル名と対応する`/dashboard`、`/expense`、`/income`、`/history`を設定しました。URLは大文字と小文字を区別して扱われる可能性があるため、`/history`を含め、すべて小文字で統一します。

`end`は、現在のURLが`to`に指定したURLと最後まで一致するときだけ、そのリンクを選択中として扱うための指定です。今回のような単純なURLでは大きな違いはありませんが、あとで階層を持つURLを追加したときに選択状態を判定しやすくなります。

## 開発サーバーで確認する

ファイルを保存したら、`frontend`ディレクトリで開発サーバーを起動します。

```bash:ターミナル
pnpm dev
```

ターミナルに表示されたURLをブラウザで開き、末尾を`/dashboard`にします。ポート番号が5173の場合は、次のURLです。

```text:ブラウザで開くURL
http://localhost:5173/dashboard
```

ポート番号は環境によって変わることがあるため、5173以外の番号が表示された場合は、ターミナルに表示されたURLを使用します。

4つのリンクが表示されれば、まず`dashboard.tsx`がルートとして認識されています。

![ルーティングの動作確認](/images/7464cf3f3b7b86/routing.png =308x)

続いて、`Home`、`Expense`、`Income`、`History`を順番に選択します。ブラウザのURLが、それぞれ次のように変わることを確認します。

```text:確認するURL
/dashboard
/expense
/income
/history
```

今回はすべてのページへ同じ内容を書いているため、リンクを選んでも見た目は変わりません。URLが切り替わり、エラー画面が表示されなければページ遷移は成功です。ブラウザの戻る・進むボタンでも、直前に表示したURLへ移動できることを確認します。

開発サーバーを終了するときは、起動したターミナルでControl+Cを押します。

## うまく表示されない場合

### 404エラーになる

`app/routes`内のファイル名と、`NavLink`の`to`に指定したURLが一致しているか確認します。たとえば、`expenses.tsx`という複数形のファイルは`/expenses`に対応するため、`/expense`では表示されません。

また、ファイルを`frontend/routes`など別の場所へ作っていないかも確認します。今回の設定で読み込まれるのは、`frontend/app/routes`内のファイルです。

### パッケージが見つからない

`Cannot find package '@react-router/fs-routes'`のようなエラーが出た場合は、`frontend`ディレクトリで次のコマンドを実行したか確認します。

```bash:ターミナル
pnpm add @react-router/fs-routes
```

あわせて、`package.json`の`dependencies`に`@react-router/fs-routes`が追加されていることも確認します。

### URLは変わるが違いが分からない

現段階では4ページの内容が同じなので、これは想定した動作です。ブラウザのアドレス欄でURLが変わっていることを確認します。各画面の内容と見た目は、今後の章で実装します。

## まとめ

この章では、`@react-router/fs-routes`を導入し、ファイル名とURLを対応させるファイルベースルーティングを設定しました。さらに、ダッシュボード、支出管理、収入管理、履歴管理の4ページを作成し、`NavLink`を使って各ページを移動できることを確認しました。

これで、前章までに設計した各画面を個別のURLで表示する準備ができました。次の章から、それぞれのページへ実際のUIを実装していきます。
