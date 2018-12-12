---
layout: post
title: アプリケーションテスト
---
<!-- original:
It is good to test your apps. This page will take you through the testing setup we have provided.

## Running tests
There are many ways to test your app, and many frameworks in which you can write your tests. We provided samples for both unit tests and integration tests below -- the integration tests use [Puppeteer](https://github.com/GoogleChrome/puppeteer), and the unit tests use [WCT](https://github.com/Polymer/web-component-tester). As you modify and the customize your app, make sure you also update the tests :)

The test folder has the following structure:
```
test
├── integration
|   └── screenshots-baseline
|   └── router.js
|   └── visual.js
├── unit
|   └── index.html
|   └── counter-element.js
|   └── views-a11y.js
```
Where
- `screenshots-baseline` is the golden ("correct") set of screenshots for the app, as used in visual testing.
- `router.js` tests that the router is working correctly, and clicking on the nav links actually does a navigation.
- `visual.js` does a visual diffing of what your application currently looks like, and what it _should_ look like according to the screenshots in `screenshots-baseline`.
- `counter-element.js` is a WCT unit test that tests the simple counter element used in the first Redux demo.
- `views-a11y.js` is a WCT unit test that uses [axe-core](https://github.com/dequelabs/axe-core) to test that each of the application's views are accessible.

You can run the entire test suite via
```
npm test
```

Or a particular test suite via
```
npm run test:unit
npm run test:integration
```

## Unit testing

### Basic testing
We use [WCT](https://github.com/Polymer/web-component-tester) to run unit tests. WCT comes pre-packaged with `<test-fixture>`, an element that defines a template of content and copies a clean, new instance of that content into each test suite (more information [here](https://www.polymer-project.org/3.0/docs/tools/tests#test-fixtures)). To add a new unit test to the suite:
- Create a new file under `test/unit/` (or just copy and rename `test/unit/counter-element` for a starting point.
- Add the test to the WCT suite, in `test/unit/index.html`:

```js
WCT.loadSuites([
  ...
  // Load 'my-new-test.html' test suite, using native shadow dom:
  'my-new-test.html',
  // Load 'my-new-test.html' test suite, using shadydom
  'my-new-test.html?wc-shadydom=true&wc-ce=true',
]);
```
- Once you've added the new tests to this test suite, you can run the unit tests via
```
npm run test:unit
```

By default, the tests will be run on all of your local browsers (note that if you're on a Mac OS machine, you might have to [enable remote automation](https://webkit.org/blog/6900/webdriver-support-in-safari-10/) in Safari from the "Develop" menu for this to work correctly). If you want to configure the browsers used for testing, you can use the `-l` command line argument:

```
npm run test:unit -l chrome -l firefox
```

For more information on writing unit tests with WCT, check out the [testing documentation](https://www.polymer-project.org/3.0/docs/tools/tests#overview) or the [WCT documentation](https://github.com/Polymer/tools/tree/master/packages/web-component-tester#test-fixture).

### A11y testing
[Axe-core](https://github.com/dequelabs/axe-core) is a library that automatically audits your HTML for accessibility violations. In order to use this more easily inside of unit tests, we've created a small wrapper, [`axe-report.js`](https://github.com/Polymer/pwa-helpers#axe-reportjs), that returns an `Error` containing all the violations. You can use this to unit test a [specific element](https://github.com/Polymer/pwa-starter-kit/blob/master/test/unit/counter-element.html#L70) or a [whole page](https://github.com/Polymer/pwa-starter-kit/blob/master/test/unit/views-a11y.html) as well. If you already have a `WCT` unit test set up to test the functionality of an element `el`, then you can also add an a11y test for it via:
```js
import {axeReport} from '../axe-report.js';
suite('my-element tests', function() {
  test('a11y', function() {
    const el = fixture('some-fixture');
    return axeReport(el);
  });
});
```

### Setting up Travis
By default, `npm test` runs the tests on the command line. However, you can set up a continuous integration server,
like [Travis](https://travis-ci.org/), to run the tests every time a new commit is made.

Before you do anything, make sure you've set up Travis on your Github repository according to the [Getting Started](https://docs.travis-ci.com/user/getting-started/) guide, including flipping your repo "on" on your [Travis profile page](https://travis-ci.org/profile)

The `pwa-starter-kit` Travis config lives in `.travis.yml`, and has a couple of different configurations in its matrix
- `os: osx` is a macOS build, which runs the integration tests. This is because in order to match, the baseline tests needs to be ran on the same OS as the tests (or else you're likely to run into web font problems). On our team, we generated the screenshots on a macOS machine, so it makes sense the tests are setup in the same way.
- `os: linux` runs the unit tests on Firefox/Chrome.
- [SauceLabs](https://saucelabs.com/) testing for browsers that aren't available on Travis, such as Edge.

To trigger your first continuous integration test, push a new commit to a branch.

### SauceLabs testing
At the moment, Travis CI only has Mac OS and Linux VMs, which means you need to use a separate third-party service to automate testing on Microsoft browsers. We use [SauceLabs](https://saucelabs.com/) for this, and in order to set it up on Travis, you need to include some private keys in the [.travis.yml](https://github.com/Polymer/pwa-starter-kit/blob/master/.travis.yml#L20) file (see the [SauceConnect docs](https://docs.travis-ci.com/user/sauce-connect/), [encrypted variables docs](https://docs.travis-ci.com/user/environment-variables#Defining-encrypted-variables-in-.travis.yml)). If you don't want to use SauceLabs at all, delete [this line](https://github.com/Polymer/pwa-starter-kit/blob/master/.travis.yml#L19) and [this section](https://github.com/Polymer/pwa-starter-kit/blob/master/.travis.yml#L20) from the configuration file.

## Integration testing with Puppeteer
[Puppeteer](https://github.com/GoogleChrome/puppeteer) is an `npm` library that lets you control Chrome. It makes it really easy to do things like click on particular elements in the page, wait until certain elements are loaded, and take screenshots. Because it only runs in Chrome, we're only including it as an easy way to do basic integration testing -- making sure that the basics of your app don't break in between commits.

### Router tests
The router tests (in `test/integration/router.js`) provide you with a starting point for using Puppeteer to interact with your page. They show you how you can find a particular node in a shadow root (like the navigation links), and interact with it (in particular, by clicking on it).

We've provided two different examples of doing the same thing (clicking on a node): by injecting a "deep" query selector into your testing page, and using that to find a particular node, or by using the Puppeteer API to target the node on the test side.

### Screenshot testing
Another thing that is useful to test is seeing if the app has changed visually. The examples we've provided test the layout of the app in both wide and narrow screen viewports (since the layout changes). If you want to add a new kind of visual test, you should add a new test suite, similar to [this one](https://github.com/Polymer/pwa-starter-kit/blob/master/test/integration/visual.js#L50), where in the `beforeEach` method you would use Puppeteer methods (similar to the code in the `router.js` tests) to set up the screenshot: clicking on some elements, focusing inputs, submitting forms, etc.

The "golden" (or baseline) screenshots are generated by the [`regenerate-baseline.js`](https://github.com/Polymer/pwa-starter-kit/blob/master/test/integration/screenshots-baseline/regenerate.js) script, which you can run with the `npm run test:regenerate_screenshots` command. If you want to test new parts of the app (for example, a mobile layout with the drawer open), make sure that you add both a new function to generate the screenshot with a similar set up as your test.

Note that if you run `pwa-starter-kit`'s screenshot tests fresh after cloning the repo, they might fail: this is because the checked in baseline screenshots are rendered on a MacOS machine, and if you're running the tests on a different platform, the app might render slightly differently (most likely due to the font metrics not being identical). You should re-generate the baseline screenshots in this case, and then try the test again.
-->

アプリをテストすることは良いことです。 このページでは、私たちが提供したテストの設定をお届けします。

## テストの実行

あなたのアプリをテストするには多くの方法があり、テストを書くことができる多くのフレームワークがあります。以下のユニットテストと統合テストのサンプルを提供しました。統合テストでは[Puppeteer](https://github.com/GoogleChrome/puppeteer)を使用し、ユニットテストでは[WCT](https://github.com/Polymer/web-component-tester)が使えます。アプリケーションを変更したりカスタマイズしたりするときには、テストも更新してください :)

テストフォルダの構造は次のとおりです:
```
test
├── integration
|   └── screenshots-baseline
|   └── router.js
|   └── visual.js
├── unit
|   └── index.html
|   └── counter-element.js
|   └── views-a11y.js
```

どこに

- `screenshots-baseline`は、ビジュアルテストで使用されているように、アプリのスクリーンショットのゴールデン（「正しい」）セットです。
- `router.js` ルータが正しく動作していることをテストし、ナビゲーションリンクをクリックすると実際にナビゲーションが実行されます。
- `visual.js`はあなたのアプリケーションが現在どのように見えるのか、`screenshots-baseline`のスクリーンショットに従っているかを視覚的に比較します。
- `counter-element.js`は、最初のReduxデモで使用された単純なカウンタ要素をテストするWCTユニットテストです。
- `views-a11y.js`は[axe-core](https://github.com/dequelabs/axe-core)を使って各アプリケーションのビューにアクセス可能であることをテストするWCTユニットテストです。

下記のコマンドでテストスイート全体を実行することができます
```
npm test
```

または部分的なテストスイートは下記の通りです。
```
npm run test:unit
npm run test:integration
```

## ユニットテスト

### ベーシックテスト

We use [WCT](https://github.com/Polymer/web-component-tester) to run unit tests. WCT comes pre-packaged with `<test-fixture>`, an element that defines a template of content and copies a clean, new instance of that content into each test suite (more information [here](https://www.polymer-project.org/3.0/docs/tools/tests#test-fixtures)). To add a new unit test to the suite

ユニットテストを実行するには、[WCT](https://github.com/Polymer/web-component-tester)を使用します。 WCTには、コンテンツのテンプレートを定義し、そのコンテンツのクリーンで新しいインスタンスを各テストスイートにコピーする要素である `<test-fixture>`があらかじめパッケージ化されています(詳細は[こちら](https://www.polymer -project.org/3.0/docs/tools/tests#test-fixtures))。スイートに新しい単体テストを追加するには:

- `test/unit/`の下に新しいファイルを作成します（あるいは、すでにある`test/unit/counter-element`をコピーして名前を変更するだけです）。
- テストをWCTスイートの `test/unit/index.html`に追加してください:

```js
WCT.loadSuites([
  ...
  // ネイティブのshadow DOMを使用して 'my-new-test.html'テストスイートを読み込みます:
  'my-new-test.html',
  // shady DOMを使って 'my-new-test.html'テストスイートを読み込みます:
  'my-new-test.html?wc-shadydom=true&wc-ce=true',
]);
```

- このテストスイートに新しいテストを追加したら、下記のコマンドでユニットテストを実行できます:

```
npm run test:unit
```

デフォルトでは、すべてのローカルブラウザでテストが実行されます(Mac OSマシンを使用している場合は、[リモートオートメーションを有効](https://webkit.org/blog/6900/webdriver-support-in-safari-10/)にする必要があります。これを正しく動作させるには、「開発」メニューからSafariのを選択します。 テストに使用するブラウザを設定する場合は、 `-l`コマンドライン引数を使用できます:

```
npm run test:unit -l chrome -l firefox
```

WCTを使用したユニットテストの作成の詳細については、[テストドキュメント](https://www.polymer-project.org/3.0/docs/tools/tests#overview)または[WCTドキュメント](https:/github.com/Polymer/tools/tree/master/packages/web-component-tester#test-fixture)を参照してください。

### アクセシビリティ(A11y)テスト

[Ax-core](https://github.com/dequelabs/axe-core)は、アクセシビリティ違反をHTMLで自動的に監査するライブラリです。

これをユニットテストの内部でより簡単に使用するために、小さなラッパー、[`ax-report.js`](https://github.com/Polymer/pwa-helpers#axe-reportjs)を作成しました。すべての違反を含む `Error`を返します。

これを使用して[特定の要素](https://github.com/Polymer/pwa-starter-kit/blob/master/test/unit/counter-element.html#L70)または[ページ全体](https://github.com/Polymer/pwa-starter-kit/blob/master/test/unit/views-a11y.html)も参照してください。

要素 `el`の機能をテストするために既に` WCT`でテストを設定している場合は、それにa11yテストを追加することもできます:

```js
import {axeReport} from '../axe-report.js';
suite('my-element tests', function() {
  test('a11y', function() {
    const el = fixture('some-fixture');
    return axeReport(el);
  });
});
```

### Travisの設定

デフォルトでは、 `npm test`はコマンドラインでテストを実行します。ただし、[Travis](https://travis-ci.org/)のような継続的な統合サーバーを設定して、新しいコミットが行われるたびにテストを実行することができます。

何かをする前に、[Getting Started](https://docs.travis-ci.com/user/getting-started/)ガイドに従ってGithubリポジトリにTravisの設定して、あなたの[Travisプロファイル](https://travis-ci.org/profile)にレポジトリがあるかも含めて確認してください。

`pwa-starter-kit`のTravis設定は`.travis.yml`にあり、設定にはいくつかの種類があります。
- `os：osx`は、統合テストを実行するmacOSビルドです。 これは、一致させるためには、ベースラインテストをテストと同じOSで実行する必要があります（そうでなければ、Webフォントの問題に遭遇する可能性があります）。私たちのチームでは、macOSマシン上でスクリーンショットを生成したので、同じ方法でテストが設定されていることが理にかなっています。
- `os：linux`はFirefox/Chromeで単体テストを実行します。
- [SauceLabs](https://saucelabs.com/)はEdgeなどのTravisで利用できないブラウザをテストできます。

最初の継続的な統合テストを開始するには、新しいコミットをブランチにプッシュします。

### SauceLabsテスト

現時点では、Travis CIにはMac OSとLinux VMしかありません。つまり、Microsoftのブラウザでのテストを自動化するために、別のサードパーティサービスを使用する必要があります。これには[SauceLabs](https://saucelabs.com/)を使用します。Travisでそれを設定するには、[.travis.yml](https://githubPolymer/pwa-starter-kit/blob/master/.travis.yml#L20)にいくつかの秘密鍵を含める必要があります。 ファイル（[SauceConnect docs](https://docs.travis-ci.com/user/sauce-connect/) [暗号化された変数のドキュメント](https://docs.travis-ci.com/user/environment-variables#Defining-encrypted-variables-in-.travis.yml)。 SauceLabsをまったく使用したくない場合は、[この行](https://github.com/Polymer/pwa-starter-kit/blob/master/.travis.yml#L19)と[このセクション](https://github.com/Polymer/pwa-starter-kit/blob/master/.travis.yml#L20)を設定ファイルから削除します。

## Puppeteerによる統合テスト

[Puppeteer](https://github.com/GoogleChrome/puppeteer)は、Chromeを制御できる `npm`ライブラリです。 ページ内の特定の要素をクリックしたり、特定の要素がロードされるのを待ったり、スクリーンショットを撮ったりするのはとても簡単です。 これはChromeでしか実行されないため、基本的な統合テストを行う簡単な方法として、アプリの基本がコミットの間に壊れないようにするだけです。

### ルータテスト

ルーターテスト（ `test/integration/router.js`）は、Puppeteerを使用してページとやりとりするための出発点を提供します。それらは、（ナビゲーションリンクのような）shadow Root内の特定のノードをどのように見つけて、それと対話するか（特にそれをクリックすることによって）を示します。

同じことをする（ノードをクリックする）2つの異なる例を示します。「深い」クエリーセレクタをテストページに挿入し、それを使用して特定のノードを見つけるか、またはPuppeteer APIを使用してテスト側のノードを見つけるか

### スクリーンショットテスト

テストに役立つもう一つのことは、アプリが視覚的に変化したかどうかを確認することです。私たちが提供した例は、（レイアウトが変更されているので）ワイドスクリーンビューアと狭いスクリーンビューポートの両方でアプリのレイアウトをテストします。新しい種類の視覚テストを追加する場合は、[これ](https://github.com/Polymer/pwa-starter-kit/blob/master/test/integration/visual.js#L50)に似た新しいテストスイートを追加する必要があります。 `beforeEach`メソッドの中でPuppeteerメソッド（`router.js`テストのコードに似ています）を使ってスクリーンショットを設定します: いくつかの要素をクリックし、入力をフォーカスし、フォームを送信します等

"ゴールデン"（またはベースライン）のスクリーンショットは、['regenerate-baseline.js'](https://github.com/Polymer/pwa-starter-kit/blob/master/test/integration/screenshots-baseline/regenerate.js)スクリプトを実行します。これは `npm run test:regenerate_screenshots`コマンドで実行できます。アプリの新しい部分（たとえば、draowerが開いているモバイルレイアウト）をテストする場合は、テストと同様の設定でスクリーンショットを生成するための新しい機能を追加する必要があります。

リポジトリを複製した後にに`pwa-starter-kit`のスクリーンショットテストを新しく実行すると失敗するかもしれないことに注意してください。これはチェックされたベースラインのスクリーンショットがMacOSマシン上でレンダリングされ、別のプラットフォームでは、アプリは多少異なってレンダリングされることがあるからです（フォントメトリックが同一ではないため）。この場合、ベースラインのスクリーンショットを再生成してから、もう一度試してみるべきです。
