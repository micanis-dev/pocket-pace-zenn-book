---
title: "共通ヘッダーの作成"
---

画面下のナビゲーションを共有できたので、今度は画面上のヘッダーを作ります。Figmaにあるアプリ名とユーザーアイコンを表示し、こちらも4ページで使えるようにします。

実装の前に、ファイルをどこへ置くかで迷いました。今回はFeatureベースという分け方を試し、ヘッダー専用の部品は`features/header`へ置きます。shadcn/uiのAvatarのように、ほかでも使えそうな部品は`shared/ui`へ置くことにしました。

## デザインをコンポーネントへ分ける

Figmaで作成したヘッダーを、役割の異なる三つのコンポーネントへ分けます。

![ヘッダーのコンポーネント分割](/images/7464cf3f3b7b86/header-figma.png =475x)

### 緑色の枠

緑色の枠は、ヘッダー全体を包む`Header`です。白い背景、ヘッダーの高さ、内側の横幅と、左右にある要素の配置を担当します。

`Header`はアプリケーション名やユーザーアイコンの細かな見た目を直接持たず、内側のコンポーネントを横並びにします。

### 黄色の枠

黄色の枠は、ヘッダーを構成する最小単位です。左側の`AppLogo`はアプリケーション名、右側の`UserAvatar`は利用者を表すアイコンを担当します。

`UserAvatar`の内側では、shadcn/uiから追加する汎用的な`Avatar`を使用します。`Avatar`そのものはヘッダー専用ではないため共有層へ置き、グラデーションやアクセシビリティの設定はヘッダー機能の`UserAvatar`で加えます。

すべてを`Header.tsx`へ直接記述しても同じ見た目は作れますが、アプリケーション名、ユーザーアイコン、全体の配置では変更する理由が異なります。そこで、最初は一つにまとまっていた表示を`AppLogo`と`UserAvatar`へ分け、`Header`は二つの部品を配置する役割へ絞ります。

## FeatureとSharedの役割を分ける

これまでに作成したフッターナビゲーションは、ページ間を移動するというアプリケーション固有の役割を持っています。そのため、`app/features/footer`へ配置しました。今回作成する`Header`、`AppLogo`、`UserAvatar`もPocketPaceのヘッダーを構成するため、`app/features/header`へ配置します。

一方、shadcn/uiのButtonやAvatarは、どの機能からでも利用できる汎用的なUI部品です。たとえばAvatarは、ヘッダー以外にもアカウント設定や取引履歴で利用する可能性があります。このような部品を特定のFeatureへ置くと、ほかのFeatureから利用するときに依存関係が分かりにくくなります。

そこで、このプロジェクトでは次の基準で配置します。

| 配置先 | 役割 | 例 |
| --- | --- | --- |
| `app/features` | アプリケーション固有の機能 | ヘッダー、フッターナビゲーション |
| `app/shared/ui` | 機能に依存しないUI部品 | Button、Avatar |
| `app/shared/lib` | 機能に依存しない関数 | クラス名を組み立てる`cn` |

shadcn/uiを初期化した直後は、Buttonが`app/components/ui`、`cn`が`app/lib`へ作成されていました。どちらも共有して使うものなので、この章で`shared`へ移します。

## shadcn/uiの生成先を変更する

shadcn/uiのコンポーネントを追加するときの生成先は、プロジェクトルートにある`components.json`の`aliases`で設定します。今後追加する部品も`shared`へ揃うように変更します。

```diff json:components.json
 "aliases": {
-  "components": "~/components",
-  "utils": "~/lib/utils",
-  "ui": "~/components/ui",
-  "lib": "~/lib",
-  "hooks": "~/hooks"
+  "components": "~/shared",
+  "utils": "~/shared/lib/utils",
+  "ui": "~/shared/ui",
+  "lib": "~/shared/lib",
+  "hooks": "~/shared/hooks"
 }
```

`ui`はshadcn/uiのコンポーネント、`utils`は生成されたコンポーネントが利用する共通関数の参照先です。`components`、`lib`、`hooks`も同じ共有層へ揃えておくと、あとから別の部品を追加したときも配置が分散しません。

## 既存の共通ファイルを移動する

共有層のディレクトリを作り、shadcn/uiの初期化時に追加されたButtonと`cn`を移動します。

```bash:ターミナル
mkdir -p app/shared/lib app/shared/ui

mv app/lib/utils.ts app/shared/lib/utils.ts
mv app/components/ui/button.tsx app/shared/ui/button.tsx
```

