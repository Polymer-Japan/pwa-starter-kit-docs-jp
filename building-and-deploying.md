---
layout: post
title: ビルドとデプロイ
---
<!-- original:
This page will take you through the steps you need to do to build and deploy your application to production.

## Table of Contents
- [prpl-server (recommended)](#prpl-server-recommended)
  - [Building for prpl-server](#building-for-prpl-server)
  - [Previewing prpl-server](#previewing-prpl-server)
  - [Deploying prpl-server](#deploying-prpl-server)
    - [App Engine](#app-engine)
    - [Firebase Hosting + Firebase Functions](#firebase-hosting--firebase-functions)
- [Static hosting](#static-hosting)
  - [Building for static hosting](#building-for-static-hosting)
  - [Previewing static hosting](#previewing-static-hosting)
  - [Deploying static hosting](#deploying-static-hosting)
    - [App Engine](#app-engine-1)
    - [Firebase Hosting](#firebase-hosting)
    - [Netlify](#netlify)
- [Service Worker](#service-worker)

## `prpl-server` (recommended)
[prpl-server](https://github.com/Polymer/prpl-server-node) is a node server that uses differential serving to deliver the optimal response for each browser. Our provided configuration ([`polymer.json`](https://github.com/Polymer/pwa-starter-kit/blob/master/polymer.json)) supports the following responses:

- `esm-bundled` - Bundled ES modules for browsers that support ES modules
- `es6-bundled` - Bundled ES6/2015 code using [AMD](http://requirejs.org/docs/whyamd.html) for other browsers that support ES6/2015
- `es5-bundled` - Bundled ES5 code using [AMD](http://requirejs.org/docs/whyamd.html) for other browsers
- Server-side rendered pages (with [Rendertron](https://github.com/GoogleChrome/rendertron)) for supported bots/web crawlers

### Building for `prpl-server`
To run the build:

```
npm run build
```

This will populate the `server/build/` directory:

```
server/
├── build/
|   └── es5-bundled/
|   └── es6-bundled/
|   └── esm-bundled/
|   └── polymer.json
├── app.yaml
├── package-lock.json
└── package.json
```

### Previewing `prpl-server`
To preview the build using prpl-server locally:

```
npm run serve
```

### Deploying `prpl-server`
After building, the contents of `server/` contains all the files and configuration necessary to run the app in production. The provided `server/package.json` specifies server dependencies and the start command which can be used on almost any hosting service that supports Node.js.

#### App Engine

##### Flexible Environment
The contents of `server/app.yaml` is pre-configured to be deployed to [Google App Engine Node.js Flexible Environment](https://cloud.google.com/appengine/docs/flexible/nodejs/). Use the `gcloud` tool to deploy the contents of `server/` (e.g. `gcloud app deploy server/app.yaml`).

##### Standard Environment (Beta)
To deploy to [Google App Engine Node.js Standard Environment (Beta)](https://cloud.google.com/appengine/docs/standard/nodejs/), replace the entire contents of `server/app.yaml` with:

```yaml
runtime: nodejs8
```

Use the `gcloud` tool to deploy the contents of `server/` (e.g. `gcloud app deploy server/app.yaml`).

#### Firebase Hosting + Firebase Functions
_Firebase Hosting_ alone is not sufficient for hosting the `prpl-server` build since it requires some server-side processing of the user agent string. Instead, you will have to use `Firebase Functions` for server-side processing. [This gist](https://gist.github.com/Dabolus/314bd939959ebe68f57f1dcebe120a7e) contains detailed instructions on how to accomplish this.

## Static hosting
If you don't need differential serving and want to serve the same build to all browsers, you can just deploy to a static server.

### Building for static hosting
To build the production site, run:

```
npm run build:static
```

This will create three different build outputs:

```
build/
├── es5-bundled/
├── es6-bundled/
├── esm-bundled/
└── ...
```

- `esm-bundled` - Bundled ES modules for browsers that support ES modules
- `es6-bundled` - Bundled ES6/2015 code using [AMD](http://requirejs.org/docs/whyamd.html) for other browsers that support ES6/2015
- `es5-bundled` - Bundled ES5 code using [AMD](http://requirejs.org/docs/whyamd.html) for other browsers

### Previewing static hosting
To preview it locally, run:

```
npm run serve:static
```

Our provided configuration will serve the `es5-bundled` build. If you don't need to support legacy browsers, you can use a more modern build by modifying the `serve:static` script in package.json to use `es6-bundled` or `esm-bundled` instead. Be sure that all page navigation requests are served the contents of `index.html`.

### Deploying static hosting
By default, static hosting servers aren't set up to work with single page apps (SPAs) -- in particular, the problem is that an SPA uses routes that do not correspond to full file path names. For example, in `pwa-starter-kit` the second view's URL is `http://localhost:8081/view2`, but that doesn't correspond to a file that the browser can use. Each static hosting server has a different approach to working around this:

#### App Engine
Download the [Google App Engine SDK](https://cloud.google.com/appengine/downloads) and follow the instructions for your platform to install it. Here we are using Python SDK.

[Sign up for an App Engine account](https://cloud.google.com/appengine) and go to [project dashboard](https://pantheon.corp.google.com/cloud-resource-manager) page to create a new project. Make note of the project ID associated with your project.

Create an App Engine config file (`app.yaml`) with the following:

```yaml
runtime: python27
api_version: 1
threadsafe: yes

handlers:

- url: /images
  static_dir: build/es5-bundled/images
  secure: always

- url: /node_modules
  static_dir: build/es5-bundled/node_modules
  secure: always

- url: /src
  static_dir: build/es5-bundled/src
  secure: always

- url: /manifest.json
  static_files: build/es5-bundled/manifest.json
  upload: build/es5-bundled/manifest.json
  secure: always

- url: /service-worker.js
  static_files: build/es5-bundled/service-worker.js
  upload: build/es5-bundled/service-worker.js
  secure: always

- url: /.*
  static_files: build/es5-bundled/index.html
  upload: build/es5-bundled/index.html
  secure: always

skip_files:
- build/es6-bundled/
- build/esm-bundled/
- images/
- node_modules/
- server/
- src/
- test/
```

To deploy your project:
```
gcloud app deploy --project <project_ID>
```

#### Firebase Hosting
[Firebase](https://firebase.google.com/docs/hosting/) provides easy http2-enabled static hosting, a real-time database, server functions, and edge-caching all over the globe.

Install the Firebase CLI:
```
npm install -g firebase-tools
```
[Sign up for a Firebase account](https://www.firebase.com/signup/) if you don't have one. Then go to [Firebase Console](https://www.firebase.com/) to create a new project. Make note of the project ID associated with your project.

Login to the Firebase and set the previously created project as the active Firebase project for your working directory:
```
firebase login
firebase use <project_ID>
```

Create a Firebase config file (`firebase.json`) with the following:

```json
{
  "hosting": {
    "public": "build/es5-bundled/",
    "rewrites": [
      {
        "source": "**/!(*.*)",
        "destination": "/index.html"
      }
    ],
     "headers": [
      {
        "source":"/service-worker.js",
        "headers": [
          {
            "key": "Cache-Control",
            "value": "no-cache"
          }
        ]
      }
    ]
  }
}
```

To deploy your project:
```
firebase deploy
```

#### Netlify
[Netlify](https://www.netlify.com/) has built-in [Continuous Deployment](https://www.netlify.com/docs/continuous-deployment/) which automatically runs your build commands and deploys the result whenever you push to your Git repository.

Create a `_redirects` file with the following [rewrite rules](https://www.netlify.com/docs/redirects/):
```
/*    /index.html   200
```
Go to [netlify project](https://app.netlify.com/signup) page and setup the Git hosting for the new project. In `Basic build settings`, put `npm run build:static` as the build command and `build/es5-bundled` as the publish directory.

Click `Deploy site`.

## Service Worker
A Service Worker is loaded and registered in the [`index.html`](https://github.com/Polymer/pwa-starter-kit/blob/master/index.html#L68) file. However, during development (to make debugging easier), the Service Worker does not actually exist, and only a [stub](https://github.com/Polymer/pwa-starter-kit/blob/master/service-worker.js) file is used.

The production time Service Worker is automatically created during build time, i.e. by running `npm run build` or `npm run build:static`. This file is generated based on the [`polymer.json`](https://github.com/Polymer/pwa-starter-kit/blob/master/polymer.json) configuration file, and you can find it under each of the build directories:

```
build/
├── es5-bundled/
|   └── service-worker.js
├── es6-bundled/
|   └── service-worker.js
├── esm-bundled/
|   └── service-worker.js
└── ...
```

By default, all of the source files (inside the `/src` directory) will be pre-cached, as specified in the [`sw-precache-config.js`](https://github.com/Polymer/pwa-starter-kit/blob/master/sw-precache-config.js) configuration file. If you want to change this behaviour, check out the [`sw-precache-config` docs](https://www.polymer-project.org/3.0/toolbox/service-worker).
-->

このページでは、アプリケーションを作成して本番環境に展開するために必要な手順について説明します。

## 目次
- [prpl-server (推奨)](#prpl-server-recommended)
  - [prpl-server用のビルド](#building-for-prpl-server)
  - [prpl-serverのプレビュー](#previewing-prpl-server)
  - [prpl-serverのデプロイ](#deploying-prpl-server)
    - [App Engine](#app-engine)
    - [Firebase Hosting + Firebase Functions](#firebase-hosting--firebase-functions)
- [静的ホスティング](#static-hosting)
  - [静的ホスティングのためのビルド](#building-for-static-hosting)
  - [静的ホスティングのプレビュー](#previewing-static-hosting)
  - [静的ホスティングのデプロイ](#deploying-static-hosting)
    - [App Engine](#app-engine-1)
    - [Firebase Hosting](#firebase-hosting)
    - [Netlify](#netlify)
- [Service Worker](#service-worker)

<a id="prpl-server-recommended">

## `prpl-server` (推奨)

[prpl-server](https://github.com/Polymer/prpl-server-node)は、各ブラウザに対して最適に配信するために振り分け機能を提供するノードサーバーです。
[`polymer.json`](https://github.com/Polymer/pwa-starter-kit/blob/master/polymer.json)の設定では、以下の応答をサポートします:

- `esm-bundled` - ESモジュールをサポートするブラウザ用のバンドルされたESモジュール
- `es6-bundled` - ES6/2015をサポートする他のブラウザ用に[AMD](http://requirejs.org/docs/whyamd.html)を使用してバンドルされたES6/2015コード
- `es5-bundled` - 他のブラウザ用に[AMD](http://requirejs.org/docs/whyamd.html)を使用してバンドルされたES5コード
- サーバサイドレンダリングをWebのbotやクローラ向けに([Rendertron](https://github.com/GoogleChrome/rendertron)によって)


<a id="building-for-prpl-server">

### `prpl-server`のためのビルド

ビルドを実行するには:

```
npm run build:prpl-server
```

これで `server/build/`ディレクトリが生成されます:

```
server/
├── build/
|   └── es5-bundled/
|   └── es6-bundled/
|   └── esm-bundled/
|   └── polymer.json
├── app.yaml
├── package-lock.json
└── package.json
```

<a id="previewing-prpl-server">

### `prpl-server`のプレビュー

prpl-serverをローカルで使用してビルドをプレビューするには:

```
npm run serve:prpl-server
```

<a id="deploying-prpl-server">

### `prpl-server`のデプロイ

構築後、 `server/`の中には、本番環境でアプリケーションを実行するために必要なすべてのファイルと設定が含まれています。
提供されている `server/package.json`は、サーバーの依存関係と、Node.jsをサポートするほぼすべてのホスティングサービスで使用できるstartコマンドを指定されています。

<a id="app-engine">

#### App Engine

##### フレキシブル環境

The contents of `server/app.yaml` is pre-configured to be deployed to [Google App Engine Node.js Flexible Environment](https://cloud.google.com/appengine/docs/flexible/nodejs/). Use the `gcloud` tool to deploy the contents of `server/` (e.g. `gcloud app deploy server/app.yaml`).

`server/app.yaml`のコンテンツは[Google App Engine Node.jsフレキシブル環境](https://cloud.google.com/appengine/docs/flexible/nodejs/)にデプロイできるように事前設定されています。 `gcloud`ツールを使用して` server/`の内容をデプロイします（例: `gcloud app deploy server/app.yaml`）。

##### スタンダード環境 (ベータ)

[Google App Engine Node.js スタンダード環境（Beta）](https://cloud.google.com/appengine/docs/standard/nodejs/)にデプロイするには、 `server/app.yaml`の下記で置き換えます:

```yaml
runtime: nodejs8
```

`gcloud`ツールを使用して`server/`の内容をデプロイします (例: `gcloud app deploy server/app.yaml`).

<a id="firebase-hosting--firebase-functions">

#### Firebase Hosting + Firebase Functions

_Firebase Hosting_だけでは、ユーザエージェント文字列のサーバサイド処理が必要なため、 `prpl-server`ビルドのホスティングには不十分です。代わりに、サーバ側の処理に `Firebase Functions`を使わなければなりません。 [このgist](https://gist.github.com/Dabolus/314bd939959ebe68f57f1dcebe120a7e)には、これを実施するための詳細な手順が記載されています。

<a id="static-hosting">

## 静的ホスティング

ブラウザ環境別に分けた配信が不要で、すべてのブラウザに同じビルドを提供したい場合は、静的なサーバーに展開するだけで済みます。

<a id="building-for-static-hosting">

### 静的ホスティングのためのビルド

本番サイトを構築するには、:

```
npm run build:static
```

これにより、3つの異なるビルド出力が作成されます:

```
build/
├── es5-bundled/
├── es6-bundled/
├── esm-bundled/
└── ...
```

- `esm-bundled` - ESモジュールをサポートするブラウザ用のバンドルされたESモジュール
- `es6-bundled` - ES6/2015をサポートする他のブラウザ用に[AMD](http://requirejs.org/docs/whyamd.html)を使用してバンドルされたES6/2015コード
- `es5-bundled` - 他のブラウザ用に[AMD](http://requirejs.org/docs/whyamd.html)を使用してバンドルされたES5コード

<a id="previewing-static-hosting">
### 静的ホスティングのプレビュー

ローカルでプレビューするには:

```
npm run serve:static
```

デフォルトの設定は `es5-bundled`ビルドを使います。レガシーブラウザをサポートする必要がない場合は、package.jsonの `serve:static`スクリプトを` es6-bundled`または `esm-bundled`を使うように修正することで、より現代的なビルドを使用することができます。すべてのページナビゲーションリクエストが `index.html`の内容を提供していることを確認してください。

<a id="deploying-static-hosting">

### 静的ホスティングのデプロイ

By default, static hosting servers aren't set up to work with single page apps (SPAs) -- in particular, the problem is that an SPA uses routes that do not correspond to full file path names. For example, in `pwa-starter-kit` the second view's URL is `http://localhost:8081/view2`, but that doesn't correspond to a file that the browser can use. Each static hosting server has a different approach to working around this


デフォルトでは、スタティックホスティングサーバーはシングルページアプリケーション(SPA)で動作するように設定されていません。特に、SPAが完全なファイルパス名に対応しないルートを使用するという問題があります。たとえば、 `pwa-starter-kit`の2番目のビューのURLは`http://localhost:8081/view2`ですが、ブラウザが使用できるファイルには対応していません。各静的ホスティングサーバーには、これを回避するための異なるアプローチを持っています:

<a id="app-engine-1">
#### App Engine

[Google App Engine SDK](https://cloud.google.com/appengine/downloads)をダウンロードし、ご使用のプラットフォームの手順に従ってインストールしてください。ここでは、Python SDKを使用しています。

[App Engineにログイン](https://cloud.google.com/appengine)し、[プロジェクトダッシュボード](https://pantheon.corp.google.com/cloud-resource-manager)から新規プロジェクトを作ります。 プロジェクトに関連付けられているプロジェクトIDをメモしておきます。

App Engine設定ファイル（`app.yaml`）を次のように作成します:

```yaml
runtime: python27
api_version: 1
threadsafe: yes

handlers:

- url: /images
  static_dir: build/es5-bundled/images
  secure: always

- url: /node_modules
  static_dir: build/es5-bundled/node_modules
  secure: always

- url: /src
  static_dir: build/es5-bundled/src
  secure: always

- url: /manifest.json
  static_files: build/es5-bundled/manifest.json
  upload: build/es5-bundled/manifest.json
  secure: always

- url: /service-worker.js
  static_files: build/es5-bundled/service-worker.js
  upload: build/es5-bundled/service-worker.js
  secure: always

- url: /.*
  static_files: build/es5-bundled/index.html
  upload: build/es5-bundled/index.html
  secure: always

skip_files:
- build/es6-bundled/
- build/esm-bundled/
- images/
- node_modules/
- server/
- src/
- test/
```

プロジェクトをデプロイするには:
```
gcloud app deploy --project <project_ID>
```

<a id="firebase-hosting">

#### Firebase Hosting

[Firebase](https://firebase.google.com/docs/hosting/)は、世界中に配信できる簡単なHTTP2対応静的ホスティング、リアルタイムデータベース、サーバー機能、エッジキャッシングを提供しています。

Firebase CLIをインストール:
```
npm install -g firebase-tools
```

[Firebaseにログイン](https://www.firebase.com/signup/)します。[Firebaseコンソール](https://www.firebase.com/)で新規プロジェクトを作成し、プロジェクトに関連付けられているプロジェクトIDをメモしておきます。

Firebaseにログインし、作成したプロジェクトを作業ディレクトリ用のアクティブなFirebaseプロジェクトとして設定します:
```
firebase login
firebase use <project_ID>
```

Firebase設定ファイル（ `firebase.json`）を以下のように作成します:

```json
{
  "hosting": {
    "public": "build/es5-bundled/",
    "rewrites": [
      {
        "source": "**/!(*.*)",
        "destination": "/index.html"
      }
    ],
     "headers": [
      {
        "source":"/service-worker.js",
        "headers": [
          {
            "key": "Cache-Control",
            "value": "no-cache"
          }
        ]
      }
    ]
  }
}
```

プロジェクトをデプロイするには:
```
firebase deploy
```

<a id="netlify">

#### Netlify
[Netlify](https://www.netlify.com/)は[継続的デプロイ](https://www.netlify.com/docs/continuous-deployment/)がビルトインされ、ビルドコマンドを自動的に実行し、Gitリポジトリにプッシュするたびに結果をデプロイします。

次の[書き換えルール](https://www.netlify.com/docs/redirects/)を使って `_redirects`ファイルを作成します:

```
/*    /index.html   200
```

[netlify](https://app.netlify.com/signup)に移動し、新しいプロジェクトのGitホスティング先を設定します。`Basic build settings`では、ビルドコマンドとして` npm run build:static`を、パブリッシュディレクトリとして `build/es5-bundled`を置きます。

`Deploy site`をクリックします。


<a id="service-worker">
## Service Worker

Service Workerが[`index.html`](https://github.com/Polymer/pwa-starter-kit/blob/master/index.html#L68)でロードされ、登録されます。しかし、開発中（デバッグを簡単にするため）は Service Workerは実際には存在せず、[空ファイル](https://github.com/Polymer/pwa-starter-kit/blob/master/service-worker.js)のみが存在します。

Service Workerの設定はビルド時、つまり `npm run build:static`または`npm run build:prpl-server`を実行することによって自動的に作成されます。ビルドディレクトリのこれらのファイルは、[`polymer.json`](https://github.com/Polymer/pwa-starter-kit/blob/master/polymer.json)の設定ファイルに基づいて生成されます:

```
build/
├── es5-bundled/
|   └── service-worker.js
├── es6-bundled/
|   └── service-worker.js
├── esm-bundled/
|   └── service-worker.js
└── ...
```

デフォルトでは、 `/src`ディレクトリ内のすべてのソースファイルは、[`sw-precache-config.js`](https://github.com/Polymer/pwa-starter-kit/blob/master/sw-precache-config.js)で指定された通りにプリキャッシュされます。この動作を変更したい場合は、[`sw-precache-config`ドキュメント](https://www.polymer-project.org/3.0/toolbox/service-worker)を参照してください。
