---
title: "共通レイアウトとフッターナビゲーションの配置"
---

前章では、フッターナビゲーションを三つのコンポーネントへ分け、ダッシュボードで表示とページ遷移を確認しました。この章では、ナビゲーションを四つのページで共有し、画面下部へ固定します。

## 共通レイアウトを作る

`FooterNav`は、ダッシュボード、支出管理、収入管理、履歴管理のすべてで使用します。各ページから個別に読み込むと、同じ記述が複数のファイルに増えてしまいます。そこで、ページの内容と`FooterNav`をまとめる共通レイアウトを作ります。

### `_app.tsx`を作る

`app/routes/_app.tsx`を作り、次のコードを記述します。

```tsx:app/routes/_app.tsx
import { Outlet } from "react-router";

import FooterNav from "~/features/footer/FooterNav";

export default function AppLayout() {
  return (
    <>
      <main className="min-h-dvh bg-slate-100">
        <Outlet />
      </main>
      <FooterNav />
    </>
  );
}
```

`Outlet`には、現在のURLに対応する子ルートが表示されます。ページの内容と`FooterNav`を同じレイアウトへ置くことで、各ページは自身の内容だけを担当できます。

### 各ページを子ルートにする

各ページが`_app.tsx`の`Outlet`へ表示されるよう、ファイル名を変更します。

```diff text:app/routes
-dashboard.tsx
-expense.tsx
-history.tsx
-income.tsx
+_app.dashboard.tsx
+_app.expense.tsx
+_app.history.tsx
+_app.income.tsx
+_app.tsx
```

ファイル名の先頭にある`_app`は、URLへパスを追加せずに子ルートをまとめます。たとえば、`_app.dashboard.tsx`のURLは、これまでと同じ`/dashboard`です。

前章で`dashboard.tsx`へ追加した`FooterNav`の読み込みは不要になるため、ページの内容だけへ戻します。

```tsx:app/routes/_app.dashboard.tsx
export default function Dashboard() {
  return null;
}
```

ほかの三つのページもファイル名だけを変更し、内容はそのままにします。

## `root.tsx`の役割を絞る

背景色とページの最低高さは、`_app.tsx`の`main`要素へ移しました。そのため、`root.tsx`では`children`を囲んでいた`main`要素を外します。

また、画面下部の安全領域をCSSから利用するため、viewportの設定へ`viewport-fit=cover`を追加します。

```diff tsx:app/root.tsx
 <meta
   name="viewport"
-  content="width=device-width, initial-scale=1"
+  content="width=device-width, initial-scale=1, viewport-fit=cover"
 />

 <body>
-  <main className="min-h-dvh bg-slate-100">{children}</main>
+  {children}
   <ScrollRestoration />
   <Scripts />
 </body>
```

これで、`root.tsx`はHTML全体の土台、`_app.tsx`は認証後に表示するアプリケーション画面のレイアウトを担当します。ルートURLの`/`は、今後ログインなどの認証画面に使用する予定です。

## フッター用の寸法を変数で管理する

フッターナビゲーションを画面下部へ固定すると、通常の文書の流れから外れます。そのままでは、ページ末尾の内容と重なる可能性があります。

ナビゲーションの高さと下端からの距離をCSS変数で定義し、ナビゲーションの配置とページ側の余白で共有します。`app/app.css`の`:root`へ追加します。

```diff css:app/app.css
 :root {
+  --floating-nav-height: 3.5rem;
+  --floating-nav-offset: 1rem;
+  --floating-nav-clearance: calc(
+    var(--floating-nav-height) + var(--floating-nav-offset) + env(safe-area-inset-bottom)
+  );
   /* 既存の変数は省略 */
 }
```

- `--floating-nav-height`: フッターナビゲーションの高さ
- `--floating-nav-offset`: 画面下端とナビゲーションの間隔
- `--floating-nav-clearance`: ページ内容と重ならないために確保する余白

`env(safe-area-inset-bottom)`は、ホームインジケーターなどと重ならないよう、ブラウザが提供する下側の余白です。対象となる領域がない環境では0として扱われます。

変数を用意できたので、`_app.tsx`の`main`要素へ下余白を追加します。

```diff tsx:app/routes/_app.tsx
-<main className="min-h-dvh bg-slate-100">
+<main className="min-h-dvh bg-slate-100 pb-[var(--floating-nav-clearance)]">
```