移動後は、`cn`を読み込んでいるファイルのimportを変更します。現時点では、Buttonに加えて、前章までに作成した`LabeledIcon`と`PillGroup`が対象です。

```diff tsx:app/shared/ui/button.tsx、app/features/footer/components/*.tsx
-import { cn } from "~/lib/utils";
+import { cn } from "~/shared/lib/utils";
```

古い参照が残っていないことは、次のコマンドで確認できます。

```bash:ターミナル
rg '"~/lib/utils"' app
```

何も表示されなければ、すべて新しいパスへ変更できています。`rg`をインストールしていない場合は、エディターのプロジェクト内検索を使っても構いません。

## Avatarを追加する

ヘッダーのユーザーアイコンには、shadcn/uiのAvatarを使用します。`frontend`ディレクトリで次のコマンドを実行します。

```bash:ターミナル
pnpm exec shadcn add avatar
```

`components.json`の`ui`を変更しているため、Avatarは`app/shared/ui/avatar.tsx`へ追加されます。shadcn/uiはコンポーネントのソースコードをプロジェクトへ追加する仕組みなので、このファイルもアプリケーション側で変更できます。

ここまでの主な構成は次のようになります。

```text:ディレクトリ構成
app/
├── features/
│   ├── footer/
│   └── header/
│       ├── components/
│       │   ├── AppLogo.tsx
│       │   └── UserAvatar.tsx
│       └── Header.tsx
└── shared/
    ├── lib/
    │   └── utils.ts
    └── ui/
        ├── avatar.tsx
        └── button.tsx
```

`shared/ui/avatar.tsx`はshadcn/uiが生成した汎用部品のままにします。PocketPace固有の表示や配置は、`features/header`内のコンポーネントが担当します。

## アプリケーション名を作る

まず、黄色い枠の左側に当たる`app/features/header/components/AppLogo.tsx`を作成します。

```tsx:app/features/header/components/AppLogo.tsx
export default function AppLogo() {
  return (
    <span className="text-lg font-bold text-neutral-900">PocketPace</span>
  );
}
```

現段階のロゴは画像ではなく文字で表示します。`text-lg`で文字を大きくし、`font-bold`で太字にします。文字色には、Figmaの画面案に近い`text-neutral-900`を使用します。

`AppLogo`は表示だけを担当し、ヘッダー内の位置や周囲の余白は持ちません。配置は外側の`Header`で決めます。

## ユーザーアイコンを作る

次に、黄色い枠の右側に当たる`app/features/header/components/UserAvatar.tsx`を作成します。

```tsx:app/features/header/components/UserAvatar.tsx
import { Avatar, AvatarFallback } from "~/shared/ui/avatar";

export default function UserAvatar() {
  return (
    <Avatar
      aria-label="ユーザーアイコン"
      className="size-9 after:border-0"
      role="img"
    >
      <AvatarFallback className="bg-linear-to-br from-fuchsia-500 to-blue-500" />
    </Avatar>
  );
}
```

`Avatar`は、ユーザーアイコン全体の大きさや丸い形を管理する外側のコンポーネントです。その内側では、画像を表示する`AvatarImage`と、画像を表示できない場合の代替表示を担当する`AvatarFallback`を組み合わせられます。

現段階では利用者の画像を保存する機能がないため、`AvatarImage`は使用せず、`AvatarFallback`だけを置きます。一般的なユーザーアイコンでは、Fallbackへ利用者名の頭文字などを表示します。今回はFigmaの画面案にある色付きの円を再現するため、文字は渡さず、背景のグラデーションだけを表示しています。

`size-9`でアイコンの縦横を2.25remにします。Figmaの画面案に合わせ、代替表示には`bg-linear-to-br`で右下方向のグラデーションを付けています。`from-fuchsia-500`が開始色、`to-blue-500`が終了色です。Avatarが既定で表示する外周の線は`after:border-0`で外しました。

ユーザーアイコンを選択したときに表示する画面は、まだMVPの範囲に含めていません。そのため、現段階ではリンクやボタンにはせず、`role="img"`と`aria-label`を付けた表示要素にしています。設定画面を実装するときに、操作できるボタンへ変更します。

## Headerで二つの部品を組み合わせる

緑色の枠に当たる`app/features/header/Header.tsx`を作成し、`AppLogo`と`UserAvatar`を組み合わせます。

