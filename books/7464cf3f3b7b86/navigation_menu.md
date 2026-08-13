---
title: "フッターナビゲーションのコンポーネント設計"
---

前章では、ダッシュボード、支出管理、収入管理、履歴管理の4ページを作成し、React Routerでページ間を移動できるようにしました。ただし、現時点では各ページに同じリンクを直接記述しただけで、4章で設計した画面下部のナビゲーションにはなっていません。

この章では、前章で作ったルーティングをナビゲーションメニューへ仕立てます。まずは部品へ分けて実装し、コンポーネントの役割と組み合わせ方を確認します。すべてのページへ共通して配置する方法は、次章で扱います。

## デザインをコンポーネントへ分ける

Figmaで作ったデザインをそのまま一つの大きなコンポーネントへ変換するのではなく、役割を考えていくつかの部品へ分けます。今回は、4章で作成した画面下部のナビゲーションを次のように分けました。

![フッターナビゲーションのコンポーネント分割](/images/7464cf3f3b7b86/footer-nav.png =500x)

### 緑色の枠

緑色の枠は、ナビゲーション全体を包む`FooterNav`です。メニュー全体の幅や余白、内側にある項目の並び方を担当します。

### 赤色の枠

赤色の枠は、複数の項目をまとめる`PillGroup`です。内側に渡された項目の配置と、背景色や角丸などの見た目を担当します。

Figmaの案では、左側にページを移動する四つのリンク、右側に「その他」メニューを開くボタンを置いています。「その他」メニューはMVPの要件に含めませんが、レイアウトは残したいため、現段階ではボタンの表示だけを実装します。設定などが必要になった段階で、メニューを開く処理を追加する予定です。

### 黄色の枠

黄色の枠は、アイコンとラベルを組み合わせる`LabeledIcon`です。外側から受け取ったアイコンと文字列を、決められた間隔と大きさで並べます。

`LabeledIcon`自体には移動先のURLを持たせません。`FooterNav`がReact Routerの`NavLink`と組み合わせ、ページを移動できる項目にします。

## コンポーネントの配置を決める

今回は、機能ごとに関連するファイルをまとめるFeatureベースの構成で開発を進めます。フッターナビゲーションで使用する部品は、同じ機能のディレクトリへまとめます。

```bash:ターミナル
mkdir -p app/features/footer/components
```

作成するファイルを含めると、構成は次のようになります。

```text:ディレクトリ構成
app/
└── features/
    └── footer/
        ├── components/
        │   ├── LabeledIcon.tsx
        │   └── PillGroup.tsx
        └── FooterNav.tsx
```

`LabeledIcon`と`PillGroup`は、現時点ではフッターナビゲーションでしか使用しません。将来ほかの機能でも使う可能性だけで共通部品にはせず、まずは`footer`の中へ置きます。実際に複数の機能から使うことになった段階で、`shared`などの共通ディレクトリへ移します。