Tailwind CSSの角括弧は任意値を指定する記法です。`pb-[var(--floating-nav-clearance)]`により、CSS変数の値を`padding-bottom`へ適用します。ナビゲーションの寸法を変更するときはCSS変数を直せば、ページ側の余白も同時に変わります。

## フッターナビゲーションを画面下部へ固定する

前章で付けた確認用の境界線を外し、`FooterNav`を画面下部へ配置します。最終的なコードを一度に置き換えるのではなく、変更する場所を順番に確認します。

### 外側の配置を変更する

```diff tsx:app/features/footer/FooterNav.tsx
-<footer>
-  <nav className="flex gap-2 border border-green-400">
+<footer className="fixed inset-x-0 bottom-[calc(var(--floating-nav-offset)+env(safe-area-inset-bottom))] z-50">
+  <nav className="mx-auto flex w-9/10 max-w-sm items-center gap-2">
```

`fixed`を指定すると、ページをスクロールしてもビューポートを基準にした位置へ表示されます。`inset-x-0`で左右を0にし、`bottom-[calc(...)]`で下端からの間隔とSafe Areaを加えた位置へ固定します。

`w-9/10`で画面幅の90%まで広げ、`max-w-sm`で最大幅を制限します。`mx-auto`により、画面の中央へ配置されます。二つの`PillGroup`は`gap-2`で間隔を空けます。

### 確認用の境界線を外す

```diff tsx:app/features/footer/FooterNav.tsx
 <PillGroup
-  classNames={{ root: "flex-1 border border-red-400" }}
+  classNames={{ root: "flex-1 bg-white p-1" }}
 >
   {/* ページリンクは省略 */}
 </PillGroup>

 <PillGroup
-  classNames={{ root: "w-fit border border-red-400" }}
+  classNames={{ root: "w-fit bg-white" }}
 >
   {/* Otherボタンは省略 */}
 </PillGroup>
```

`LabeledIcon`へ渡していた黄色い境界線も削除します。

```diff tsx:app/features/footer/FooterNav.tsx
 <LabeledIcon
-  classNames={{ root: "border border-yellow-400" }}
   icon={item.icon}
   label={item.label}
 />
```

左側の`PillGroup`へ`flex-1`、右側へ`w-fit`を渡し、ページリンク側だけが利用できる幅まで広がるようにしています。`classNames.root`から`p-1`を渡すと、`PillGroup`の既定値である`p-2`が上書きされます。

### 最終的な`FooterNav`を確認する

変更後のコードは次のようになります。

```tsx:app/features/footer/FooterNav.tsx
import {
  BanknoteArrowDown,
  BanknoteArrowUp,
  Calendar,
  Home,
  type LucideIcon,
  Menu,
} from "lucide-react";
import { NavLink } from "react-router";

import LabeledIcon from "~/features/footer/components/LabeledIcon";
import PillGroup from "~/features/footer/components/PillGroup";

interface NavigationItem {
  to: string;
  label: string;
  icon: LucideIcon;
}

const navigationItems: NavigationItem[] = [
  { to: "/dashboard", label: "Home", icon: Home },
  { to: "/expense", label: "Expense", icon: BanknoteArrowDown },
  { to: "/income", label: "Income", icon: BanknoteArrowUp },
  { to: "/history", label: "History", icon: Calendar },
];

export default function FooterNav() {
  return (
    <footer className="fixed inset-x-0 bottom-[calc(var(--floating-nav-offset)+env(safe-area-inset-bottom))] z-50">
      <nav className="mx-auto flex w-9/10 max-w-sm items-center gap-2">
        <PillGroup classNames={{ root: "flex-1 bg-white p-1" }}>
          {navigationItems.map((item) => (
            <NavLink
              className={({ isActive }) =>
                `flex w-full justify-center rounded-full p-1 ${
                  isActive
                    ? "bg-slate-100 text-sky-400"
                    : "bg-transparent text-neutral-800"
                }`
              }
              end
              key={item.to}
              to={item.to}
            >
              <LabeledIcon icon={item.icon} label={item.label} />
            </NavLink>
          ))}
        </PillGroup>

        <PillGroup classNames={{ root: "w-fit bg-white" }}>
          <button
            className="block rounded-full text-neutral-800"
            type="button"
          >
            <LabeledIcon icon={Menu} label="Other" />
          </button>
        </PillGroup>
      </nav>
    </footer>
  );
}
```

「Other」はFigmaのレイアウトを保つために表示しますが、MVPの要件には含めません。現段階ではクリック時の処理を持たせず、設定などを実装する段階でメニューを開く処理を追加します。

