---
layout: post
title: FAQ
---
<!-- original:
We often get very similar questions across issues and PRs, and we figured it would be useful to list them here, for a reference.

## Replace an existing library with a different one

Sample issues: [#201](https://github.com/Polymer/pwa-starter-kit/issues/201), [#195](https://github.com/Polymer/pwa-starter-kit/pull/195)

We built pwa-starter kit (both the included templates and the sample apps) to be a good starting point for building a fairly complex PWA. This means that we had to make some decisions about which components, libraries and patterns to use -- these decisions were made around ease of use, popularity, and available documentation. There are many other options and libraries that are objectively good choices, and at the end of the day they all come down to a matter of personal preference. Since that’s not something that everyone can come to a consensus on, it’s unlikely that we will change an existing choice for a new one. You can, of course, replace any bits and pieces of pwa-starter-kit in _your_ application.

## Add example of using a library/framework

Sample issues: [#196](https://github.com/Polymer/pwa-starter-kit/issues/196), [#201](https://github.com/Polymer/pwa-starter-kit/issues/207)

pwa-starter-kit is not meant to replace any frameworks or application architecture patterns -- it’s meant to get you started towards building a complex PWA, but does not promise to build the entire thing for you. The templates are built in such a way that most JS libraries can be plugged in, since they don’t rely on a specific application structure. However, providing an example for each one of these patterns and libraries is a daunting task (and is generally applicable to a very small number of people).

If you’re looking for a good starting point of where to plug in a library, `store.js` is a good starting point -- it’s the place in the application that initializes Redux, which is a library that we plugged in, so your might need something similar.

## Can't add libraries not distributed as an ES module (ESM)

Sample issues: [#199](https://github.com/Polymer/pwa-starter-kit/issues/199)

Libraries must provide ES modules (ESM) - other module formats, such as UMD, `module.exports`, AMD, CommonJS, etc., are not compatible with Polymer tools. If the library's `pkg.main` is not already ESM, check if `package.json` defines `pkg.module` or `pkg[‘jsnext:main’]` - our tools will prefer those if present. Alternatively, you can import from `some-lib/src` if the source is written as ESM. If the library sets browser globals, you can reference them through the window object (e.g. `window.someLib`). Otherwise, you have to request ESM from the library author.

## I’m getting errors when running the tests

Sample issues: [#193](https://github.com/Polymer/pwa-starter-kit/issues/193)

The integration tests are fairly fragile, and require that you have the correct setup for the screenshot testing to match the expected output. When in doubt, test results from Travis CI should be considered as correct.
-->

Issuesやプルリクエストによく似た質問があるので、返答及び有益な情報となるようにのためにここにリストします。

## 現在のライブラリAを別のライブラリBに置き換えてほしい

同じようなIssues: [#201](https://github.com/Polymer/pwa-starter-kit/issues/201), [#195](https://github.com/Polymer/pwa-starter-kit/pull/195)

かなり複雑なPWAを構築するための出発点として、pwa-starterキット（付属のテンプレートとサンプルアプリの両方）を構築しました。 これは、使用するコンポーネント、ライブラリ、パターンについていくつかの明確な方向性を持たなければいけないと考えました。これらの方向性は、使いやすく、一般的で、入手可能なものを中心に議論されました。 客観的に良い選択肢である他の多くのオプションとライブラリがあり、最終的にはそれらはすべて個人的な好みの問題になります。そこに誰もがコンセンサスを得ることができないので、既存の選択肢を新しいものに変更することはまずありません。もちろん、あなたのアプリケーションでpwa-starter-kitを部分的に置き換えることはできます。

## 別のライブラリAやフレームワークBを使ったサンプルを追加してほしい

同じようなIssues: [#196](https://github.com/Polymer/pwa-starter-kit/issues/196), [#201](https://github.com/Polymer/pwa-starter-kit/issues/207)

pwa-starter-kitは、フレームワークやアプリケーションアーキテクチャのパターンを置き換えるものではありません。それはあなたが複雑なPWAを構築する出発点となるようにしますが、あなたのために全体を構築することをは約束できません。テンプレートは、特定のアプリケーション構造に依存しないため、ほとんどのJSライブラリをプラグインできるように構築されています。しかし、これらのパターンとライブラリのそれぞれの例を提供することは、大変な作業です。(一般的に非常に少数の人々に適用されます)。

あなたがライブラリのどこにプラグインするかの良い出発点を探しているなら、 `store.js`はよい出発点です。これはアプリケーションがReduxを初期化する場所です。これはプラグインされたライブラリであるため、同様のことができるでしょう。

## ESモジュール（ESM）として配布されていないライブラリが追加できない

同じようなIssue: [#199](https://github.com/Polymer/pwa-starter-kit/issues/199)

ライブラリはESモジュール（ESM）で提供されている必要があります。 UMD、module.exports、AMD、CommonJSなどの他のモジュールフォーマットは、Polymerのツールと互換性がありません。 もしライブラリの`pkg.main`がESMでないなら,`package.json`が`pkg.module`か`pkg[‘jsnext:main’]`を定義していないかチェックしてください。もしあるなら私たちのツールは適合するようにするでしょう。あるいは、ソースがESMとして書かれている場合は、 `some-lib/src`からインポートすることができます。 ライブラリがブラウザのグローバルを設定する場合は、ウィンドウオブジェクトから参照できます(例として`window.someLib`)。 それ以外の場合は、ライブラリ作成者にESMへの対応をお願いする必要があります。

## テストを実行するとエラーが発生します

同じようなIssues: [#193](https://github.com/Polymer/pwa-starter-kit/issues/193)

統合テストは非常にセンシティブで、期待される出力と正しく一致するようにスクリーンショットテストを設定する必要があります。不確かな場合は、Travis CIのテスト結果を正しいものとみなしてください。
