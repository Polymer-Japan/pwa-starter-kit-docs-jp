---
layout: post
title: カスタマイズとパーソナライズ
---
<!-- original:
This page will take you through the steps you need to do to modify the app and add your own content.

## Table of Contents
- [Folder structure](#folder-structure)
- [Naming and code conventions](#naming-and-code-conventions)
- [Customizing the app](#customizing-the-app)
  - [Changing the name of your app](#changing-the-name-of-your-app)
  - [Adding a new page](#adding-a-new-page)
  - [Using icons](#using-icons)
  - [Sharing styles](#sharing-styles)
  - [Fonts](#fonts)
  - [But I don't want to use Redux](#but-i-dont-want-to-use-redux)
- [Advanced topics](#advanced-topics)
  - [Responsive layout](#responsive-layout)
  - [Conditionally rendering views](#conditionally-rendering-views)
  - [Routing](#routing)
  - [SEO](#seo)
  - [Fetching data](#fetching-data)
  - [Responding to network state changes](#responding-to-network-state-changes)
  - [State management](#state-management)
  - [Theming](#theming)

# Folder structure
Your app will be initialized with a bunch of folders and files, that looks like this:
```
my-app
├── images
│   ├── favicon.ico
│   └── manifest
│       ├── icon-48x48.png
│       └── ...
├── src
│   ├── store.js
│   ├── actions
│   │   └── ...
│   ├── reducers
│   │   └── ...
│   ├── components
│   │   └── ...
├── test
│   ├── unit
│   │   └── ...
│   └── integration
│       └── ...
├── index.html
├── README.md
├── package.json
├── polymer.json
├── manifest.json
├── service-worker.js
├── sw-precache-config.js
├── wct.conf.json
├── .travis.yml
```
- `images/` has your logos and favicons. If you needed to add any other assets to your application, this would be a good place for them.
- `src/` is where all the code lives. It's broken down in 4 areas:
  - `components/` is the directory that contains all the custom elements in the application.
  - `actions/`, `reducers/` and `store.js` are Redux specific files and folders. Check out the [Redux and state management]({{site.baseurl}}/redux-and-state-management) page for details on that setup.
- `test/` is the directory with all of your tests. it's split in `unit` tests (that are run across different browsers), and `integration` tests, that just run on headless Chrome to ensure that the end-to-end application runs and is accessible. Check out the [application testing]({{site.baseurl}}/application-testing) page for more information.
- `index.html` is your application's starting point. It's where you load your polyfills and the main entry point element.
- `package.json`: the `npm` configuration file, where you specify your dependencies. Make sure you run `npm install` any time you make any changes to this file.
- `polymer.json`: the `polymer cli` configuration file, that specifies how your project should be bundled, what's included in the service worker, etc. ([docs](https://www.polymer-project.org/3.0/docs/tools/polymer-json)).
- `manifest.json` is the PWA [metadata file](https://developers.google.com/web/fundamentals/web-app-manifest/). It contains the name, theme color and logos for your app, that are used whenever a user adds your application to the homescreen.
- `service-worker.js` is a placeholder file for your Service Worker. In each build directory, the `polymer cli` will populate this file with [actual contents](https://www.polymer-project.org/3.0/toolbox/service-worker), but during development it is disabled.
- `sw-precache-config.js` is a [configuration file](https://www.polymer-project.org/3.0/toolbox/service-worker) that sets up the precaching behaviour of the Service Worker (such as which files to be precached, the navigation fallback, etc.).
- `wct.conf.json` is the [web-component-tester](https://github.com/Polymer/web-component-tester) configuration file, that specifies the folder to run tests from, etc.
- `.travis.yml` sets up the integration testing we run on every commit on [Travis](https://docs.travis-ci.com/user/customizing-the-build/).

You can add more app-specific folders if you want, to keep your code organized -- for example, you might want to create a `src/views/` folder and move all the top-level views in there.

# Naming and code conventions
This section covers some background about the naming and coding conventions you will find in this template:
- Generally, an element `<sample-element>` will be created in a file called `src/components/sample-element.js`, and the class used to register it will be called `SampleElement`.
- The elements use a mix of Polymer 3 and `lit-html` via the [`LitElement`](https://github.com/Polymer/lit-element) base class. The structure of one of these elements is basically:

```js
import { LitElement, html } from 'lit-element';
class SampleElement extends LitElement {
  // The properties that your element exposes.
  static get properties() { return {
    publicProperty: { type: Number },
    _privateProperty: { type: String }    /* note the leading underscore */
  }};

  constructor() {
    super();
    // Set up the property defaults here
    this.publicProperty = 0;
    this._privateProperty = '';
  }

  render() {
    // Note the use of the object spread to explicitely
    // call out which properties you're using for rendering.
    const {publicProperty, _privateProperty} = this;

    // Anything code that is related to rendering should be done in here.

    return html`
      < !-- your element's template goes here -- >
    `;
  });

  firstUpdated() {
    // Any code that relies on render having been called once goes here.
    // (for example setting up listeners, etc)
  }
  ...
}
window.customElements.define('sample-element', SampleElement);
```
- Note that private properties are named with a leading underscore (`_foo` instead of `foo`). Since JavaScript doesn't have proper private properties, this in a coding convention that implies this property shouldn't be used outside of the element itself (so you would never write `<sample-element _foo="bar">`).

# Customizing the app
Here are some changes you might want to make to your app to personalize it.

## Changing the name of your app
By default, your app is called `my-app`. If you want to change this (which you obviously will), you'll want to make changes in a bunch of places:
- Config files: `package.json`, `polymer.json` and `manifest.json`
- In the app: `index.html`, the `<title>`, several `meta` fields, and the `appTitle` attribute on the `<my-app>` element

## Adding a new page
There are 4 places where the active page is used at any time:
- As a view in the [`<main>`](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-app.js#L197) element.
- As a navigation link in the [`drawer <nav>`](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-app.js#L189) element. This is the side nav that is shown in the small-screen (i.e. mobile) view.
- As a navigation link in the [`toolbar <nav>`](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-app.js#L179) element. This is the toolbar that is shown in the wide-screen (i.e. desktop) view.
- In the code for [lazy loading](https://github.com/Polymer/pwa-starter-kit/blob/master/src/actions/app.js#L29) pages. We explicitly list these pages, rather that doing something like `import('./my-'+page+'.js')`, so that the bundler knows these are separate routes, and bundles their dependencies accordingly. ⚠️Don't change this! :)

To add a new page, you need to add a new entry in each of these places. Note that if you only want to add an external link or button in the toolbar, then you can skip adding anything to the `<main>` element.

### Create a new page
First, let's create a new element, that will represent the new view for the page. The easiest way to do this is to copy the [`<my-view404>`](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-view404.js) element, since that's a good and basic starting point:
- Copy that file, and rename it to `my-view4.js`. We're going to assume the element's name is also `my-view4`, but if you want to use a name that makes more sense (like `about-page` or something), you can totally use that -- just make sure you are consistent!
- In this new file, rename the class to `MyView404` to `MyView4` (in 2 places), and the element's name to `my-view4`. When you're done, it should look like this:

```js
import { html } from 'lit-element';
import { PageViewElement } from './page-view-element.js';
import { SharedStyles } from './shared-styles.js';

class MyView4 extends PageViewElement {
  render() {
    return html`
      ${SharedStyles}
      <section>
        <h2>Oops! You hit a 404</h2>
        <p>The page you're looking for doesn't seem to exist. Head back
           <a href="/">home</a> and try again?
        </p>
      </section>
    `
  }
}
window.customElements.define('my-view4', MyView4);
```

(🔎This page extends `PageViewElement` rather than `LitElement` as an optimization; for more details on that, check out the [conditional rendering]({{site.baseurl}}/configuring-and-personalizing#conditionally-rendering-views) section).

### Adding the page to the application
Great! Now we that we have our new element, we need to add it to the application!

First, add it to each of the list of nav links. In the toolbar (the wide-screen view) add:
```html
<nav class="toolbar-list">
  ...
  <a ?selected="${_page === 'view4'}" href="/view4">New View!</a>
</nav>
```

Similarly, we can add it to the list of nav links in the drawer:
```html
<nav class="drawer-list">
  ...
  <a ?selected="${_page === 'view4'}" href="$/view4">New View!</a>
</nav>
```

And in the main content itself:
```html
<main class="main-content">
  ...
  <my-view4 class="page" ?active="${_page === 'view4'}"></my-view4>
</main>
```

Note that in all of these code snippets, the `selected` attribute is used to [highlight](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-app.js#L99) the active page, and the `active` attribute is also used to ensure that only the active page is [actually rendered](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/page-view-element.js#L16).

Finally, we need to lazy load this page. Without this, the links will appear, but they won't be able to navigate to your new page, since `my-view4` will be undefined (we haven't imported its source code anywhere). In the [`loadPage ` action creator](https://github.com/Polymer/pwa-starter-kit/blob/master/src/actions/app.js#L29), add a new `case` statement:
```js
switch(page) {
  ...
  case 'view4':
    import('../components/my-view4.js');
    break;
}
```

Don't worry if you don't know what an _action creator_ is yet. You can find a complete explanation of how it fits into the state management story in the [Redux]({{site.baseurl}}/redux-and-state-management) page.

That's it! Now, if you refresh the page, you should be able to see the new link and page. Don't forget to re-build your application before you deploy to production (or test that build), since this new page needs to be added to the output.

### Adding the page to the push manifest

To take advantage of HTTP/2 server push, you need to specify what scripts are needed for the new page. Add a new entry to `push-manifest.json`:

```js
{
  "/view4": {
    "src/components/my-app.js": {
      "crossorigin": "anonymous",
      "type": "script",
      "weight": 1
    },
    "src/components/my-view4.js": {
      "crossorigin": "anonymous",
      "type": "script",
      "weight": 1
    },
    "node_modules/@webcomponents/webcomponentsjs/webcomponents-loader.js": {
      "type": "script",
      "weight": 1
    }
  },

  /* other entries */
}
```

## Using icons
You can inline an `<svg>` directly where you need it in the page, but if there's any reusable icons you'd
like to define once and use in several places, `my-icons.js` is a good spot for that. To add a new icon, you
can just add a new line to that file:
```js
export const closeIcon = html`<svg>...</svg>`
```

Then, you can import it and use it as a template literal in an element's `render()` method:
```js
import { closeIcon } from './my-icons.js';
render() {
  return html`
    <button title="close">${closeIcon}</button>
  `;
}
```

## Sharing styles
Similarly, shared styles are also just exported template literals. If you take a look at `shared-styles.js`, it
exports a `<style>` node template, that is then inlined in an element's `render()` method:
```js
import { SharedStyles } from './shared-styles.js';
render() {
  return html`
    ${SharedStyles}
    <div>...</div>
  `;
}
```

## Fonts
The app doesn't use any web fonts for the content copy, but does use a Google font for the app title. Be careful not too load too many fonts, however: aside from increasing the download size of your first page, web fonts also [slow down the performance](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/webfont-optimization) of an app, and cause flashes of unstyled content.

## But I don't want to use Redux
The `pwa-starter-kit` is supposed to be the well-lit path to building a fairly complex PWA, but it should in no way feel restrictive. If you know what you're doing, and don't want to use Redux to manage your application's state, that's totally fine! We've created a separate template, [`template-no-redux`](https://github.com/Polymer/pwa-starter-kit/tree/template-no-redux), that has the same UI and PWA elements as the main template, but does not have Redux.

Instead, it uses a unidirectional data flow approach: some elements are in charge of [maintaining the state](https://github.com/Polymer/pwa-starter-kit/blob/template-no-redux/src/components/my-view2.js#L44) for their section of the application, and they [pass that data down](https://github.com/Polymer/pwa-starter-kit/blob/template-no-redux/src/components/my-view2.js#L35) to children elements. In response, when the children elements need to update the state, they [fire an event](https://github.com/Polymer/pwa-starter-kit/blob/template-no-redux/src/components/counter-element.js#L56).

# Advanced topics

## Responsive layout
By default, the `pwa-starter-kit` comes with a responsive layout. At `460px`, the application switches from a wide, desktop view to a small, mobile one. You can change this value if you want the mobile layout to apply at a different size.

For a different kind of responsive layout, the [`template-responsive-drawer-layout`](https://github.com/Polymer/pwa-starter-kit/tree/template-responsive-drawer-layout) template displays a persistent app-drawer, inline with the content on wide screens (and uses the same small-screen drawer as the main template).

#### Changing the wide screen styles
The wide screen styles are controlled in CSS by a [media-query](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-app.js#L161). In that block you can add any selectors that would only apply when the window viewport's width is at least `460px`; you can change this pixel value if you want to change the size at which these styles get applied (or, can add a separate style if you want to have several breakpoints).

#### Changing narrow screen styles
The rest of the styles in [`my-app`](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-app.js#L35) are outside of the media-query, and thus are either general styles (if they're not overwritten by the media-query styles), or narrow-screen specific, like [this one](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-app.js#L87) (in this example, the  `<nav class="toolbar-list">` is hidden in the narrow screen view, and visible in the wide screen view).

#### Responsive styles in JavaScript
If you want to run specific JavaScript code when the size changes from a wide to narrow screen (for example, to make the drawer persistent, etc), you can use the [`installMediaQueryWatcher`](https://github.com/Polymer/pwa-helpers/blob/master/media-query.js) helper from `pwa-helpers`. When you [set it up](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-app.js#L244), you can specify the callback that is ran whenever the media query matches.

## Conditionally rendering views
Which view is visible at a given time is controlled through an `active` attribute, that is set if the name of the page matches the location, and is then used for [styling](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-app.js#L148):

```html
<style>
  .main-content .page {
    display: none;
  }
  .main-content .page[active] {
    display: block;
  }
</style>
<main class="main-content">
  <my-view1 class="page" ?active="${page === 'view1'}"></my-view1>
  <my-view2 class="page" ?active="${page === 'view2'}"></my-view2>
  ...
</main>
```

However, just because a particular view isn't visible doesn't mean it's "inactive" -- its JavaScript can still run. In particular, if your application is using Redux, and the view is connected (like [`my-view2`](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-view2.js#L17) for example), then it will get notified **any** time the Redux store changes, which could trigger `render()` to be called. Most of the time this is probably not what you want -- a hidden view shouldn't be updating itself until it's actually visible on screen. Apart from being inefficient (you're doing work that nobody is looking at), you could run into really weird side effects: if a view's `render()` function also updates the title of the application, for example, the title may end up being set incorrectly by one of these inactive views, just because it was the last view to set it.

To get around that, the views inherit from a [`PageViewElement`](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/page-view-element.js) base class, rather than `LitElement` directly. This base class checks whether the `active` attribute is set on the host (the same attribute we use for styling), and calls `render()` only if it [is set](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/page-view-element.js#L15).

If this isn't the behaviour you want, and you want hidden pages to update behind the scenes, then all you have to do is change the view's base class back to `LitElement` (i.e. changing [this line](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-view1.js#L17)). Just look out for those side effects!

## Routing
The app uses a very [basic router](https://github.com/Polymer/pwa-helpers/blob/master/src/router.ts), that listens to changes to the `window.location`. You install the router by passing it a callback, which is a function that will be called any time the location changes:

```js
installRouter((location) => this._locationChanged(location));
```
Then, whenever a link is clicked (or the user navigates back to a page), `this._locationChanged` is called with the new location. You can check the [Redux page]({{site.baseurl}}/redux-and-state-management#routing) to see how this location is stored in the Redux store.

Sometimes you might need to update this location (and the Redux store) imperatively -- for example if you have custom code for link hijacking, or you're managing page navigations in a custom way. In that case, you can manually update the browser's history state yourself, and then call the `this._locationChanged` method manually (thus simulating an action from the router):

```js
// This function would get called whenever you want
// to manually manage the location.

onArticleLinkClick(page) {
  const newLocation = `/article/${page}`
  window.history.pushState({}, '', newLocation);
  this._locationChanged(newLocation);
};
```

## SEO
We've added a starting point for adding rich social graph content to each pages, both using the [Open Graph](http://ogp.me/) protocol (used on Facebook, Slack etc) and [Twitter cards](https://developer.twitter.com/en/docs/tweets/optimize-with-cards/overview/abouts-cards).

This is done in two places:
- Statically, in the [`index.html`](https://github.com/Polymer/pwa-starter-kit/blob/master/index.html#L60). These are used by the homepage, and represent any of the common metadata across all pages (for example, if you don't have a page specific description or image, etc).
- Automatically, after you change pages, in [`my-app.js`](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-app.js#L240), using the [`updateMetadata`](https://github.com/Polymer/pwa-helpers/blob/master/metadata.js) helper from `pwa-helpers`. By default, we update the URL and the title of each page, but there are multiple ways in which you can add page-specific content that depend on your apps.

A different approach is to update this metadata differently, depending on what page you are. For example, the **Books** doesn't update the metadata in the main [top-level element](https://github.com/PolymerLabs/books/blob/master/src/components/book-app.js#L47), but on specific sub-pages. It uses the image thumbnail of a book only on the [detail pages](https://github.com/PolymerLabs/books/blob/master/src/components/book-detail.js#L61), and adds the search query on the [explore page](https://github.com/PolymerLabs/books/blob/master/src/components/book-explore.js#L35).

If you want to test how your site is viewed by Googlebot, Sam Li has a great [article](https://medium.com/dev-channel/polymer-2-and-googlebot-2ad50c5727dd) on gotchas to look out for -- in particular, the testing section covers a couple tools you can use, such as [Fetch as Google](https://support.google.com/webmasters/answer/6066468?hl=en) and [Mobile-Friendly Test](https://search.google.com/test/mobile-friendly).

## Fetching data
If you want to fetch data from an API or a different server, we recommend dispatching an action creator from a component, and making that fetch asynchronously in a Redux action. For example, the **Flash Cards** sample app dispatches a [`loadAll`](https://github.com/notwaldorf/flash-cards/blob/master/src/components/my-app.js#L148) action creator when the main element boots up; it is that action creator that then does the [actual fetch](https://github.com/notwaldorf/flash-cards/blob/master/src/components/my-app.js#L148) of the file and sends it back to the main component by adding the data to the state [in a reducer](https://github.com/notwaldorf/flash-cards/blob/master/src/reducers/data.js#L7).

A similar approach is taken in the **Hacker News** app where an element [dispatches an action creator](https://github.com/PolymerLabs/polymer-redux-hn/blob/master/src/components/hn-item.ts#L55), and it's that action creator that actually [fetches the data](https://github.com/PolymerLabs/polymer-redux-hn/blob/master/src/actions/items.ts#L33) from the HN API.

## Responding to network state changes
You might want to change your UI as a response to the network state changing (i.e. going from offline to online).

Using the [`installOfflineWatcher`](https://github.com/Polymer/pwa-helpers/blob/master/network.js) helper from `pwa-helpers`, we've added a [callback](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-app.js#L243) that will be called any time we go online or offline. In particular, we've added a [snackbar](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-app.js#L219) that gets shown; you can configure its contents and style in [`snack-bar.js`](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/snack-bar.js). Note that the snackbar is shown as a result of a Redux [action creator](https://github.com/Polymer/pwa-starter-kit/blob/master/src/actions/app.js#L69) being dispatched, and its duration can be configured there.

Rather than just using it as an FYI, you can use the offline status to display conditional UI in your application. For example, the **Books** sample app displays an [offline view](https://github.com/PolymerLabs/books/blob/master/src/components/book-offline.js) rather than the details view when the [application is offline](https://github.com/PolymerLabs/books/blob/master/src/components/book-detail.js#L293).

## State management
There are many different ways in which you can manage your application's state, and choosing the right one depends a lot on the size of your team and application. For simple applications, a uni-directional data flow pattern might be enough (the top level, `<my-app>` element could be in charge of being the source of state truth, and it could pass it down to each of the elements, as needed); if that's what you're looking for, check out the [`template-no-redux`](https://github.com/Polymer/pwa-starter-kit/tree/template-no-redux) branch.

Another popular approach is [Redux](https://redux.js.org/), which keeps the state in a store outside of the app, and passes immutable copies to each element. To see how that is set up, check out the [Redux and state management]({{site.baseurl}}/redux-and-state-management) section for an explainer, and more details.


## Theming
This section is useful both if you want to change the default colours of the app, or if you want to let your users be able to switch between different themes.

#### Changing the default colours
For ease of theming, we've defined all of the colours the application uses as [CSS custom properties](https://developer.mozilla.org/en-US/docs/Web/CSS/--*), in the [`<my-app>`](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-app.js#L40) element. Custom properties are variables that can be reused throughout your CSS. For example, to change the application header to be white text on a lavender background, then you need to update the following properties:
```css
--app-header-background-color: lavender;
--app-header-text-color: black;
```

And similarly for the other UI elements in the application.

#### Switching themes
Re-theming the entire app basically involves updating the custom properties that are used throughout the app.
If you just want to personalize the default template with your own theme, all you have to do is change the values of the app's [custom properties](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-app.js#L46).

If you want to be able to switch between two different themes in the app (for example between a "light" and "dark" theme), you could just add a class (for example, `dark-theme`) to the `my-app` element for the new theme, and style that separately. This would end up looking similar to this:

```css
:host {
  /* This is the default, light theme */
  --app-primary-color: red;
  --app-secondary-color: black;
  --app-text-color: var(--app-secondary-color);
  --app-header-background-color: white;
  --app-header-text-color: var(--app-text-color);
  ...
}

:host.dark-theme {
  /* This is the dark theme */
  --app-primary-color: yellow;
  --app-secondary-color: white;
  --app-text-color: var(--app-secondary-color);
  --app-header-background-color: black;
  --app-header-text-color: var(--app-text-color);
  ...
}
```

You control when this class is added; this could be when a "use dark theme" button is clicked, or based on a hash parameter in the location, or the time of day, etc.

## Next steps
Now that you're done configuring your application, check out the next steps:
- [Testing the performance]({{site.baseurl}}/performance-testing) of your app to ensure your users have a fast experience.
- [General testing]({{site.baseurl}}/application-testing) your app to make sure new changes don't accidentally cause regressions.
- [Building and deploying]({{site.baseurl}}/building-and-deploying) to production.
-->

このページでは、アプリを変更して独自のコンテンツを追加するために必要な手順について説明します。

## 目次
- [ディレクトリ構造](#folder-structure)
- [命名規則とコード規則](#naming-and-code-conventions)
- [アプリのカスタマイズ](#customizing-the-app)
  - [アプリの名前を変更する](#changing-the-name-of-your-app)
  - [新しいページを追加する](#adding-a-new-page)
  - [アイコンの使用](#using-icons)
  - [スタイルの共有](#sharing-styles)
  - [フォント](#fonts)
  - [Reduxを使わない方法は？](#but-i-dont-want-to-use-redux)
- [高度な使い方](#advanced-topics)
  - [レスポンシブレイアウト](#responsive-layout)
  - [条件付きレンダリングビュー](#conditionally-rendering-views)
  - [ルーティング](#routing)
  - [SEO](#seo)
  - [データの取得](#fetching-data)
  - [ネットワーク状態を取得する](#responding-to-network-state-changes)
  - [状態管理](#state-management)
  - [テーマ](#theming)


<a id="folder-structure">

# ディレクトリ構造

アプリは多くのフォルダとファイルで初期化されます:

```
my-app
├── images
│   ├── favicon.ico
│   └── manifest
│       ├── icon-48x48.png
│       └── ...
├── src
│   ├── store.js
│   ├── actions
│   │   └── ...
│   ├── reducers
│   │   └── ...
│   ├── components
│   │   └── ...
├── test
│   ├── unit
│   │   └── ...
│   └── integration
│       └── ...
├── index.html
├── README.md
├── package.json
├── polymer.json
├── manifest.json
├── service-worker.js
├── sw-precache-config.js
├── wct.conf.json
├── .travis.yml
```
- `images /`にはあなたのロゴとファビコンがあります。他のアセットをアプリケーションに追加する必要がある場合は、これが適しています。
- `src /`はすべてのコードが存在する場所です。それは4つのエリアで分解されています:
  - `components /`は、アプリケーション内のすべてのカスタム要素を含むディレクトリです。
  - `actions /`、 `reducers /`と `store.js`はRedux固有のファイルとフォルダです。その設定の詳細については、[Redux and state management]({{site.baseurl}}/redux-and-state-management)のページを参照してください。
- `test /`はすべてのテストを含むディレクトリです。これは、（異なるブラウザ間で実行される） `unit`テストと、エンドレスエンドのアプリケーションが実行され、アクセス可能であることを保証するために、ヘッドレスChromeで実行される`integration`テストに分割されています。詳細については、[アプリケーションテスト]({{site.baseurl}}/application-testing)のページを参照してください。
- `index.html`はアプリケーションの出発点です。これは、ポリフィルとメインエントリポイント要素をロードする場所です。
- `package.json`: あなたの依存関係を指定する `npm`設定ファイル。このファイルに変更を加えるたびに、 `npm install`を必ず実行してください。
- `polymer.json`: あなたのプロジェクトのバンドル方法、サービスワーカーに含まれるものなどを指定する `polymer cli`設定ファイル ([docs](https://www.polymer-project.org/3.0/docs/tools/polymer-json)).
- `manifest.json`はPWAの[メタデータファイル](https://developers.google.com/web/fundamentals/web-app-manifest/)です。ユーザーがアプリケーションをホームスクリーンに追加するたびに使用される、アプリケーションの名前、テーマの色、ロゴが含まれています。
- `service-worker.js`はサービスワーカーのプレースホルダファイルです。各ビルドディレクトリで、 `polymer cli`がこのファイルに[実際のコンテンツ](https://www.polymer-project.org/3.0/toolbox/service-worker)を登録しますが、開発中は空になっています。
- `sw-precache-config.js`はサービスワーカーのプリキャッシュ動作を設定する[設定ファイル](https://www.polymer-project.org/3.0/toolbox/service-worker)です。(保護されるファイル、ナビゲーションのフォールバックなど）。
- `wct.conf.json`は、テストを実行するフォルダを指定する[web-component-tester](https://github.com/Polymer/web-component-tester)の設定ファイルです。
- `.travis.yml`は、[Travis](https://docs.travis-ci.com/user/customizing-the-build/)のコミットごとに実行する統合テストを設定します。

必要に応じて、アプリケーション固有のフォルダを追加し、コードを整理しておくことができます。たとえば、 `src/views/`フォルダを作成し、そこにあるすべてのトップレベルのビューを移動することができます。

<a id="naming-and-code-conventions">

# 命名規則とコード規則

このセクションでは、このテンプレートにある命名規則とコーディング規則の背景について説明します:

- 一般に、要素 `<sample-element>`は `src/components/sample-element.js`というファイルに作成され、それを登録するためのクラスは` SampleElement`と呼ばれます。
- 要素は、[`LitElement`](https://github.com/Polymer/lit-element)基本クラスを介して、Polymer 3と `lit-html`の組み合わせを使用します。これらの要素の1つの構造は基本的には次のとおりです。

```js
import { LitElement, html } from '@polymer/lit-element';
class SampleElement extends LitElement {
  // 要素が使うプロパティ
  static get properties() { return {
    publicProperty: { type: Number },
    _privateProperty: { type: String }    /* 先頭にアンダースコアを使っています */
  }};

  constructor() {
    super();
    // ここでプロパティのデフォルト値を設定
    this.publicProperty = 0;
    this._privateProperty = '';
  }

  render() {
    // オブジェクトの分割代入の使っています。
    // レンダリングに使用するプロパティを呼び出します。
    const {publicProperty, _privateProperty} = this;

    // レンダリングに関連するコードはここで行う必要があります。

    return html`
      <!-- 要素のテンプレートはここにあります -->
    `;
  });

  firstUpdated() {
    // ここに一度だけ呼び出されるレンダリングに依存するコード
    // (例えばイベントリスナーの設定など)
  }
  ...
}
window.customElements.define('sample-element', SampleElement);
```
- プライベートプロパティの先頭にアンダースコア（ `foo`ではなく` _foo`）が付けられていることに注意してください。 JavaScriptは適切なプライベートプロパティを持っていないので、このプロパティを要素自体の外部で使うべきではないことを暗示するコーディング規則です(`<sample-element _foo = "bar">`と書くことはできません)。

<a id="customizing-the-app">

# アプリのカスタマイズ

アプリをパーソナライズするためにいくつかの変更を加えたい場合があります。

<a id="changing-the-name-of-your-app">

## アプリの名前を変更する

デフォルトでは、あなたのアプリは `my-app`と呼ばれます。これを変更したい場合は、あなたは複数の場所で変更を加える必要があります:

- 設定ファイル: `package.json`, `polymer.json`, `manifest.json`
- アプリ内: `index.html`, `<title>`, いくつかの `meta` タグ, `<my-app>`要素の`appTitle`属性

<a id="adding-a-new-page">

## 新しいページを追加する

アクティブなページに使用される場所は4箇所あります:

- [`<main>`](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-app.js#L197)要素
- [`ドロワー <nav>`](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-app.js#L189)要素の中のナビゲーションのリンク。これは、小さな画面（モバイル）で表示されるサイドナビゲーターです。
- [`toolbar <nav>`](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-app.js#L179)要素の中のナビゲーションのリンク。これは、ワイドスクリーン（デスクトップ）ビューで表示されるツールバーです。
- [遅延読み込み](https://github.com/Polymer/pwa-starter-kit/blob/master/src/actions/app.js#L29)の処理部分。`import('./my +'+ page +'.js')`のようなことをするのではなく、これらのページは明示的にリストアップします。 ⚠️これを代えてはいけません! :)

新しいページを追加するには、それぞれの場所に新しいエントリを追加する必要があります。ツールバーに外部リンクやボタンだけを追加したい場合は `<main>`要素に追加するのをスキップすることができます。

### 新しいページの作成

まず、ページの新しいビューを表す新しい要素を作成しましょう。これを行う最も簡単な方法は、[`<my-view404>`](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-view404.js)をコピーすることです。これは基本的な出発点です:

- そのファイルをコピーし、 `my-view4.js`に名前を変更してください。要素の名前も `my-view4`であると仮定しますが、（about-pageなどのように）より意味のある名前を使用したい場合でも、それをちゃんと使用できます。一貫した名前付けをしましょう！

- この新しいファイルで、クラスの名前を`MyView404`から`MyView4`(2か所)を変更し、要素の名前を「my-view4」に変更します。完了したら、これは次のようになります:

```js
import { html } from '@polymer/lit-element/lit-element.js';
import { PageViewElement } from './page-view-element.js';
import { SharedStyles } from './shared-styles.js';

class MyView4 extends PageViewElement {
  render() {
    return html`
      ${SharedStyles}
      <section>
        <h2>Oops! You hit a 404</h2>
        <p>The page you're looking for doesn't seem to exist. Head back
           <a href="/">home</a> and try again?
        </p>
      </section>
    `
  }
}
window.customElements.define('my-view4', MyView4);
```

(🔎このページは最適化として `LitElement`ではなく` PageViewElement`を継承しています。さらに詳しい情報は[条件付き描画]({{site.baseurl}}/configuring-and-personalizing#conditionally-rendering-views)を参照してください)

### ページをアプリケーションに追加する

すばらしいです！新しい要素ができたので、アプリケーションに追加します。

まず、ナビゲーションリンクの各リストに追加します。ツールバー（ワイドスクリーンビュー）への追加:

```html
<nav class="toolbar-list">
  ...
  <a ?selected="${_page === 'view4'}" href="/view4">New View!</a>
</nav>
```

同様に、引き出し内のナビゲーションリンクのリストに追加することもできます:

```html
<nav class="drawer-list">
  ...
  <a ?selected="${_page === 'view4'}" href="$/view4">New View!</a>
</nav>
```

そして、メインコンテンツ自体へ:
```html
<main class="main-content">
  ...
  <my-view4 class="page" ?active="${_page === 'view4'}"></my-view4>
</main>
```

これらのすべてのコードスニペットでは、 `selected`属性が[ハイライト](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-app.js#L99)の為に使用されています。アクティブなページだけが[実際にレンダリングされる](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/page-view-element.js#L16)ことを保証するために `active`属性も使用されます。

最後に、このページを遅延読み込みに対応させる必要があります。これがなければ、リンクは表示されますが、my-view4が定義されていないので（ソースコードのどこにもインポートしてるところがないため）、新しいページに移動することはできません。 [`loadPage`アクションクリエータ](https://github.com/Polymer/pwa-starter-kit/blob/master/src/actions/app.js#L29)に新しい`case`ステートメントを追加します:

```js
switch(page) {
  ...
  case 'view4':
    import('../components/my-view4.js');
    break;
}
```

_action creator_が何であるかを知らないと心配しないでください。 [Redux]({{site.baseurl}}/redux-and-state-management)のページでは、状態管理について完全に説明されています。

これでおしまい！ここでページを更新すると、新しいリンクとページが表示されるはずです。この新しいページを出力に追加する必要があるため、プロダクションにデプロイする前にアプリケーションを再ビルドすることを忘れないでください（またはそのビルドをテストしてください）。

### push manifestにページを追加する

HTTP/2サーバープッシュを利用するには、新しいページに必要なスクリプトを指定する必要があります。 `push-manifest.json`に以下の新しいエントリを追加してください:

```js
{
  "/view4": {
    "src/components/my-app.js": {
      "crossorigin": "anonymous",
      "type": "script",
      "weight": 1
    },
    "src/components/my-view4.js": {
      "crossorigin": "anonymous",
      "type": "script",
      "weight": 1
    },
    "node_modules/@webcomponents/webcomponentsjs/webcomponents-loader.js": {
      "type": "script",
      "weight": 1
    }
  },

  /* 他のエントリ */
}
```

<a id="using-icons">

## アイコンの使用

ページ内で `<svg>`を直接インライン化することはできますが、再利用可能なアイコンがあれば、一度定義していくつかの場所で使うことができます。 `my-icons.js`はそのための良い場所です。新しいアイコンを追加するには、そのファイルに新しい行を追加するだけです:

```js
export const closeIcon = html`<svg>...</svg>`
```

次にそれをインポートし、それを要素の `render()`メソッドでテンプレートリテラルとして使用することができます:

```js
import { closeIcon } from './my-icons.js';
render() {
  return html`
    <button title="close">${closeIcon}</button>
  `;
}
```

<a id="sharing-styles">

## スタイルの共有

同様に、共有スタイルはエクスポートされたテンプレートリテラルにすぎません。 `shared-styles.js`を見ると、要素の` render()`メソッドでインライン化された`<style>`ノードのテンプレートがエクスポートされています:

```js
import { SharedStyles } from './shared-styles.js';
render() {
  return html`
    ${SharedStyles}
    <div>...</div>
  `;
}
```

<a id="fonts">

## フォント

アプリはコンテンツにウェブフォントを使用しませんが、アプリのタイトルにGoogleフォントを使用します。あまりにも多くのフォントを読み込まないように注意してください: 最初のページのダウンロードサイズを増やすこととは別に、Webフォントも[パフォーマンスを低下させ](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/webfont-optimization)、スタイルのないコンテンツを点滅させてしまいます。

<a id="but-i-dont-want-to-use-redux">

## Reduxを使わない方法は？

` pwa-starter-kit`は、かなり複雑なPWAを構築するうえよい方法だと思われますが、それによって制限があるわけではありません。あなたが何をしているのか分かっていて、Reduxを使ってアプリケーションの状態を管理したくないのなら、それはよい考えだと思われます!同じUIとPWAを持つ別のテンプレート[`template-no-redux`](https://github.com/Polymer/pwa-starter-kit/tree/template-no-redux)が用意されていて、要素をメインテンプレートするのは同じですが、Reduxを使いません。

Instead, it uses a unidirectional data flow approach: some elements are in charge of [maintaining the state](https://github.com/Polymer/pwa-starter-kit/blob/template-no-redux/src/components/my-view2.js#L44) for their section of the application, and they [pass that data down](https://github.com/Polymer/pwa-starter-kit/blob/template-no-redux/src/components/my-view2.js#L35) to children elements. In response, when the children elements need to update the state, they [fire an event](https://github.com/Polymer/pwa-starter-kit/blob/template-no-redux/src/components/counter-element.js#L56).

代わりに、一方向データフローアプローチを使用しています。いくつかの要素が[状態の維持](https://github.com/Polymer/pwa-starter-kit/blob/template-no-redux/src/components/my-view2.js＃L44)を担当しており、アプリケーションにセクションを追加し、子要素に[データを渡します](https://github.com/Polymer/pwa-starter-kit/blob/template-no-redux/src/components/my-view2.js＃L35)。その応答で子要素が状態を更新する必要があるとき、彼らは[イベントを発生させる](https://github.com/Polymer/pwa-starter-kit/blob/template-no-redux/src/components/counter-element.js＃L56)必要があります。

<a id="advanced-topics">

# 高度な使い方

<a id="responsive-layout">

## レスポンシブレイアウト

デフォルトで`pwa-starter-kit`はレスポンシブレイアウトとなりますがあります。 `460px`では、アプリケーションは広いデスクトップビューから小さなモバイルビューに切り替わります。モバイルレイアウトを異なるサイズで適用する場合は、この値を変更できます。

For a different kind of responsive layout, the [`template-responsive-drawer-layout`](https://github.com/Polymer/pwa-starter-kit/tree/template-responsive-drawer-layout) template displays a persistent app-drawer, inline with the content on wide screens (and uses the same small-screen drawer as the main template).

異なる種類のレスポンスレイアウトの場合、[`template-responsive-drawer-layout`](https://github.com/Polymer/pwa-starter-kit/tree/template-responsive-drawer-layout)テンプレートには、永続的なアプリケーションドロワーがワイド画面のコンテンツとインラインで表示されます（メインテンプレートと同じ小さなdrawerを使用します）。

#### ワイドスクリーンスタイルを変更する

ワイドスクリーンスタイルはCSSの[media-query](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-app.js#L161)で制御されます。このブロックでは、ウィンドウビューポートの幅が少なくとも460pxのときにのみ適用されるセレクタを追加できます。これらのスタイルが適用されるサイズを変更する場合はこのピクセル値を変更できます（複数のブレークポイントが必要な場合は別のスタイルを追加できます）。

#### ナロースクリーンスタイルの変更

[`my-app`](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-app.js#L35)の他のスタイルは、media-queryの外部であり、一般的なスタイル（メディアクエリスタイルで上書きされない場合）として[このように](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-app.js#L87)ナロースクリーン向けに定義されています（この例では`<nav class =" toolbar-list ">`はナロースクリーンでは隠されていて、ワイドスクリーンで表示されます）。

#### JavaScriptでレスポンシブスタイルを制御する

サイズが幅の広い画面から狭い画面に変更されたときに特定のJavaScriptコードを実行する場合(例えばdrawerをそのままにするなど)、 `pwa-helpers`の[`installMediaQueryWatcher`](https://github.com/Polymer/pwa-helpers/blob/master/media-query.js)ヘルパーが使えます。もし[設定されていれば](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-app.js#L244)、メディアクエリが一致するたびに実行されるコールバックを指定できます。

<a id="conditionally-rendering-views">

## 条件付きレンダリングビュー

ある時点でどのビューが表示されているかは、`active`属性によって制御されます。これは、ページの名前が場所と一致する場合に設定され、[スタイリング](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-app.js#L148)されます:

```html
<style>
  .main-content .page {
    display: none;
  }
  .main-content .page[active] {
    display: block;
  }
</style>
<main class="main-content">
  <my-view1 class="page" ?active="${page === 'view1'}"></my-view1>
  <my-view2 class="page" ?active="${page === 'view2'}"></my-view2>
  ...
</main>
```

ただし、特定のビューが表示されていないという理由だけで、そのビューが「非アクティブ」であることを意味するわけではありません。そのJavaScriptは引き続き実行できます。
特に、アプリケーションがReduxを使用しており、ビューが接続されている場合、(例えば[`my-view2`](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-view2.js#L17)のように)、Reduxストアが変更された**どんな**タイミングでも`render()`を呼び出すことができます。
もっともほとんどの場合、これはおそらくあなたが望むものではありません。実際に画面に表示されるまで、隠しビューは更新されるべきではありません。たとえば、タイトルは、最後に設定したビューであるため、これらの非アクティブなビューの1つによって誤って設定される可能性があります。

この問題を回避するために、ビューは`LitElement`ではなく[`PageViewElement`](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/page-view-element.js)のベースクラスを継承し、この基本クラスは `active`属性がホスト上で設定されているかどうか（スタイリングに使用するのと同じ属性）をチェックし、`render()`が[設定されている場合のみ](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/page-view-element.js#L15)を呼び出されます。

もしこの動きが気にくわなくて、隠れたページをバックグラウンドで更新したいのであれば、ビューの基本クラスを `LitElement`に戻すだけです（つまり、[ここ](https：//github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-view1.js#L17)を変更するだけです)。実際にそこで起きる副作用を見てみてください！

<a id="routing">

## ルーティング

このアプリは非常に[基本的なルータ](https://github.com/Polymer/pwa-helpers/blob/master/src/router.ts)を使っていて、それは `window.location`への変更を待ち受けます。ルータをインストールするには、コールバックを渡します。コールバックは、ロケーションが変わるたびに呼び出される関数です:

```js
installRouter((location) => this._locationChanged(location));
```

これでリンクがクリックされるたびに（またはユーザーがページに戻るとき）、新しいロケーションで `this._locationChanged`が呼び出されます。 [Reduxページ]({{site.baseurl}}/redux-and-state-management#routing)をチェックして、この場所がReduxストアにどのように格納されているかを確認することができます。

リンク移動を上書きするカスタマイズがしたい場合やナビゲーションをカスタマイズして管理している場合など、この部分（およびReduxストア）を変更したい場合があるでしょう。その場合は、ブラウザの履歴状態を手動で更新してから、 `this._locationChanged`メソッドを手動で呼び出すことで実現できます（ルータからのアクションをシミュレートします）:

```js
// この関数は、手動で場所を管理するときに
// 呼び出されます。

onArticleLinkClick(page) {
  const newLocation = `/article/${page}`
  window.history.pushState({}, '', newLocation);
  this._locationChanged(newLocation);
};
```

<a id="seo">

## SEO

[Open Graph](http://ogp.me/)プロトコル（Facebook、Slackなどで使用）、[Twitter Card](http://developer.twitter.com/en/docs/tweets/optimize-with-cards/overview/abouts-cards)などのリッチなソーシャルグラフコンテンツを各ページに追加されています。

これは2か所で行われます:

- 静的には、[`index.html`](https://github.com/Polymer/pwa-starter-kit/blob/master/index.html#L60)にあります。これらはホームページで使用され、すべてのページで共通のメタデータを表します（たとえば、ページ固有の説明や画像などがない場合など）。
- 動的には、ページを移動した後、[`my-app.js`](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-app.js#L240)で、 `pwa-helpers`の[`updateMetadata`](https://github.com/Polymer/pwa-helpers/blob/master/metadata.js)ヘルパーが使われます。既定では、各ページのURLとタイトルを更新しますが、アプリに依存するページ固有のコンテンツを追加する方法もあります。

別のアプローチは、あなたがどのページであるかに応じて、このメタデータを異なる方法で更新することです。たとえば、**Booksアプリ**ではメインの[トップレベル要素](https://github.com/PolymerLabs/books/blob/master/src/components/book-app.js#L47)ではメタデータを更新せず、サブ要素で指定しています。[詳細ページ](https://github.com/PolymerLabs/books/blob/master/src/components/book-detail.js#L61)で書籍の画像サムネイルを使用し、[検索ページ](https://github.com/PolymerLabs/books/blob/master/src/components/book-explore.js#L35)に検索クエリを追加します。

If you want to test how your site is viewed by Googlebot, Sam Li has a great [article](https://medium.com/dev-channel/polymer-2-and-googlebot-2ad50c5727dd) on gotchas to look out for -- in particular, the testing section covers a couple tools you can use, such as [Fetch as Google](https://support.google.com/webmasters/answer/6066468?hl=en) and [Mobile-Friendly Test](https://search.google.com/test/mobile-friendly).

<a id="fetching-data">

## データの取得

APIまたは別のサーバーからデータをフェッチする場合は、Redux actionでaction creatorをdispatchしてから非同期で処理することをお勧めします。

For example, the **Flash Cards** sample app dispatches a [`loadAll`](https://github.com/notwaldorf/flash-cards/blob/master/src/components/my-app.js#L148) action creator when the main element boots up; it is that action creator that then does the [actual fetch](https://github.com/notwaldorf/flash-cards/blob/master/src/components/my-app.js#L148) of the file and sends it back to the main component by adding the data to the state [in a reducer](https://github.com/notwaldorf/flash-cards/blob/master/src/reducers/data.js#L7).

たとえば、**Flash Cards**サンプルアプリケーションは、メイン要素が起動するときに[`loadAll`](https://github.com/notwaldorf/flash-cards/blob/master/src/components/my-app.js#L148) actionをdispatchします。そのaction creatorが[実際のfetch](https://github.com/notwaldorf/flash-cards/blob/master/src/components/my-app.js#L148)を実行し、その状態を[reducer](https://github.com/notwaldorf/flash-cards/blob/master/src/reducers/data.js#L7)で追加して、メインコンポーネントに戻します。

A similar approach is taken in the **Hacker News** app where an element [dispatches an action creator](https://github.com/PolymerLabs/polymer-redux-hn/blob/master/src/components/hn-item.ts#L55), and it's that action creator that actually [fetches the data](https://github.com/PolymerLabs/polymer-redux-hn/blob/master/src/actions/items.ts#L33) from the HN API.
**Hacker News**アプリでも[action creatorをdispatch]((https://github.com/PolymerLabs/polymer-redux-hn/blob/master/src/components/hn-item.ts#L55)する同様のアプローチが取られています。実際にHN APIから[データを取得](https://github.com/PolymerLabs/polymer-redux-hn/blob/master/src/actions/items.ts#L33)するのはaction creatorです。

<a id="responding-to-network-state-changes">

## ネットワーク状態を取得する

ネットワーク状態の変化（オフラインからオンライン）への応答としてUIを変更することができます。

`pwa-helpers`の[`installOfflineWatcher`](https://github.com/Polymer/pwa-helpers/blob/master/network.js)ヘルパーを使用して、オンラインまたはオフラインになるたびに呼び出されるコールバックを追加できます。それは[スナックバー]((https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-app.js#L219))で使用されています。[snack-bar.js](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/snack-bar.js)でコンテンツとスタイルを設定することができます。 シナックバーはRedux [action creator](https://github.com/Polymer/pwa-starter-kit/blob/master/src/actions/app.js#L69)にdispatchされた結果として表示されることを覚えておいてください。

FYI 参考までに、オフラインステータスを使用して、アプリケーションに条件付きUIを表示できます。 例えば、**Booksアプリ**は[オフラインになると](https://github.com/PolymerLabs/books/blob/master/src/components/book-detail.js#L293)、詳細画面ではなく[オフラインビュー](https://github.com/PolymerLabs/books/blob/master/src/components/book-offline.js)となります。

<a id="state-management">

## 状態管理

アプリケーションの状態を管理するにはさまざまな方法があり、適切なものを選択することはチームとアプリケーションのサイズに大きく依存します。

単純なアプリケーションでは、一方向のデータフローパターンで十分かもしれません(最上位レベルの `<my-app>`要素は状態真理のソースである可能性があり、各要素に渡すことができます)。それで問題なければ[`template-no-redux`](https://github.com/Polymer/pwa-starter-kit/tree/template-no-redux)ブランチを見てください。

別の一般的なアプローチは、[Redux](https://redux.js.org/)です。これは、アプリケーションの外部のストアに状態を保持し、不変のコピーを各要素に渡します。設定方法を確認するには、[Reduxと状態管理]({{site.baseurl}}/redux-and-state-management)セクションを参照してください。

<a id="theming">

## テーマ

このセクションは、アプリのデフォルトの色を変更したい場合や、ユーザーが異なるテーマを切り替えることができるようにする場合に便利です。

#### デフォルトの色を変更する

テーマ設定を簡単にするため、アプリケーションで使用するすべての色を[CSSカスタムプロパティ](https://developer.mozilla.org/en-US/docs/Web/CSS/--*)で[`<my-app>`](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-app.js#L40)要素で宣言されています。カスタムプロパティは、CSS全体で再利用できる変数です。たとえば、アプリケーションのヘッダーをラベンダーの背景に白いテキストに変更するには、次のプロパティを更新するだけです:

```css
--app-header-background-color: lavender;
--app-header-text-color: black;
```

また、アプリケーションの他のUI要素も同様です。

#### テーマの切り替え

アプリ全体を再テーマ化するには、アプリ全体で使用されるカスタムプロパティを更新する必要があります。

独自のテーマでデフォルトテンプレートをパーソナライズしたいだけの場合は、アプリの[カスタムプロパティ](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-app.js#L46)の値を変更するだけです。

アプリケーションの2つの異なるテーマ（明るいと暗いなど）を切り替えるには、`my-app`要素に`dark-theme`などのクラスを設定し、別々にそのスタイルを指定します。これは次のようなコードになるでしょう:

```css
:host {
  /* これはデフォルトで明るい色調のテーマ */
  --app-primary-color: red;
  --app-secondary-color: black;
  --app-text-color: var(--app-secondary-color);
  --app-header-background-color: white;
  --app-header-text-color: var(--app-text-color);
  ...
}

:host.dark-theme {
  /* これは暗い色調のテーマ */
  --app-primary-color: yellow;
  --app-secondary-color: white;
  --app-text-color: var(--app-secondary-color);
  --app-header-background-color: black;
  --app-header-text-color: var(--app-text-color);
  ...
}
```

これは「ダークテーマを使用する」ボタンがクリックされたときや、その場所のハッシュパラメータ、または時刻などに基づいてこのクラスがいつ追加されるかをコントロールすることになるでしょう。

## 次のステップ

これで、アプリケーションの設定が完了しました。次のステップに進みましょう:

- [パフォーマンスをテスト]({{site.baseurl}}/performance-testing) してユーザが快適なスピードで利用できるように
- [一般的なテスト]({{site.baseurl}}/application-testing) で新しい変更が間違って思いもよらないエラーとなっていないか確認
- [ビルドとデプロイ]({{site.baseurl}}/building-and-deploying) で本番環境に公開