ブラウザで確認すると、フッターナビゲーションは次のようになりました。

![完成したフッターナビゲーション](/images/7464cf3f3b7b86/footer-end.png =436x)

左側には四つのページリンク、右側には独立した「Other」ボタンが表示されています。現在のURLに対応するHomeには、水色の文字と薄い背景色が適用されています。確認用に付けていた3色の境界線は表示されません。

## 動作を確認する

開発サーバーを起動します。

```bash:ターミナル
pnpm dev
```

次の内容を確認します。

- `/dashboard`、`/expense`、`/income`、`/history`のすべてにナビゲーションが表示される
- リンクを選択するとURLと選択中の色が切り替わる
- ページをスクロールしてもナビゲーションが画面下部に残る
- ページ末尾の内容がナビゲーションと重ならない

最後に、本番用のビルドも確認します。

```bash:ターミナル
pnpm build
```

## この章の最終コード

ここまでの変更を反映した主要なファイルをまとめます。各ファイル名を選択するとコードを開けます。`FooterNav.tsx`は直前に掲載したコードが最終形です。

:::details frontend/app/routes/_app.tsx

```tsx:app/routes/_app.tsx
import { Outlet } from "react-router";

import FooterNav from "~/features/footer/FooterNav";

export default function AppLayout() {
  return (
    <>
      <main className="min-h-dvh bg-slate-100 pb-[var(--floating-nav-clearance)]">
        <Outlet />
      </main>
      <FooterNav />
    </>
  );
}
```
:::

:::details frontend/app/routes/_app.dashboard.tsx
```tsx:app/routes/_app.dashboard.tsx
export default function Dashboard() {
  return null;
}
```
:::

:::details frontend/app/routes/_app.expense.tsx
```tsx:app/routes/_app.expense.tsx
export default function Expense() {
  return null;
}
```
:::

:::details frontend/app/routes/_app.income.tsx
```tsx:app/routes/_app.income.tsx
export default function Income() {
  return null;
}
```
:::

:::details frontend/app/routes/_app.history.tsx
```tsx:app/routes/_app.history.tsx
export default function History() {
  return null;
}
```
:::

:::details frontend/app/features/footer/components/LabeledIcon.tsx
```tsx:app/features/footer/components/LabeledIcon.tsx
import type { ComponentType } from "react";

import { cn } from "~/lib/utils";

interface LabeledIconProps {
  icon: ComponentType<{ className?: string }>;
  label: string;
  classNames?: {
    root?: string;
    icon?: string;
    label?: string;
  };
}

export default function LabeledIcon({
  icon: Icon,
  label,
  classNames,
}: LabeledIconProps) {
  return (
    <span
      className={cn(
        "flex size-10 flex-col items-center justify-center text-center",
        classNames?.root,
      )}
    >
      <Icon className={cn("size-6", classNames?.icon)} />
      <span className={cn("text-[10px] leading-none", classNames?.label)}>
        {label}
      </span>
    </span>
  );
}
```
:::

:::details frontend/app/features/footer/components/PillGroup.tsx
```tsx:app/features/footer/components/PillGroup.tsx
import type { ReactNode } from "react";

import { cn } from "~/lib/utils";

interface PillGroupProps {
  children: ReactNode;
  classNames?: {
    root?: string;
  };
}

export default function PillGroup({ children, classNames }: PillGroupProps) {
  return (
    <div
      className={cn(
        "flex w-full items-center justify-around rounded-full bg-white p-2",
        classNames?.root,
      )}
    >
      {children}
    </div>
  );
}
```
:::