```tsx:app/features/header/Header.tsx
import AppLogo from "~/features/header/components/AppLogo";
import UserAvatar from "~/features/header/components/UserAvatar";

export default function Header() {
  return (
    <header className="h-[8dvh] bg-white">
      <div className="mx-auto flex h-full w-9/10 max-w-sm items-center justify-between">
        <AppLogo />
        <UserAvatar />
      </div>
    </header>
  );
}
```

外側の`header`は白い背景を担当します。`h-[8dvh]`では、ブラウザの表示領域に応じて変化する`dvh`を使い、ヘッダーの高さを画面高の8%にしています。

その内側は`h-full`でヘッダーと同じ高さまで広げます。横幅はフッターナビゲーションと同じく`w-9/10 max-w-sm`とし、スマートフォンでは画面幅の90%、広い画面では決めた最大幅に収めます。

`items-center`で二つの部品を縦方向の中央へ揃え、`justify-between`で`AppLogo`と`UserAvatar`を左右へ配置します。二つの部品は自身の位置を決めず、`Header`が全体のレイアウトを管理します。

## 共通レイアウトへHeaderを追加する

ヘッダーは四つのページすべてで表示するため、各ページではなく`app/routes/_app.tsx`へ追加します。

```diff tsx:app/routes/_app.tsx
 import { Outlet } from "react-router";
 import FooterNav from "~/features/footer/FooterNav";
+import Header from "~/features/header/Header";
 export default function AppLayout() {
   return (
     <>
-      <main className="min-h-dvh bg-slate-100 pb-[var(--floating-nav-clearance)]">
-        <Outlet />
-      </main>
+      <div className="min-h-dvh bg-slate-100">
+        <Header />
+        <main className="pb-[var(--floating-nav-clearance)]">
+          <Outlet />
+        </main>
+      </div>
       <FooterNav />
     </>
   );
 }
```

これまでは`main`だけで画面の最低高さと背景色を管理していました。ヘッダーを追加したあとも画面全体へ背景色が続くように、`Header`と`main`を囲む`div`へ`min-h-dvh bg-slate-100`を移します。

フッターナビゲーションは画面下部へ固定されているため、引き続き通常のレイアウトの外側へ置きます。内容との重なりを防ぐ`pb-[var(--floating-nav-clearance)]`も`main`へ残します。

変更後の`_app.tsx`は次のようになります。

```tsx:app/routes/_app.tsx
import { Outlet } from "react-router";

import FooterNav from "~/features/footer/FooterNav";
import Header from "~/features/header/Header";

export default function AppLayout() {
  return (
    <>
      <div className="min-h-dvh bg-slate-100">
        <Header />
        <main className="pb-[var(--floating-nav-clearance)]">
          <Outlet />
        </main>
      </div>
      <FooterNav />
    </>
  );
}
```

## 表示を確認する

開発サーバーを起動します。

```bash:ターミナル
pnpm dev
```

`/dashboard`を開き、画面上部の白い領域へ「PocketPace」とグラデーションのユーザーアイコンが表示されることを確認します。続いて、フッターナビゲーションから支出管理、収入管理、履歴管理へ移動し、どのページでも同じヘッダーが表示されることを確認します。

![完成した共通ヘッダー](/images/7464cf3f3b7b86/header-fin.png =410x)

あわせて、次の点も確認します。

- ヘッダーの内容が中央の最大幅に収まっている
- アプリケーション名とユーザーアイコンが左右に配置されている
- ヘッダーの下からページの背景色が切り替わっている
- フッターナビゲーションがこれまでどおり画面下部に表示される

最後に、型チェックと本番用ビルドを実行します。

```bash:ターミナル
pnpm typecheck
pnpm build
```

## まとめ

Feature専用の部品と共有する部品をどう分けるか考え、shadcn/uiの生成先を`shared/ui`へ変更しました。既存のButtonと`cn`も移動しましたが、今の分け方がずっと正解とは限らないので、使いにくくなったら見直します。

`AppLogo`と`UserAvatar`を`Header`で組み合わせ、shadcn/uiのAvatarを使ったヘッダーができました。`_app.tsx`へ置くと、4ページすべてに同じものが表示されました。

これで画面の上にヘッダー、下にフッターナビゲーション、その間にページ内容という枠ができました。まだ真ん中は空ですが、少しずつアプリらしくなってきました。次はダッシュボードへ予算情報を追加します。