設計方法の全体像については、[Reactのコンポーネント設計パターンを紹介する記事](https://qiita.com/ktdatascience/items/58a38c0efc915651b2cc)が参考になりました。

## 使用するライブラリを確認する

以降のコマンドは`frontend`ディレクトリで実行します。

```bash:ターミナル
cd ~/Develop/pocket-pace-noai/frontend
```

ナビゲーションのアイコンには[Lucide](https://lucide.dev/)を使用します。shadcn/uiを初期化したときにLucideを選択したため、`lucide-react`はすでに依存関係へ追加されています。

```bash:ターミナル
pnpm list lucide-react
```

パッケージ名とバージョンが表示されれば、追加のインストールは必要ありません。

## Noto Sans JPを追加する

shadcn/uiのプリセットではNoto Sansを選択しましたが、このアプリケーションでは日本語を中心に表示します。そこで、日本語の文字を含むNoto Sans JPへ切り替えます。

```bash:ターミナル
pnpm add @fontsource-variable/noto-sans-jp
```

`app/app.css`で読み込むフォントと、Tailwind CSSで`font-sans`を使用したときのフォントを変更します。

```diff css:app/app.css
-@import "@fontsource-variable/noto-sans";
+@import "@fontsource-variable/noto-sans-jp";

 @theme inline {
-  --font-sans: "Noto Sans Variable", sans-serif;
+  --font-sans: "Noto Sans JP Variable", sans-serif;
 }
```

`app/root.tsx`にGoogle FontsからInterを読み込む`links`関数がある場合は削除します。また、`html`要素の`lang`属性を日本語へ変更します。

```diff tsx:app/root.tsx
-export const links: Route.LinksFunction = () => [
-  /* Interを読み込む設定 */
-];
-
 export function Layout({ children }: { children: React.ReactNode }) {
   return (
-    <html lang="en">
+    <html lang="ja">
```

アプリケーション全体の背景色もFigmaのデザインへ近づけます。

```diff tsx:app/root.tsx
 <body>
-  {children}
+  <main className="min-h-dvh bg-slate-100">{children}</main>
   <ScrollRestoration />
   <Scripts />
 </body>
```

使用しなくなったNoto Sansのパッケージを削除します。

```bash:ターミナル
pnpm remove @fontsource-variable/noto-sans
```

## アイコンとラベルを組み合わせる

`app/features/footer/components/LabeledIcon.tsx`を作ります。

```tsx:app/features/footer/components/LabeledIcon.tsx
import type { ComponentType } from "react";

interface LabeledIconProps {
  icon: ComponentType;
  label: string;
}

export default function LabeledIcon({
  icon: Icon,
  label,
}: LabeledIconProps) {
  return (
    <span className="border border-yellow-400">
      <Icon />
      <span>{label}</span>
    </span>
  );
}
```

Propsには、アイコンを受け取る`icon`と、文字列を受け取る`label`を定義しました。どちらも表示に必要なため、必須にしています。

`ComponentType`は、Reactコンポーネントを受け取るための型です。`icon`を分割代入するときに`Icon`という大文字から始まる名前へ変更すると、受け取ったLucideのアイコンを`<Icon />`として描画できます。

このコンポーネントにはURLやクリック時の処理を持たせていません。そのため、利用する側でリンクやボタンと組み合わせられます。ルートには`span`を使用し、どちらの内側でも使える形にしました。

`border border-yellow-400`は完成デザインのスタイルではありません。図で黄色く囲んだ範囲と実際のコンポーネントが対応しているか、ブラウザ上で確認するための仮の境界線です。

## 項目をまとめるコンテナを作る

`app/features/footer/components/PillGroup.tsx`を作ります。

```tsx:app/features/footer/components/PillGroup.tsx
import type { ReactNode } from "react";

interface PillGroupProps {
  children: ReactNode;
}

export default function PillGroup({ children }: PillGroupProps) {
  return (
    <div className="border border-red-400 bg-white">
      {children}
    </div>
  );
}
```

`children`は、コンポーネントの開始タグと終了タグの間へ渡された内容です。型には、Reactが画面へ描画できる値を表す`ReactNode`を指定します。

`PillGroup`自身は、内側へ何個の部品が渡されたかや、それらがリンクかボタンかを判断しません。受け取った`children`をそのまま表示します。ここでも、コンポーネントの範囲を確認するため、仮の赤い境界線を付けています。

## `FooterNav`で部品を組み合わせる

`app/features/footer/FooterNav.tsx`を作り、ページの情報を配列へまとめます。

```tsx:app/features/footer/FooterNav.tsx
import {
  BanknoteArrowDown,
  BanknoteArrowUp,
  Calendar,
  Home,
  Menu,
} from "lucide-react";
import type { ComponentType } from "react";
import { NavLink } from "react-router";

import LabeledIcon from "~/features/footer/components/LabeledIcon";
import PillGroup from "~/features/footer/components/PillGroup";

interface NavigationItem {
  to: string;
  label: string;
  icon: ComponentType;
}

const navigationItems: NavigationItem[] = [
  { to: "/dashboard", label: "Home", icon: Home },
  { to: "/expense", label: "Expense", icon: BanknoteArrowDown },
  { to: "/income", label: "Income", icon: BanknoteArrowUp },
  { to: "/history", label: "History", icon: Calendar },
];

export default function FooterNav() {
  return (
    <footer>
      <nav className="border border-green-400">
        <PillGroup>
          {navigationItems.map((item) => (
            <NavLink
              to={item.to}
              end
              key={item.to}
              className={({ isActive }) =>
                `block ${
                  isActive
                    ? "bg-slate-100 text-blue-400"
                    : "text-neutral-800"
                }`
              }
            >
              <LabeledIcon icon={item.icon} label={item.label} />
            </NavLink>
          ))}
        </PillGroup>

        <PillGroup>
          <button className="block text-neutral-800" type="button">
            <LabeledIcon icon={Menu} label="Other" />
          </button>
        </PillGroup>
      </nav>
    </footer>
  );
}
```

`navigationItems`には、移動先のURL、ラベル、アイコンをページごとにまとめました。配列を`map`で一つずつ`NavLink`へ変換するため、項目を増減するときは配列を変更します。

`NavLink`の`className`へ関数を渡すと、現在のURLとリンク先が一致しているかを`isActive`で確認できます。表示中のページには薄いグレーの背景と青い文字を適用しました。これらも動作を確認するための仮のスタイルです。

「Other」はボタンとして表示しますが、現段階では選択したときの処理を持たせません。機能はMVPの後に実装します。

`type="button"`は、将来このナビゲーションがフォーム内に置かれた場合に、意図せずフォームを送信しないための指定です。

## 途中の状態を確認する

作成した`FooterNav`を確認するため、`app/routes/dashboard.tsx`で読み込みます。

```tsx:app/routes/dashboard.tsx
import FooterNav from "~/features/footer/FooterNav";

export default function Dashboard() {
  return <FooterNav />;
}
```

開発サーバーを起動して`/dashboard`を表示すると、次のようになります。

![コンポーネントの範囲を境界線で確認する](/images/7464cf3f3b7b86/footer-nav01.png =402x)

黄色の線は`LabeledIcon`、赤色の線は`PillGroup`、緑色の線は`FooterNav`内の`nav`要素の範囲です。四つのリンクが最初の`PillGroup`へまとまり、「Other」ボタンが二つ目の`PillGroup`へ入っていることを確認できます。また、現在表示しているHomeだけに選択中の色が適用されています。

まだ項目は縦に並び、画面下部にも配置されていません。境界線も構造を確認するための仮のスタイルです。この段階でコンポーネントの分け方とリンクの動作を確認し、次にそれぞれの役割に沿って見た目を整えます。

## 外側からスタイルを変更できるようにする

ここまでの`LabeledIcon`と`PillGroup`では、使用するクラスをコンポーネントの内部へ直接記述していました。この方法でも共通した見た目は作れますが、利用する場所に応じて一部のスタイルだけを変更できません。

そこで、コンポーネントが持つ既定のクラスへ、外側から渡したクラスを追加できるようにします。クラス名を組み立てるため、shadcn/uiの初期化時に作成された`cn`関数を使用します。

### `cn`の仕組みを確認する

`cn`は`app/lib/utils.ts`に定義されています。

```ts:app/lib/utils.ts
import { clsx, type ClassValue } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

`clsx`は、複数のクラス名や条件付きのクラス名を一つの文字列へまとめます。`tailwind-merge`は、まとめたTailwind CSSのクラスに競合がある場合、後から指定されたクラスを残します。

たとえば、コンポーネント内部で`size-10`を指定し、外側から`size-12`を渡した場合を考えます。

```tsx
cn("size-10", "size-12");
```

どちらも要素の大きさを指定するクラスなので、`tailwind-merge`によって後から渡した`size-12`が残ります。これにより、既定のスタイルを利用しながら、必要な部分だけを利用側から変更できます。

### `LabeledIcon`へクラスを渡せるようにする

`LabeledIcon`は、全体、アイコン、ラベルの三か所にクラスを渡せるようにします。

```diff tsx:app/features/footer/components/LabeledIcon.tsx
 import type { ComponentType } from "react";
+import { cn } from "~/lib/utils";

 interface LabeledIconProps {
-  icon: ComponentType;
+  icon: ComponentType<{ className?: string }>;
   label: string;
+  classNames?: {
+    root?: string;
+    icon?: string;
+    label?: string;
+  };
 }

 export default function LabeledIcon({
   icon: Icon,
   label,
+  classNames,
 }: LabeledIconProps) {
   return (
-    <span className="border border-yellow-400">
-      <Icon />
-      <span>{label}</span>
+    <span
+      className={cn(
+        "flex size-10 flex-col items-center justify-center text-center",
+        classNames?.root,
+      )}
+    >
+      <Icon className={cn("size-6", classNames?.icon)} />
+      <span className={cn("text-[10px] leading-none", classNames?.label)}>
+        {label}
+      </span>
     </span>
   );
 }
```

`classNames`は省略可能なPropsです。その中に、最も外側の`span`へ渡す`root`、アイコンへ渡す`icon`、ラベルの`span`へ渡す`label`を用意しました。変更したい場所が名前から分かるため、一つの`className`ですべてを扱うよりも利用側で指定しやすくなります。

既定のクラスを`cn`の第1引数、外側から受け取ったクラスを第2引数へ渡します。`classNames`が省略された場合は、既定のクラスだけが適用されます。

### `PillGroup`へクラスを渡せるようにする

`PillGroup`にも、最も外側の要素を変更するための`root`を追加します。

```diff tsx:app/features/footer/components/PillGroup.tsx
 import type { ReactNode } from "react";
+import { cn } from "~/lib/utils";

 interface PillGroupProps {
   children: ReactNode;
+  classNames?: {
+    root?: string;
+  };
 }

-export default function PillGroup({ children }: PillGroupProps) {
+export default function PillGroup({ children, classNames }: PillGroupProps) {
   return (
-    <div className="border border-red-400 bg-white">
+    <div
+      className={cn(
+        "flex w-full justify-around bg-white p-1",
+        classNames?.root,
+      )}
+    >
       {children}
     </div>
   );
 }
```

`PillGroup`は現時点では外側の`div`だけを持つため、`root`のみを用意しました。必要になっていない差し込み口は先に増やしません。

### `FooterNav`から確認用のクラスを渡す

`FooterNav`では、`classNames`を使って赤色と黄色の境界線を渡します。

```diff tsx:app/features/footer/FooterNav.tsx
-<nav className="border border-green-400">
-  <PillGroup>
+<nav className="flex gap-2 border border-green-400">
+  <PillGroup
+    classNames={{ root: "flex-1 border border-red-400" }}
+  >
     {/* 省略 */}
-    <LabeledIcon icon={item.icon} label={item.label} />
+    <LabeledIcon
+      icon={item.icon}
+      label={item.label}
+      classNames={{ root: "border border-yellow-400" }}
+    />
     {/* 省略 */}
   </PillGroup>

-  <PillGroup>
+  <PillGroup
+    classNames={{ root: "w-fit border border-red-400" }}
+  >
     <button className="block text-neutral-800" type="button">
-      <LabeledIcon icon={Menu} label="Other" />
+      <LabeledIcon
+        icon={Menu}
+        label="Other"
+        classNames={{ root: "border border-yellow-400" }}
+      />
     </button>
   </PillGroup>
 </nav>
```

境界線は`LabeledIcon`や`PillGroup`の内部には含まれなくなりました。どのような見た目で使うかを知っている`FooterNav`から渡しています。

また、`LabeledIcon`ではアイコンとラベルを縦に並べ、`PillGroup`では内側の部品を横に並べました。`FooterNav`の`nav`要素にも`flex`を指定し、二つの`PillGroup`を横に配置します。

## 外側からスタイルを渡した状態を確認する

ここまでの変更をブラウザで確認すると、次のようになります。

![外側からスタイルを渡した途中経過](/images/7464cf3f3b7b86/footer-nav02.png =404x)

最初の途中経過では縦に並んでいたアイコンとラベルが整列し、`PillGroup`内の項目も横に並びます。境界線は引き続き表示されていますが、今回は各コンポーネントの内部へ固定せず、`FooterNav`から渡したものです。

## フッターナビゲーションの見た目を整える

コンポーネントの分け方とスタイルの受け渡しを確認できたので、Figmaのデザインへ近づけます。レイアウトはまだ調整途中なので、3色の境界線は残しておきます。

### `PillGroup`を丸いコンテナにする

`PillGroup`の既定スタイルを変更します。

```diff tsx:app/features/footer/components/PillGroup.tsx
 <div
   className={cn(
-    "flex w-full justify-around bg-white p-1",
+    "flex w-full items-center justify-around rounded-full bg-white p-2",
     classNames?.root,
   )}
 >
```

`items-center`で内側の部品を縦方向の中央へそろえ、`rounded-full`で両端を十分に丸くします。`p-2`で内側に余白を設けます。影は付けず、白い背景と周囲の色の違いで区切ります。

### 選択中の項目を丸くする

`NavLink`と「Other」の`button`へ角丸を追加します。

```diff tsx:app/features/footer/FooterNav.tsx
 className={({ isActive }) =>
-  `block ${
+  `flex w-full justify-center rounded-full p-1 ${
     isActive
-      ? "bg-slate-100 text-blue-400"
-      : "text-neutral-800"
+      ? "bg-slate-100 text-sky-400"
+      : "bg-transparent text-neutral-800"
   }`
 }

-<button className="block text-neutral-800" type="button">
+<button className="block rounded-full text-neutral-800" type="button">
```

背景色は操作を担当する親の`NavLink`へ適用し、内側の`LabeledIcon`はその色を引き継ぎます。`rounded-full`も親へ指定することで、選択中の背景と実際に選択できる範囲をそろえます。

これで、四つのページリンクと「Other」ボタンを持つナビゲーションの形が整いました。「Other」は見た目だけを作り、クリック時の処理はMVPの後に追加します。

現段階ではダッシュボードにだけナビゲーションを置いており、画面下部にも固定していません。次章では共通レイアウトを作り、四つのページすべてへ表示したうえで、画面下部へ配置します。

## まとめ

この章では、まず仮の境界線を使って`LabeledIcon`、`PillGroup`、`FooterNav`の範囲を確認し、その後でアイコンや項目の並び方を整えました。

また、`classNames`と`cn`を追加し、コンポーネントの既定スタイルを利用側から場所ごとに調整できるようにしました。次章では、このナビゲーションを共通レイアウトへ移し、画面下部へ固定します。