:::details frontend/app/features/footer/FooterNav.tsx
```tsx:app/features/footer/FooterNav.tsx
import {
  BanknoteArrowDown,
  BanknoteArrowUp,
  Calendar,
  Home,
  type LucideIcon,
  Menu,
} from "lucide-react";
import { NavLink } from "react-router";

import LabeledIcon from "~/features/footer/components/LabeledIcon";
import PillGroup from "~/features/footer/components/PillGroup";

interface NavigationItem {
  to: string;
  label: string;
  icon: LucideIcon;
}

const navigationItems: NavigationItem[] = [
  { to: "/dashboard", label: "Home", icon: Home },
  { to: "/expense", label: "Expense", icon: BanknoteArrowDown },
  { to: "/income", label: "Income", icon: BanknoteArrowUp },
  { to: "/history", label: "History", icon: Calendar },
];

export default function FooterNav() {
  return (
    <footer className="fixed inset-x-0 bottom-[calc(var(--floating-nav-offset)+env(safe-area-inset-bottom))] z-50">
      <nav className="mx-auto flex w-9/10 max-w-sm items-center gap-2">
        <PillGroup classNames={{ root: "flex-1 bg-white p-1" }}>
          {navigationItems.map((item) => (
            <NavLink
              className={({ isActive }) =>
                `flex w-full justify-center rounded-full p-1 ${
                  isActive
                    ? "bg-slate-100 text-sky-400"
                    : "bg-transparent text-neutral-800"
                }`
              }
              end
              key={item.to}
              to={item.to}
            >
              <LabeledIcon icon={item.icon} label={item.label} />
            </NavLink>
          ))}
        </PillGroup>

        <PillGroup classNames={{ root: "w-fit bg-white" }}>
          <button
            className="block rounded-full text-neutral-800"
            type="button"
          >
            <LabeledIcon icon={Menu} label="Other" />
          </button>
        </PillGroup>
      </nav>
    </footer>
  );
}
```
:::

:::details frontend/app/root.tsx
```tsx:app/root.tsx
import {
  isRouteErrorResponse,
  Links,
  Meta,
  Outlet,
  Scripts,
  ScrollRestoration,
} from "react-router";

import type { Route } from "./+types/root";
import "./app.css";

export function Layout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="ja">
      <head>
        <meta charSet="utf-8" />
        <meta
          name="viewport"
          content="width=device-width, initial-scale=1, viewport-fit=cover"
        />
        <Meta />
        <Links />
      </head>
      <body>
        {children}
        <ScrollRestoration />
        <Scripts />
      </body>
    </html>
  );
}

export default function App() {
  return <Outlet />;
}

export function ErrorBoundary({ error }: Route.ErrorBoundaryProps) {
  let message = "Oops!";
  let details = "An unexpected error occurred.";
  let stack: string | undefined;

  if (isRouteErrorResponse(error)) {
    message = error.status === 404 ? "404" : "Error";
    details =
      error.status === 404
        ? "The requested page could not be found."
        : error.statusText || details;
  } else if (import.meta.env.DEV && error && error instanceof Error) {
    details = error.message;
    stack = error.stack;
  }

  return (
    <main className="pt-16 p-4 container mx-auto">
      <h1>{message}</h1>
      <p>{details}</p>
      {stack && (
        <pre className="w-full p-4 overflow-x-auto">
          <code>{stack}</code>
        </pre>
      )}
    </main>
  );
}
```
:::

