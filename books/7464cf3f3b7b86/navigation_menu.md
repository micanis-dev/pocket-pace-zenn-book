---
title: "共通ナビゲーションの設計"
---

前章では、ダッシュボード、支出管理、収入管理、履歴管理の4ページを作成し、React Routerでページ間を移動できるようにしました。ただし、現時点では各ページに同じリンクを直接記述しただけで、4章で設計した画面下部のナビゲーションにはなっていません。

この章では、前章で作ったルーティングをナビゲーションメニューへ仕立てます。ナビゲーションメニューはすべてのページで共通して使用するため、独立したコンポーネントとして実装します。

## デザインをコンポーネントへ分ける

Figmaで作ったデザインをそのまま一つの大きなコンポーネントへ変換するのではなく、役割や再利用のしやすさを考え、ある程度の粒度へ分ける必要があります。今回は、4章で作成した画面下部のナビゲーションを次のように分けて考えました。

![フッターナビゲーションのコンポーネント分割](/images/7464cf3f3b7b86/footer-nav.png =500x)

### 緑色の枠

緑色の枠は、ナビゲーション全体を包む最も外側のコンポーネントです。今回は`FooterNavMenu`という名前にします。画面下部への配置、メニュー全体の幅や余白、内側にあるグループの並び方などを担当します。

### 赤色の枠

赤色の枠は、ナビゲーション項目をまとめるコンテナです。同じコンテナを再利用し、複数の項目を入れた場合は画像左側の横長な形、一つだけ入れた場合は右側の円形に近い形になることを想定しています。

コンテナ自体は、内側にいくつの項目があるかを意識しすぎず、内容に応じて幅や形が決まるようにします。これにより、同じ見た目と振る舞いを持つ別のグループが必要になった場合にも再利用できます。

### 黄色の枠

黄色の枠は、ナビゲーションを構成する最小単位です。一つのアイコンとラベルを持ち、選択すると対応するページへ移動します。

見た目はボタン形式ですが、役割は処理を実行するボタンではなく、別のURLへ移動するリンクです。そのため、実装では前章で使用したReact Routerの`NavLink`を利用します。現在表示しているページも分かるようにし、選択中の項目には別の色を適用する予定です。

## コンポーネント設計について

どこまでを一つのコンポーネントにするか、どのコンポーネントへどの役割を持たせるか、コンポーネント同士をどのように組み合わせるかを考えることを、コンポーネント設計と呼びます。

コンポーネント設計には、Atomic Design、Featureベース、PresentationalとContainerの分離など、さまざまな考え方があります。どれか一つだけが正解というわけではなく、アプリケーションの規模やチーム、再利用したい単位によって適した構成は変わります。

今回は、機能ごとに関連するコンポーネントや処理をまとめるFeatureベースの構成で開発を進めます。たとえば、フッターナビゲーションに固有のコンポーネントは同じ機能のディレクトリへ置き、複数の機能から利用できる汎用的な部品は共通のディレクトリへ分けます。

設計方法の全体像については、[Reactのコンポーネント設計パターンを紹介する記事](https://qiita.com/ktdatascience/items/58a38c0efc915651b2cc)が参考になります。「React コンポーネント設計」や「React Featureベース」などの言葉で検索し、ほかの構成と比較してみるのもよいと思います。

## 使用するライブラリを確認する

ナビゲーションの実装を始める前に、UI、アイコン、フォントに使用するライブラリを確認します。以降のコマンドは`frontend`ディレクトリで実行します。

```bash:ターミナル
cd ~/Develop/pocket-pace-noai/frontend
```

### shadcn/ui

UIコンポーネントには、前々章で初期化したshadcn/uiを使用します。現在は初期化時に作られたButtonコンポーネントだけがある状態です。今後、別のコンポーネントが必要になった場合は、その機能を実装する段階で追加します。

shadcn/uiのコンポーネントはプロジェクト内へソースコードとして追加されるため、Figmaのデザインに合わせてクラスや振る舞いを調整できます。

### Lucide

ナビゲーションのアイコンには[Lucide](https://lucide.dev/)を使用します。shadcn/uiを初期化したときに、アイコンライブラリとしてLucideを選択したため、`lucide-react`はすでに依存関係へ追加されています。

次のコマンドで確認できます。

```bash:ターミナル
pnpm list lucide-react
```

パッケージ名とバージョンが表示されれば、追加のインストールは必要ありません。実際に使用するアイコンは、ナビゲーションのコンポーネントを実装するときに選びます。

## Noto Sans JPを追加する

shadcn/uiのプリセットではNoto Sansを選択しましたが、このアプリケーションでは日本語を中心に表示します。そこで、日本語の文字を含むNoto Sans JPへ切り替えます。

Noto Sans JPの可変フォントを[Fontsource](https://fontsource.org/fonts/noto-sans-jp/install)から追加します。

```bash:ターミナル
pnpm add @fontsource-variable/noto-sans-jp
```

可変フォントは、一つのフォントファイルで複数の太さを扱える形式です。Noto Sans JPの可変フォントでは、100から900までのウェイトを利用できます。

### `app.css`を変更する

`app/app.css`で読み込んでいるNoto SansをNoto Sans JPへ変更します。あわせて、Tailwind CSSで`font-sans`を使用したときにNoto Sans JPが適用されるよう、フォントの変数も変更します。

```diff css:app/app.css
 @import "tailwindcss";
 @import "tw-animate-css";
 @import "shadcn/tailwind.css";
-@import "@fontsource-variable/noto-sans";
+@import "@fontsource-variable/noto-sans-jp";

 /* 省略 */

 @theme inline {
-  --font-sans: "Noto Sans Variable", sans-serif;
+  --font-sans: "Noto Sans JP Variable", sans-serif;
   --font-heading: var(--font-sans);
 }
```

これで、shadcn/uiのコンポーネントを含め、`font-sans`を使用する要素へNoto Sans JPが適用されます。

### `root.tsx`を変更する

React Routerの初期テンプレートでは、Google FontsからInterを読み込む`links`関数が用意されている場合があります。今回はFontsourceからNoto Sans JPを読み込むため、Inter用の`links`関数は削除します。

また、ページで使用する主な言語が日本語であることをブラウザや支援技術へ伝えるため、`html`要素の`lang`属性を`en`から`ja`へ変更します。

```diff tsx:app/root.tsx
 import type { Route } from "./+types/root";
 import "./app.css";

-export const links: Route.LinksFunction = () => [
-  { rel: "preconnect", href: "https://fonts.googleapis.com" },
-  {
-    rel: "preconnect",
-    href: "https://fonts.gstatic.com",
-    crossOrigin: "anonymous",
-  },
-  {
-    rel: "stylesheet",
-    href: "https://fonts.googleapis.com/css2?family=Inter:ital,opsz,wght@0,14..32,100..900;1,14..32,100..900&display=swap",
-  },
-];
-
 export function Layout({ children }: { children: React.ReactNode }) {
   return (
-    <html lang="en">
+    <html lang="ja">
```

使用しなくなったNoto Sansのパッケージを削除します。

```bash:ターミナル
pnpm remove @fontsource-variable/noto-sans
```

これで、Noto Sans JPへの切り替えは完了です。

## コンポーネント用のディレクトリを作る

Featureベースの構成でコンポーネントを整理するため、`app`の下に`features`と`shared`の二つのディレクトリを作ります。

```bash:ターミナル
mkdir -p app/features app/shared
```

`features`には特定の機能に属するコンポーネントや処理を置き、`shared`には複数の機能から利用できる共通部品を置きます。
