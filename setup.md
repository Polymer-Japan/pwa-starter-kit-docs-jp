---
layout: post
title: 設定方法
---
<!-- original:
This page will take you through the steps you need to do to build and run the sample app locally.

## Prerequisites
The app uses [Web Components](https://www.webcomponents.org/introduction), [lit-html](https://github.com/Polymer/lit-html), which is a small library for writing HTML templates with JavaScript string literals, and [lit-element](https://github.com/Polymer/lit-element), a small Web Component base class built on top of it.

This app depends on several [npm](https://www.npmjs.com/) packages, which you must be able to install. Make sure that you've already installed [node.js](https://nodejs.org/en/) and [npm](https://www.npmjs.com/) before moving on to the next step. If you already have `node` installed, make sure you're using the latest version -- we recommend using `v8.0.0` and above.

## Create a new app

To create a new app that uses the default `pwa-starter-kit` template:
```
git clone --depth 1 https://github.com/Polymer/pwa-starter-kit my-app
cd my-app
```

This will generate the initial project structure inside the `my-app` folder, which roughly looks like this:
```
my-app
├── images
|   └── ...
├── src
|   └── ...
├── test
|   └── ...
├── index.html
├── README.md
├── package.json
├── polymer.json
├── manifest.json
├── service-worker.js
├── sw-precache-config.js
├── ... (misc project config files)
```
Checkout the [folder structure]({{site.baseurl}}/configuring-and-personalizing#folder-structure) page for details on what each file is used for.


## Alternate templates

To create a new app based on one of the other templates listed in [Other templates]({{site.baseurl}}/overview#other-templates), you can clone the appropriate branch from the `pwa-starter-kit` repo:

```
git clone --depth 1 -b <template-name> --single-branch https://github.com/Polymer/pwa-starter-kit my-app
```

For example, to start from Typescript template (`template-typescript`):

```
git clone --depth 1 -b template-typescript --single-branch https://github.com/Polymer/pwa-starter-kit my-app
```

### Installing dependencies
To install the project's dependencies, run
```
npm install
```

You're now ready to run and see your app!

### Run the app in development mode
To run the app locally, run
```
npm start
```

This will start a local server on port `8081`. Open [http://localhost:8081](http://localhost:8081) to view your app in the browser. Note that this server can continue running as you're making changes to your application, which you will see if you refresh the browser tab.

If the port is already taken on your computer, or if you need to change the default hostname (because you're using a Docker container, for example), you can configure them using command line arguments:
```
npm start -- --hostname 0.0.0.0 --port 4444
```

### Run the tests
Check out the [Application testing]({{site.baseurl}}/application-testing) page for more information about the tests. For a quick way to run the tests, run
```
npm run test
```

## Available scripts
In the app's root directory you can run:
- `npm start` to run the application in development mode.
- `npm run test` to run the application's unit and integration tests (see the see the [testing section]({{site.baseurl}}/application-testing) for more details. To run just the unit or integration tests, both `npm run test:unit` and `npm run test:integration` are available.
- `npm run build` to build your application for production (see the [building and deploying]({{site.baseurl}}/building-and-deploying) section for more details).
- `npm run serve:static` or `npm run serve:prpl-server` to serve the built application (see the [building and deploying]({{site.baseurl}}/building-and-deploying) section for more details).

The complete list of scripts can be found in the [`package.json`](https://github.com/Polymer/pwa-starter-kit/blob/master/package.json#L10) file.

## Browser support
`pwa-starter-kit` uses fairly recent browsers APIs, that might not be natively available on all of the browsers you are supporting. To get around that, the app relies on polyfills, to add the missing web platform features to some browsers, as well as a build step, to add newer JavaScript features to browsers that don't support them (such as transpiling ES6 to ES5 for browsers like IE11, or dynamic module imports). Check out the [Browser Support]({{site.baseurl}}/browser-support) page for more details.

## Next steps
Now that you're done with the basics of running your app, check out the next steps:
- [Configuring and personalizing]({{site.baseurl}}/configuring-and-personalizing) the app by modifying and adding your own content
- [Building and deploying]({{site.baseurl}}/building-and-deploying) to production
-->

このページでは、サンプルアプリケーションをローカルにビルドして実行するために必要な手順について説明します。

## 前提条件
アプリケーションはHTMLを書くための小さなライブラリである[Webコンポーネント](https://www.webcomponents.org/introduction)、JavaScript文字列リテラルを持つテンプレート[lit-html](https://github.com/Polymer/lit-html)、またその上に構築された小さなWebコンポーネント基本クラスである[lit-element](https://github.com/Polymer/lit-element)を使用します。

このアプリはいくつかの[npm](https://www.npmjs.com/)パッケージに依存していてます。次の手順に進む前に、[node.js](https://nodejs.org/en/)と[npm](https://www.npmjs.com/)が既にインストールされていることを確認してください。既に `node`がインストールされている場合は、最新のバージョンを使用していることを確認してください.v8.0.0以上を使用することをお勧めします。

## 新しいアプリを作成する

デフォルトの `pwa-starter-kit`テンプレートを使用する新しいアプリケーションを作成するには:
```
git clone --depth 1 https://github.com/Polymer/pwa-starter-kit my-app
cd my-app
```

これは、 現在のディレクトリに`my-app`初期プロジェクトを生成します。これはおおよそ次のようになります:

```
my-app
├── images
|   └── ...
├── src
|   └── ...
├── test
|   └── ...
├── index.html
├── README.md
├── package.json
├── polymer.json
├── manifest.json
├── service-worker.js
├── sw-precache-config.js
├── ... (misc project config files)
```

各ファイルの使用方法の詳細については、[ディレクトリ構造]({{site.baseurl}}/configuring-and-personalizing#folder-structure)のページをご覧ください。

## 他のテンプレート

[他のテンプレート]({{site.baseurl}}/overview#other-templates)にリストされている他のテンプレートのいずれかに基づいて新しいアプリケーションを作成するには、 `pwa-starter-kit`リポジトリから該当のブランチをクローンして使用します:

```
git clone --depth 1 -b <template-name> --single-branch https://github.com/Polymer/pwa-starter-kit my-app
```

たとえば、Typescriptテンプレートから開始するには (`template-typescript`):

```
git clone --depth 1 -b template-typescript --single-branch https://github.com/Polymer/pwa-starter-kit my-app
```

### 依存関係のインストール

プロジェクトの依存関係をインストールするには、

```
npm install
```

これでもう今すぐアプリを実行して、見る準備が整いました!

### 開発モードでアプリケーションを実行する

アプリをローカルで実行するには、

```
npm start
```

これはポート8081でローカルサーバを起動します。[http://localhost:8081](http://localhost:8081)を開き、ブラウザでアプリケーションを表示します。このサーバーは、アプリケーションを変更している間も実行を続けることができ、ブラウザのタブを更新して表示を更新します。

8080ポートが既にコンピュータ上で使用されている場合、またはデフォルトのホスト名を変更する必要がある場合（たとえば、Dockerコンテナを使用しているため）、コマンドライン引数を使用してポートを設定できます:

```
npm start -- --hostname 0.0.0.0 --port 4444
```

### テストを実行する

テストの詳細については、[Application testing]({{site.baseurl}}/application-testing)のページを参照してください。テストをすぐに実行するには、

```
npm run test
```

## 使用可能なスクリプト

アプリのルートディレクトリで:
- `npm start` 開発モードでアプリケーションを実行します。
- `npm run test` アプリケーションのユニットと統合テストを実行します（詳細は[テストの章]({{site.baseurl}}/application-testing)を参照）。ユニットテストまたは統合テストだけを実行するには、 `npm run test:unit`と` npm run test:integration`が利用できます。
- `npm run build` 本番公開用のアプリケーションをビルドします (詳細は[ビルドとデプロイ]({{site.baseurl}}/building-and-deploying)の章を参照)。
- `npm run serve:static` もしくは `npm run serve:prpl-server` ビルドされたアプリケーションを実行します (詳細は[ビルドとデプロイ]({{site.baseurl}}/building-and-deploying)を参照。

スクリプトの完全なリストは、[`package.json`](https://github.com/Polymer/pwa-starter-kit/blob/master/package.json#L10)ファイルにあります。

## ブラウザサポート

`pwa-starter-kit`はすべてのブラウザでネイティブにサポートされていない、かなり最近のブラウザAPIを使用します。これを回避するために、アプリはポリフィルを使用し、足りないWebプラットフォームの機能をいくつかのブラウザに追加するだけでなく、新しいJavaScript機能をサポートしていないブラウザに新しいJavaScript機能を追加する必要があります（例えばIE11などのブラウザ用のES6からES5変換や動的モジュールのインポート）。詳細については、[ブラウザサポート]({{site.baseurl}}/browser-support)のページをご覧ください。

## 次のステップ

これで、アプリの基本的な操作が完了したので、次のステップを確認してください:

- [カスタマイズとパーソナライズ]({{site.baseurl}}/configuring-and-personalizing) アプリにWebのコンテンツを追加します
- [ビルドとデプロイ]({{site.baseurl}}/building-and-deploying) 本番公開用