:::details frontend/app/app.css
```css:app/app.css
@import "tailwindcss";
@import "tw-animate-css";
@import "shadcn/tailwind.css";
@import "@fontsource-variable/noto-sans-jp";

@custom-variant dark (&:is(.dark *));

html,
body {
  @apply bg-white dark:bg-gray-950;

  @media (prefers-color-scheme: dark) {
    color-scheme: dark;
  }
}

:root {
  --floating-nav-height: 3.5rem;
  --floating-nav-offset: 1rem;
  --floating-nav-clearance: calc(
    var(--floating-nav-height) + var(--floating-nav-offset) + env(safe-area-inset-bottom)
  );
  --background: oklch(1 0 0);
  --foreground: oklch(0.145 0 0);
  --card: oklch(1 0 0);
  --card-foreground: oklch(0.145 0 0);
  --popover: oklch(1 0 0);
  --popover-foreground: oklch(0.145 0 0);
  --primary: oklch(0.205 0 0);
  --primary-foreground: oklch(0.985 0 0);
  --secondary: oklch(0.97 0 0);
  --secondary-foreground: oklch(0.205 0 0);
  --muted: oklch(0.97 0 0);
  --muted-foreground: oklch(0.556 0 0);
  --accent: oklch(0.97 0 0);
  --accent-foreground: oklch(0.205 0 0);
  --destructive: oklch(0.577 0.245 27.325);
  --border: oklch(0.922 0 0);
  --input: oklch(0.922 0 0);
  --ring: oklch(0.708 0 0);
  --chart-1: oklch(0.87 0 0);
  --chart-2: oklch(0.556 0 0);
  --chart-3: oklch(0.439 0 0);
  --chart-4: oklch(0.371 0 0);
  --chart-5: oklch(0.269 0 0);
  --radius: 0.45rem;
  --sidebar: oklch(0.985 0 0);
  --sidebar-foreground: oklch(0.145 0 0);
  --sidebar-primary: oklch(0.205 0 0);
  --sidebar-primary-foreground: oklch(0.985 0 0);
  --sidebar-accent: oklch(0.97 0 0);
  --sidebar-accent-foreground: oklch(0.205 0 0);
  --sidebar-border: oklch(0.922 0 0);
  --sidebar-ring: oklch(0.708 0 0);
}

.dark {
  --background: oklch(0.145 0 0);
  --foreground: oklch(0.985 0 0);
  --card: oklch(0.205 0 0);
  --card-foreground: oklch(0.985 0 0);
  --popover: oklch(0.205 0 0);
  --popover-foreground: oklch(0.985 0 0);
  --primary: oklch(0.922 0 0);
  --primary-foreground: oklch(0.205 0 0);
  --secondary: oklch(0.269 0 0);
  --secondary-foreground: oklch(0.985 0 0);
  --muted: oklch(0.269 0 0);
  --muted-foreground: oklch(0.708 0 0);
  --accent: oklch(0.269 0 0);
  --accent-foreground: oklch(0.985 0 0);
  --destructive: oklch(0.704 0.191 22.216);
  --border: oklch(1 0 0 / 10%);
  --input: oklch(1 0 0 / 15%);
  --ring: oklch(0.556 0 0);
  --chart-1: oklch(0.87 0 0);
  --chart-2: oklch(0.556 0 0);
  --chart-3: oklch(0.439 0 0);
  --chart-4: oklch(0.371 0 0);
  --chart-5: oklch(0.269 0 0);
  --sidebar: oklch(0.205 0 0);
  --sidebar-foreground: oklch(0.985 0 0);
  --sidebar-primary: oklch(0.488 0.243 264.376);
  --sidebar-primary-foreground: oklch(0.985 0 0);
  --sidebar-accent: oklch(0.269 0 0);
  --sidebar-accent-foreground: oklch(0.985 0 0);
  --sidebar-border: oklch(1 0 0 / 10%);
  --sidebar-ring: oklch(0.556 0 0);
}

@theme inline {
  --font-sans: "Noto Sans JP Variable", sans-serif;
  --font-heading: var(--font-sans);
  --color-sidebar-ring: var(--sidebar-ring);
  --color-sidebar-border: var(--sidebar-border);
  --color-sidebar-accent-foreground: var(--sidebar-accent-foreground);
  --color-sidebar-accent: var(--sidebar-accent);
  --color-sidebar-primary-foreground: var(--sidebar-primary-foreground);
  --color-sidebar-primary: var(--sidebar-primary);
  --color-sidebar-foreground: var(--sidebar-foreground);
  --color-sidebar: var(--sidebar);
  --color-chart-5: var(--chart-5);
  --color-chart-4: var(--chart-4);
  --color-chart-3: var(--chart-3);
  --color-chart-2: var(--chart-2);
  --color-chart-1: var(--chart-1);
  --color-ring: var(--ring);
  --color-input: var(--input);
  --color-border: var(--border);
  --color-destructive: var(--destructive);
  --color-accent-foreground: var(--accent-foreground);
  --color-accent: var(--accent);
  --color-muted-foreground: var(--muted-foreground);
  --color-muted: var(--muted);
  --color-secondary-foreground: var(--secondary-foreground);
  --color-secondary: var(--secondary);
  --color-primary-foreground: var(--primary-foreground);
  --color-primary: var(--primary);
  --color-popover-foreground: var(--popover-foreground);
  --color-popover: var(--popover);
  --color-card-foreground: var(--card-foreground);
  --color-card: var(--card);
  --color-foreground: var(--foreground);
  --color-background: var(--background);
  --radius-sm: calc(var(--radius) * 0.6);
  --radius-md: calc(var(--radius) * 0.8);
  --radius-lg: var(--radius);
  --radius-xl: calc(var(--radius) * 1.4);
  --radius-2xl: calc(var(--radius) * 1.8);
  --radius-3xl: calc(var(--radius) * 2.2);
  --radius-4xl: calc(var(--radius) * 2.6);
}

@layer base {
  * {
    @apply border-border outline-ring/50;
  }
  body {
    @apply bg-background text-foreground;
  }
  html {
    @apply font-sans;
  }
}
```
:::

## まとめ

この章では、`_app.tsx`へページの表示領域と`FooterNav`をまとめ、四つのページで共通のレイアウトを使用できるようにしました。

さらに、フッターナビゲーションの寸法をCSS変数で管理し、Safe Areaを含めて画面下部へ固定しました。これで、各ページの内容を実装するときも、ナビゲーションを個別に記述する必要はありません。
