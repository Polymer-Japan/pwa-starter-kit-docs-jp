---
layout: post
title: "概要: 何が含まれているか"
---
<!-- original:
The `pwa-starter-kit` is a set of templates you can use as a starting point for building PWAs. Out of the box, the default template gives you the following features:

- All the PWA goodness (manifest, service worker).
- A responsive layout.
- Application theming.
- Example of using Redux for state management.
- Offline UI.
- Simple routing solution.
- Fast time-to-interactive and first-paint using the PRPL pattern.
- Easy deployment to prpl-server or static hosting.
- Unit and integration testing starting points.
- Documentation about other advanced patterns.

## Other templates

If you already know what you're doing and want a different template to start from, we've created several separate templates, with different sets of features:

### `template-typescript` ([code](https://github.com/Polymer/pwa-starter-kit/tree/template-typescript), [demo](https://pwa-starter-kit.appspot.com/))

This template is the same as the `master` template, but implemented using `TypeScript`.

### `template-minimal-ui` ([code](https://github.com/Polymer/pwa-starter-kit/tree/template-minimal-ui), [demo](https://template-minimal-ui-dot-pwa-starter-kit.appspot.com/))

This template uses Redux for state management like the `master` template, but doesn't use any of the `app-layout` elements (`app-header` or `app-drawer`) for the responsive UI.

### `template-no-redux` ([code](https://github.com/Polymer/pwa-starter-kit/tree/template-no-redux), [demo](https://template-no-redux-dot-pwa-starter-kit.appspot.com/))

This template has the same UI elements as the `master` one (`app-layout` elements, theming, etc), but does _not_ use Redux for state management. Instead, it does a properties-down-events-up unidirectional data flow approach, where the data source of truth is mutable, and individual elements (specifically, each view) own parts of the entire application state.

### `template-responsive-drawer-layout` ([code](https://github.com/Polymer/pwa-starter-kit/tree/template-responsive-drawer-layout), [demo](https://template-responsive-drawer-layout-dot-pwa-starter-kit.appspot.com/))

This template is very similar to the `master` template, in the sense that it keeps both Redux for state management, and all of the UI elements. The main difference is that the wide screen layout displays a persistent `app-drawer`, inline with the content.

## `pwa-helpers`
A lot of the reusable functionality of `pwa-starter-kit` has already been pulled out as helper methods, into a separate repo. The [`pwa-helpers`](https://github.com/Polymer/pwa-helpers) repo contains:

- `router.js`: A very basic router that calls a callback any time the location changes.
- `network.js`: Calls a callback whenever the network connectivity of the app changes.
- `metadata.js`: Utility method that sets the Twitter card/open graph metadata for a specific page.
- `media-query.js`: Calls a callback whenever a media-query matches in response to the viewport size changing.
- `connect-mixin.js`: Small mixin that you can add to a Custom Element base class to automatically connect to a Redux store.

Each of these helpers is very small, and can be implemented in many different, bespoke ways. However, they each represent a feature that is often needed across many different applications, so unless you already have a solution planned for your app, you could use one of these.
-->

`pwa-starter-kit`は、PWAを構築するための出発点として使用できる一連のテンプレートです。デフォルトのテンプレートをそのまま使用すると、次の機能が提供されます:

- PWAの全ての良さ（マニフェスト、サービスワーカー）
- レスポンシブレイアウト
- アプリケーションテーマ
- 状態管理にReduxを使用する例
- オフラインUI
- シンプルなルーティングソリューション
- PRPLパターンを使用してインタラクティブ・ファースト・ペイントまでの時間が短縮されます
- prpl-serverまたは静的ホスティングへの簡単なデプロイ
- ユニットと統合テストの出発点
- その他の高度なパターンに関するドキュメント
 
## 他のテンプレート

あなたが何をやっているのかが分かっていて、別のテンプレートを使いたいと思っているならば、いくつかのテンプレートが用意されています:

### `template-typescript` ([code](https://github.com/Polymer/pwa-starter-kit/tree/template-typescript), [demo](https://pwa-starter-kit.appspot.com/))

このテンプレートは `master`テンプレートと同じですが、` TypeScript`を使って実装されています。

### `template-minimal-ui` ([code](https://github.com/Polymer/pwa-starter-kit/tree/template-minimal-ui), [demo](https://template-minimal-ui-dot-pwa-starter-kit.appspot.com/))

このテンプレートは `master`テンプレートのような状態管理のためにReduxを使いますが、レスポンシブUIには` app-layout`要素（ `app-header`や` app-drawer`）を使いません。

### `template-no-redux` ([code](https://github.com/Polymer/pwa-starter-kit/tree/template-no-redux), [demo](https://template-no-redux-dot-pwa-starter-kit.appspot.com/))

このテンプレートは `master`と同じUI要素（` app-layout`要素、テーマなど）を持ちますが、状態管理のためにReduxを使用しません。代わりに、データソースが変更可能で、個々の要素（具体的には、各ビュー）がアプリケーション状態全体はプロパティーダウンイベントアップ単方向データフローアプローチとなります。
### `template-responsive-drawer-layout` ([code](https://github.com/Polymer/pwa-starter-kit/tree/template-responsive-drawer-layout), [demo](https://template-responsive-drawer-layout-dot-pwa-starter-kit.appspot.com/))

このテンプレートは `master`テンプレートと非常によく似ています。つまり、状態管理のためにReduxとすべてのUI要素を保持しているという意味でです。主な違いは、ワイド画面レイアウトでは、内容とインラインで'app-drawer`が表示されることです。

## `pwa-helpers`

`pwa-starter-kit`の再利用可能な機能の多くは、すでに別のリポジトリにヘルパーメソッドとして提供されています。[`pwa-helpers`](https://github.com/Polymer/pwa-helpers)には:

- `router.js`: ページが変わるたびにコールバックを呼び出す非常に基本的なルータ
- `network.js`: アプリケーションのネットワーク接続が変更されるたびに呼び出されるコールバックライブラリ
- `metadata.js`: Twitter CardやOGP用のメタデータを設定するユーティリティ
- `media-query.js`: viewportのサイズが変化したときに、メディアクエリが一致するたびに呼びだされるコールバックライブラリ
- `connect-mixin.js`: カスタム要素基本クラスに追加して、Reduxストアに自動的に接続できる小さなミックスイン。

これらのヘルパーはそれぞれ非常に小さく、さまざまな方法で実装できます。しかし、それらはそれぞれ異なるアプリケーションで必要とされる機能を持っています。アプリ用まだ実装を用意していない場合は、これらのいずれかを使用できます。
