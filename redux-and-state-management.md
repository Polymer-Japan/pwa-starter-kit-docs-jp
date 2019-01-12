---
layout: post
title: Reduxと状態管理
---
<!-- original:
This page will take you through the steps you need to do to use Redux to manage your application's state.

## Table of contents

- [General principles](#general-principles)
  - [Some definitions](#some-definitions)
  - [Naming conventions](#naming-conventions)
- [Connecting elements to the store](#connecting-elements-to-the-store)
  - [What to connect](#what-to-connect)
  - [How to connect](#how-to-connect)
    - [Creating a store](#creating-a-store)
    - [Connecting an element to the store](#connecting-an-element-to-the-store)
    - [Dispatching actions](#dispatching-actions)
- [Case study walkthrough](##case-study-walkthrough)
  - [Example 1: Counter](#example-1-counter)
  - [Example 2: Shopping Cart](#example-2-shopping-cart)
  - [Routing](#routing)
- [Patterns](#patterns)
  - [Connecting dom events to action creators](#connecting-dom-events-to-action-creators)
    - [Manually](#manually)
    - [Automatically](#automatically)
  - [Reducers: slice reducers](#reducers-slice-reducers)
  - [Avoid duplicate state](#avoid-duplicate-state)
  - [How to make sure third-party components don't mutate the state](#how-to-make-sure-third-party-components-dont-mutate-the-state)
  - [Routing](#routing-1)
  - [Lazy Loading](#lazy-loading)
  - [Replicating the state for storage](#replicating-the-state-for-storage)

## General principles
[Redux](https://redux.js.org/) is a small state management container, that is view agnostic and widely used. It is centered around the idea of separating your application logic (the application state) from your view layer, and having the store as a single source of truth for the application state. We recommend reading some of the [Redux docs](https://redux.js.org/) for a good introduction, as well as this awesome [cartoon intro to Redux](https://code-cartoons.com/a-cartoon-intro-to-redux-3afb775501a6) by Lin Clark.

One of the neat features of Redux is that it lets you do [time travel debugging](https://code-cartoons.com/hot-reloading-and-time-travel-debugging-what-are-they-3c8ed2812f35); in particular, we use this [Chrome extension](https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd?hl=en).

### Some definitions

When working with Redux, a bunch of words get used a lot, that might sound confusing at first:
- **state**: this is the information contained by the app. In general, any element properties that you use could be considered the state.
- **store**: the thing that holds the state. You can only have one store, and it is the source of all truth. You can get the state from the store via `store.getState()`.
- **actions**: represent the facts about “what happened” to the state, and are how the application communicates with the store (to tell it that something needs to be updated. You send an action `MEOW` to the store using `store.dispatch(MEOW)`.
- **action creators**: functions that create actions. They return an action, which can then be dispatched. You use an action creator in your app via `store.dispatch(doAMeow())`. Action creators can also dispatch asynchronous actions, which makes them very useful!
- **reducers**: describe how the state updates as a result of an action. They are functions that take the old state, an action, and (after doing some work), return a brand new copy of the state, with the right updates applied.

### Naming conventions
We recommend structuring your application code as follows:
```
src
├── store.js
├── actions
│   └── counter.js
|   └── ...
├── reducers
│   └── counter.js
|   └── ...
└── components
    └── simple-counter.js
    └── my-app.js
    └── ...
```

- Action creators and reducers can (but aren't required to) have the same name, and be named after the slice of the app's data they deal with. For example, a shopping app could have the following reducers:
  - `app.js` to deal with big picture app-related data, such as online/offline status, route paths, etc.
  - `products.js` for the list of products you can purchase.
  - `cart.js` for the shopping cart.
  - etc.
- Action type
  - Verb(+noun, optional), present tense: `ADD_TODO`, `FETCH`, `FETCH_ITEMS`, `RECEIVE_ITEMS`.
  - These should represent what's about to happen, not what has already happened.
- Action creator
  - Same as action type, camel cased (`addTodo` -> `ADD_TODO`).
- Selector
  - `categorySelector`/`itemsSelector` vs `getCategory`/`getItems` (to distinguish that one is a selector, whereas the `get*` methods could just be non-memoized selectors).

## Connecting elements to the store

### What to connect
Generally, anything that needs to have direct access to the store data should be considered a **connected** element. This includes both updating the state (by calling `store.dispatch`), or consuming the state (by calling `store.subscribe`). However, if the element only needs to consume store data, it could receive this data via data bindings from a connected parent element. If you think about a shopping cart example: the cart itself needs to be connected to the store, since “what's in the cart” is part of the application's state, but the reusable elements that are rendering each item in the cart don't need to be connected (since they can just receive their data through a data binding).

Since this is a very application specific decision, one way to start looking at it is to try connecting your lazy-loaded elements, and then go up or down one level from there. That might end up looking something like:
<img width="785" alt="screen shot 2018-01-25 at 12 22 39 pm" src="https://user-images.githubusercontent.com/1369170/35410478-7373c98a-01ca-11e8-9f7f-4b95c8a4f47c.png">

In this example, only `my-app` and `my-view1` are connected. Since `a-element` is more of a reusable component rather than an application level component, even if it needs to update the application's data, it will communicate this via a DOM event, like [this](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/counter-element.js#L56).

## How to connect
If you want to follow along with actual code, we've included a basic Redux [counter example](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-view2.js) in `pwa-starter-kit`.

### Creating a store
If you want to create a simple store, that is not lazy loading reducers, then you probably want something like this:
```js
export const store = createStore(
  reducer,
  compose(applyMiddleware(thunk))
);
```
Note that this still isn't the most basic store you can have, since it adds the [redux-thunk](https://github.com/gaearon/redux-thunk) middleware -- this allows you to dispatch async actions, which for any medium-complexity app is a requirement. In most cases however, you're going to be lazy loading routes, and they should lazy load their reducers, so you want a store that can replace its reducers after it's been initialized, which is why `pwa-starter-kit` initializes the store with a `lazyReducerEnhancer` and the `redux-thunk`:

```js
export const store = createStore(
  (state, action) => state,
  compose(lazyReducerEnhancer(combineReducers), applyMiddleware(thunk))
);
```

You can find more details on the `lazyReducerEnhancer` in the [Lazy Loading](#lazy-loading) section.

### Connecting an element to the store
An element that is connected should call `store.subscribe` in the constructor, and only update its properties in the change listener passed as the first and only argument (if it needs to). We use a mixin ([`connect-mixin.js`](https://github.com/Polymer/pwa-helpers/blob/master/connect-mixin.js)) from `pwa-helpers` that does all the connection boilerplate for you, and expects you to implement the `stateChanged` method. Example use:

```js
import { LitElement, html } from 'lit-element';
import { connect } from  '@polymer/pwa-helpers/connect-mixin.js';
import { store } from './store/store.js';

class MyElement extends connect(store)(LitElement) {
  static get is() { return 'my-element'; }

  static get properties() { return {
    clicks: { type: Number },
    value: { type: Number }
  }}

  render() {
    return html`...`;
  }

  // If you don't implement this method, you will get a
  // warning in the console.
  stateChanged(state) {
    this.clicks = state.counter.clicks;
    this.value = state.counter.value;
  }
}
```

Note that `stateChanged` gets called **any** time the store updates, not when only the things you care about update. So in the example above, `stateChanged` could be called multiple times without `state.counter.clicks` and `state.counter.value` ever changing. If you're doing any expensive work in `stateChanged`, such as transforming the data from the Redux store (with something like `Object.values(state.data.someArray)`), consider moving that logic into the `render()` function (which is called only if the properties update), using a selector, or adding some form of dirty checking:

```js
stateChanged(state) {
  if (this.clicks !== state.counter.clicks) {
    this.clicks = state.counter.clicks;
  }
}
```
### Dispatching actions
If an element needs to dispatch an action to update the store, it should call an action creator:

```js
import { increment } from './store/actions/counter.js';

firstUpdated() {
  // Every time the display of the counter updates, save
  // these values in the store
  this.addEventListener('counter-incremented', function() {
    store.dispatch(increment());
  });
}
```

Action creators say what the system _should_ do, not what it has already done. This action creator could return a synchronous action:
```js
export const increment = () => {
  return {
    type: INCREMENT
  };
};
```

An asynchronous one,
```js
export const increment = () => (dispatch, getState) => {
  // Do some sort of work.
  const state = getState();
  const howMuch  = state.counter.doubleIncrement ? 2 : 1;
  dispatch({
      type: INCREMENT,
      howMuch,
    });
  }
};
```
Or dispatch the result of another action creator:
```js
export const increment = () => (dispatch, getState) => {
  // Do some sort of work.
  const state = getState();
  const howMuch = state.counter.doubleIncrement? 2 : 1;
  dispatch(incrementBy(howMuch));
};
```

## Case study walkthrough
The goal of this walkthrough is to demonstrate how to get started with Redux, by explaining how we added 2 of the standard Redux examples in the `pwa-starter-kit` template app.

### Example 1: Counter
The [counter](https://redux.js.org/docs/introduction/Examples.html#counter-vanilla) example is very simple: we're going to add a counter custom element (that you can imagine is a reusable, third party element) to `my-view2.js`. This example is very detailed, and goes through every line of code that needs to change. If you want a higher level example, check out Example 2. The interaction between the elements, the action creators, action, reducers and the store looks something like this:
<img width="886" alt="screen shot 2018-01-25 at 12 44 24 pm" src="https://user-images.githubusercontent.com/1369170/35411408-7edd9d84-01cd-11e8-9044-d817dc1967da.png">

#### `counter.element.js`
This is a [plain element](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/counter-element.js) that's not connected to the Redux store. It has two properties, `clicks` and `value`, and 2 buttons that increment or decrement the value (and always increment `clicks`).

#### `my-view2.js`
This element is an [app-level element](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-view2.js) (as opposed to a reusable element), and it's connected to the store. This means that it will be able to read and update the application's state -- in particular, the value/clicks properties from `counter-element`. We need to:
- Add `counter-element` to this view. Note that we pass the state **down** to the element, since the state lives in the Redux store, not in the element. We do this because even though the `counter-element` updates _its_ internal properties every time you click any of the buttons, that may not necessarily be the true state of the app -- imagine a more complex example, where a different view is also updating the value of this counter. The store is then the only source of truth for the data, and the `counter-element` must always reflect that.

```html
<counter-element value="${props._value}" clicks="${props._clicks}"></counter-element>
```
- To demonstrate that it is the Redux store driving the state, and not `counter-element`'s internal, hidden state, we also added the `clicks` property to the circle in the header:

```html
<div class="circle">${props._clicks}</div>
```
- Connect the view to the store:

```js
import { connect } from '@polymer/pwa-helpers/connect-mixin.js';
class MyView2 extends connect(store)(LitElement) {
...
}
```
- Lazily load the reducers. You don't _have_ to do this (especially if you're prototyping), but since this view is lazy loaded, its reducers should be as well (to follow the “don't do anything until you actually need it” PRPL approach).

```js
import counter from '../reducers/counter.js';
store.addReducers({
  counter
});
```
- Implement the `stateChanged` method, which is called when anything is updated in the store. Since the store is the source of truth for the 2 properties (rather than `counter-element` itself), any time the Redux store updates we need to update any local properties; this keeps `counter-element` up to-date:

```js
stateChanged(state) {
  this._clicks = state.counter.clicks;
  this._value = state.counter.value;
}
```

- Note that here both `_clicks` and `_value` start with an underscore, which means they are protected -- we don't expect anyone from _outside_ the `<my-view2>` element to want to modify them.
- In turn, when `counter-element` updates its value (because the buttons were clicked), we listen to its change events and dispatch an action creator to the store:

```js
this.addEventListener('counter-incremented', function() {
  store.dispatch(increment());
})
```
- `increment` is an action creator. It is defined in `src/actions/counter.js`, and dispatches an `INCREMENT` action (and the same for `decrement`). When the store receives this action, it needs to update the state. This is done in the `src/reducers/counter.js` reducer.

### Example 2: Shopping Cart
The [shopping cart example](https://redux.js.org/docs/introduction/Examples.html#shopping-cart) is a little more complex. The main view element (`my-view3.js`) contains `<shop-products>`, a list of products that you can choose from, and `<shop-cart>`, the shopping cart. You can select products from the list to add them to the cart; each product has a limited stock, and can run out. You can perform a checkout in the cart, which has a probability of failing (which in real life could fail because of credit card validation, server errors, etc). It looks like this:
<img width="819" alt="screen shot 2018-01-25 at 12 50 22 pm" src="https://user-images.githubusercontent.com/1369170/35411643-53dccc62-01ce-11e8-8799-6a48da8901a5.png">

#### `my-view3.js`
This is a connected element that displays both the list of products, the cart, and the Checkout button. It is only connected because it needs to display conditional UI, based on whether the cart has any items (i.e. show a Checkout button or not). This could've been an unconnected element if the Checkout button belonged to the cart, for example.
- Pressing the Checkout button calls the `checkout` action creator. In this action creator you would do any credit cart/server validations, so if the operation cannot be completed, you would fire `CHECKOUT_FAILURE` here. We simulate that by flipping a coin, and conditionally dispatching the async action.
- If the checkout action succeeds, then the `products` object will be updated (with the new stock), and the `cart` will be reset to its initial value (empty).
- One thing to note: in the `src/reducers/shop.js` reducer we use a lot of slice reducers. A slice reducer is responsible for a slice (yes, really) of the whole store (for example, one product item) and updating it. To update the available stock for a specific item ID in the store, we call the `products` slide reducer (to reduce the whole store to just the products), then the `product` slice reducer for the product ID passed in the action.

#### `shop-products.js`
This element gets the list of products from the store by dispatching the `getAllProducts` action creator. When the store is updated (by fetching the products from a service, for example), its `stateChanged` method is called, which populates a `products` object. Finally, this object is used to render the list of products.
- `getAllProducts` is an action creator that simulates getting the data from a service (it doesn't, it gets it from a local object, but that's where you would out that logic). When the data is ready, it dispatches an async `GET_PRODUCTS` action.
- Note that whenever a product is added to the cart, the `addToCart` action creator is dispatched. This updates both the `products` and `cart` objects in the Redux store, which will in turn call `stateChanged` in both `shop-products` and `shop-cart`.
- Adding an item to the cart dispatches the `addToCart` action creator, which first double-checks the stock (on the Redux side) before actually adding the item to the cart. This is done to avoid any front-end hacks where you could add more items to the cart than in the stock 😅

#### `shop-cart.js`
Similar to `shop-products`, this element is also connected to the store and observes both the `products` and `cart` state. One of the Redux rules is that there should be only one source of truth, and you should not be duplicating data. For this reason, `products` is the source of truth (that contains all the available items), and `cart` contains the indexes, and number of, items that have been added to the cart.

### Routing
We use a very simple (but flexible) redux-friendly router, that uses the window location and stores it in store. We do this by using the `installRouter` helper method provided from the `pwa-helpers` package:
```js
import { installRouter } from '@polymer/pwa-helpers/router.js';
firstUpdated() {
  installRouter((location) => this._locationChanged(location));
}
```

## Patterns

### Connecting DOM events to action creators
If you don't want to connect every element to the store (and you shouldn't), unconnected elements will have to communicate the need to update the state in the store.

#### Manually
You can do this manually by firing event. If `<child-element>` is unconnected but displays and modifies a property `foo`:
- Whenever foo is modified, `<child-element>` fires an event:

```js
_onIncrement() {
  this.value++;
  this.dispatchEvent(new CustomEvent('counter-incremented');
}
```
- The connected parent of `<child-element>` can listen to this event and dispatch an action to the store:

```html
<counter-element on-counter-incremented="${() => store.dispatch(increment())}"
```

Or in JavaScript,
```js
firstUpdated() {
  this.addEventListener('counter-incremented', function() {
    store.dispatch(increment());
  });
}
```

#### Automatically
Alternatively, you can write a helper to automatically convert any Polymer `foo-changed` property change event into a Redux action. Note that this requires the `<child-element>`'s properties to be notifying (i.e. have `notify: true`), which isn't necessarily true of all third party elements out there. Here's an [example](https://gist.github.com/kevinpschaaf/995c9d1fd0f58fe021b174c4238b38c3#file-5-connect-element-mixin-js) of that.

### Reducers: slice reducers

To make your app more modular, you can split the main state object into parts ("slices") and have smaller "slice reducers" operate on each part ([read more about slice reducers](https://redux.js.org/docs/recipes/reducers/SplittingReducerLogic.html)). With the `lazyReducerEnhancer`, your app can lazily add slice reducers as necessary (e.g. add the `counter` slice reducer when `my-view2.js` is imported since only `my-view2` operates on that part of the state).

**`src/store.js:`**
```js
export const store = createStore(
  (state, action) => state,
  compose(lazyReducerEnhancer(combineReducers), applyMiddleware(thunk))
);
```

**`src/components/my-view2.js:`**
```js
// This element is connected to the Redux store.
import { store } from '../store.js';

// We are lazy loading its reducer.
import counter from '../reducers/counter.js';
store.addReducers({
  counter
});
```

### Avoid duplicate state

We avoid storing duplicate data in the state by using the [Reselect](https://github.com/reactjs/reselect) library. For example, the state may contain a list of items, and one of them is the selected item (e.g. based on the URL). Instead of storing the selected item separately, create a selector that computes the selected item:

```js
import { createSelector } from 'reselect';

const locationSelector = state => state.location;
const itemsSelector = state => state.items;

const selectedItemSelector = createSelector(
  locationSelector,
  itemsSelector,
  (location, items) => items[location.pathname.slice(1)]
);

// To get the selected item:
console.log(selectedItemSelector(state));
```

To see an example of this, check out the cart example's [cart quantity selector](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-view3.js#L107) or the [item selector](https://github.com/PolymerLabs/polymer-redux-hn/blob/master/src/components/hn-item.ts#L59) from the [Redux-HN](https://github.com/PolymerLabs/polymer-redux-hn) sample app. In both examples, the selector is actually defined in a reducer, since it's being used both on the Redux side, as well as in the view layer.

### How to make sure third-party components don't mutate the state
Most third-party components were not written to be used in an immutable way, and are not connected to the Redux store so you can't guarantee that they will not try to update the store. For example, `paper-input` has a `value` property, that it updates based on internal actions (i.e. you typing, validating, etc). To make sure that elements like this don't update the store:
- Use one-way data bindings to pass primitives (Strings, Numbers, etc) down to the element.
  - `<paper-input value="${foo}"></paper-input>`
  - Because it's a primitive value, paper-input receives a copy of `foo`. When it updates `foo`, it only updates **its** copy, not the actual property in the store
  - Listen to `foo-changed` events outside the element, and dispatch an action to update the store from there ([example](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-view2.js#L51)).
- Since arrays/objects are mutable, pass down a **copy** of an array or object down to the element:
  - `<other-input data="${_copy(fooArray)}"></other-input>`
  - `<other-input data="${_deepCopy(fooObj)}"></other-input>`
  - Listen to change events as above to dispatch an action to update the store.

### Routing
With Redux, you're basically on your own for routing. However, we have provided a [helper router](https://github.com/Polymer/pwa-helpers/blob/master/src/router.ts) to get you started. Our suggestion is to update the location state based on `window.location`. That is, whenever a link is clicked (or the user navigates back), an action is dispatched to update the state based on the location. This works well with time-travel debugging - jumping to a previous state doesn't affect the URL bar or history stack.

Example of installing and using the router:

```js
// ...
import { installRouter } from '@polymer/pwa-helpers/router.js';

class MyApp extends connect(store)(LitElement) {
    // ...
    firstUpdated() {
      installRouter((location) => this._locationChanged(location));

      // The argument passed to installRouter is a callback. If you don't
      // have any other work to do other than dispatching an action, you
      // can also write something like:
      // installRouter((location) => store.dispatch(navigate(location.pathname)));
    }

    _locationChanged(location) {
      // What action creator you dispatch and what part of the location
      // will depend on your app.
      store.dispatch(navigate(location.pathname));

      // Do other work, if needed, like closing drawers, etc.
    }
  }
}
```

### Lazy loading
One of the main aspects of the PRPL pattern is lazy loading your application's components as they are needed. If one of these lazy-loaded elements is connected to the store, then your app needs to be able to lazy load that element's reducers as well.

There are many ways in which you can do this. We've implemented one of them as a [helper](https://github.com/Polymer/pwa-helpers/blob/master/src/lazy-reducer-enhancer.ts), which can be added to the store:
```js
import lazyReducerEnhancer from '@polymer/pwa-helpers/lazy-reducer-enhancer.js';

// Not-lazy loaded reducers that are initially loaded.
import app from './reducers/app.js';

export const store = createStore(
  (state, action) => state,
  compose(lazyReducerEnhancer, applyMiddleware(thunk))
);

// Initially loaded reducers.
store.addReducers({
  app
});
```

In this example, the application will boot up and install the `app` reducers, but no others. In your lazy-loaded element, to load its reducer, all you need to do is call `store.addReducers`:
```js
// If this element was lazy loaded, we must also install its reducer
import { someReducer } from './store/reducers/one.js';
import { someOtherReducer } from './store/reducers/two.js';

// Lazy-load the reducer
store.addReducers({someReducer, someOtherReducer});

class MyElement extends ... {
…
}
```

### Replicating the state for storage
One of the things your app might want to do is save the state of the app in a storage location (like a database, or `localStorage`. Redux is very useful for this, since you basically just need to install a new reducer to subscribe to the state, that will dump the state into storage.

To do this, we can first create two functions, called `saveState` and `loadState`, to read to/from storage:
```js
export const saveState = (state) => {
  let stringifiedState = JSON.stringify(state);
  localStorage.setItem(MY_KEY, stringifiedState);
}
export const loadState = () => {
  let json = localStorage.getItem(MY_KEY) || '{}';
  let state = JSON.parse(json);

  if (state) {
    return state;
  } else {
    return undefined;  // To use the defaults in the reducers
  }
}
```

Now, in `store.js`, we basically want to use the result of `loadState()` as the default state in the store, and call `saveState()` every time the store updates:

```js
export const store = createStore(
  (state, action) => state,
  loadState(),  // If there is local storage data, load it.
  compose(lazyReducerEnhancer(combineReducers), applyMiddleware(thunk))
);

// This subscriber writes to local storage anytime the state updates.
store.subscribe(() => {
  saveState(store.getState());
});
```

That's it! If you want to see a demo of this in a project, the [**Flash-Cards**](https://github.com/notwaldorf/flash-cards/blob/master/src/localStorage.js) app implements this pattern.
-->

このページでは、Reduxを使用してアプリケーションの状態を管理するために必要な手順について説明します。

## 目次

- [一般な原則](#general-principles)
  - [いくつかの定義](#some-definitions)
  - [命名規則](#naming-conventions)
- [ストアに要素を接続する](#connecting-elements-to-the-store)
  - [接続するもの](#what-to-connect)
  - [接続方法](#how-to-connect)
    - [ストアの作成](#creating-a-store)
    - [ストアへの要素の接続](#connecting-an-element-to-the-store)
    - [ディスパッチアクション](#dispatching-actions)
- [ケーススタディをみる](#case-study-walkthrough)
  - [例1: カウンター](#example-1-counter)
  - [例2: ショッピングカート](#example-2-shopping-cart)
  - [ルーティング](#routing)
- [パターン](#patterns)
  - [アクションクリエイターにDOMイベントを接続する](#connecting-dom-events-to-action-creators)
    - [手動](#manually)
    - [自動](#automatically)
  - [Reducers: スライスする](#reducers-slice-reducers)
  - [重複状態を避ける](#avoid-duplicate-state)
  - [サードパーティのコンポーネントが状態を変更しないようにする方法](#how-to-make-sure-third-party-components-dont-mutate-the-state)
  - [ルーティング](#routing-1)
  - [遅延読み込み](#lazy-loading)
  - [ストレージの状態を複製する](#replicating-the-state-for-storage)

<a id="general-principles">

## 一般な原則

[Redux](https://redux.js.org/)は小さな状態管理コンテナで、ビューとは関係なく広く使われています。これは、アプリケーションロジック（アプリケーション状態）をビューレイヤーから分離し、そのストアをアプリケーション状態の真の単一のソースとして持つという考えを中心にしています。私たちは[Redux docs](https://redux.js.org/)の一部と、Lin Clark氏によるこの素晴らしい[Reduxの紹介漫画](https://code-cartoons.com/a-cartoon-intro-to-redux-3afb775501a6)を読むことをお勧めします。

Reduxのすばらしい機能の1つは、[タイムトラベルデバック](https://code-cartoons.com/hot-reloading-and-time-travel-debugging-what-are-they-3c8ed2812f35)を実行できることです。特に、この[Chrome拡張機能](https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd?hl=ja)を使います。

<a id="some-definitions">

### いくつかの定義

Reduxを使って作業するとき、たくさんの言葉が多用されます。最初は混乱するかもしれません:
- **state**: これはアプリに含まれている状態です。一般に、使用する要素のプロパティはすべて状態と見なすことができます。
- **store**: 状態を保持するもの。storeは1つしかできず、すべての正しい状態の源です。 `store.getState()`を介してストアから状態を取得できます。
- **actions**: 状態に"何が起ったか"を伝え、アプリがどのようにstoreと交信するのかを定義するもの(よって何かを更新する必要があることを伝えます。 `MEOW`アクションは `store.dispatch(MEOW)` を使ってストアに送られます。
- **action creators**: アクションを作成する関数。彼らはアクションを返し、それをディスパッチすることができます。 あなたは `store.dispatch(doAMeow())' を介してあなたのアプリでアクションクリエータを使用します。 アクションクリエイターは、非同期アクションをディスパッチすることもでき、非常に便利です！
- **reducers**: アクションの結果として状態がどのように更新されるかを記述します。 それらは、既存の状態とアクション、および（いくつかの作業を行った後）、適切な更新を適用して状態の新しいコピーを返します。

<a id="naming-conventions">

### 命名規則

アプリケーションコードを次のように配置することをお勧めします:

```
src
├── store.js
├── actions
│   └── counter.js
|   └── ...
├── reducers
│   └── counter.js
|   └── ...
└── components
    └── simple-counter.js
    └── my-app.js
    └── ...
```

- アクションクリエイターとレデューサーは同じ名前を持つことができます（必須ではありません）。たとえば、ショッピングアプリでは次のようなreducerを使用できます:
  - `app.js` オンライン/オフラインステータス、ルートパスなど、大きな画像アプリ関連のデータを処理する
  - `products.js` 購入できる製品のリスト用
  - `cart.js` ショッピングカート用
  - その他...
- アクションタイプ
  - 動詞（+名詞、オプション）、使用されているのは: `ADD_TODO`, `FETCH`, `FETCH_ITEMS`, `RECEIVE_ITEMS`.
  - これらは、起こっていることを表し、すでに起こっていることを表すものではありません。
- アクションクリエイター
  - アクションタイプと同じですが、 キャメルケースで表します (`addTodo` -> `ADD_TODO`).
- セレクタ
  - `categorySelector`/`itemsSelector` vs `getCategory`/`getItems` (それらがセレクタであることを区別するために、 `get*`メソッドを非メモ化セレクタとして表します).

<a id="connecting-elements-to-the-store">

## ストアに要素を接続する

<a id="what-to-connect">

### 接続するとは

一般に、ストアデータに直接アクセスする必要があるものは、**connected**要素とみなす必要があります。 これには、状態を更新する（ `store.dispatch`を呼び出す）か状態監視する（`store.subscribe`を呼び出す）の両方が含まれます。

ただし、要素がストアデータを監視する必要がある場合、接続された親要素からのデータバインディングを介してこのデータを受け取ることができます。 ショッピングカートの例について考えると、カートの内容はアプリケーションの状態の一部なので、カート自体をストアに接続する必要がありますが、接続する必要のあるカート内の各アイテムをレンダリングする再利用可能な要素は必要ありません（データバインディングを介してデータを受け取るだけなので）

これは非常にアプリケーション固有の決定なので、それを見始める1つの方法は、遅延ロードされた要素を接続してそこから1つ上または下に移動することです。それは下記のように見えるかもしれません:
<img width="785" alt="screen shot 2018-01-25 at 12 22 39 pm" src="https://user-images.githubusercontent.com/1369170/35410478-7373c98a-01ca-11e8-9f7f-4b95c8a4f47c.png">

この例では、 `my-app`と` my-view1`だけが接続されています。 `a-element`はアプリケーションレベルのコンポーネントではなく、再利用可能なコンポーネントであるため、アプリケーションのデータを更新する必要がある場合でも、 [これ](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/counter-element.js#L56)のようなDOMイベントを介してこれを伝えます。

<a id="how-to-connect">

## 接続方法

実際のコードに従う場合、基本的なRedux [カウンターサンプル](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-view2.js)が`pwa-starter-kit`に含まれています。

<a id="creating-a-store">

### ストアの作成

シンプルなストアを作成したい場合、それは遅延読み込みするreducerではないので、おそらくこのようなものが必要です:

```js
export const store = createStore(
  reducer,
  compose(applyMiddleware(thunk))
);
```

[redux-thunk](https://github.com/gaearon/redux-thunk)ミドルウェアが追加されているため、これはまだ最も基本的なストアではないことに注意してください。これにより、非同期アクションをディスパッチできます。これは中程度の複雑さのアプリケーションでは必須です。
しかし、ほとんどの場合、怠惰な遅延読み込みのルートになるでしょうし、reducerも遅延してロードする必要があります。そのため、初期化後にreducerを置き換えることができるストアが必要なので、 `pwa-starter-kit`では`lazyReducerEnhancer`と` redux-thunk`を使います:

```js
export const store = createStore(
  (state, action) => state,
  compose(lazyReducerEnhancer(combineReducers), applyMiddleware(thunk))
);
```

`lazyReducerEnhancer`については[遅延読み込み](#lazy-loading)により詳細な説明があります。

<a id="connecting-an-element-to-the-store">

### ストアへの要素の接続

接続される要素は、コンストラクタで `store.subscribe`を呼び出す必要があり、リスナーからの変更通知によって直ちにその通知された更新部分に限ってプロパティを更新します（その要があれば）。`pwa-helpers`にはmixin([`connect-mixin.js`](https://github.com/Polymer/pwa-helpers/blob/master/connect-mixin.js))が用意されており、そこですべての接続がされ、`stateChanged`メソッドを実装することにより利用できます。サンプルとしては:

```js
import { LitElement, html } from '@polymer/lit-element/lit-element.js'
import { connect } from  '@polymer/pwa-helpers/connect-mixin.js';
import { store } from './store/store.js';

class MyElement extends connect(store)(LitElement) {
  static get is() { return 'my-element'; }

  static get properties() { return {
    clicks: { type: Number },
    value: { type: Number }
  }}

  render() {
    return html`...`;
  }

  // もしこのメソッドが実装されていないと
  // コンソールに警告が表示されます。
  stateChanged(state) {
    this.clicks = state.counter.clicks;
    this.value = state.counter.value;
  }
}
```

If you're doing any expensive work in `stateChanged`, such as transforming the data from the Redux store (with something like `Object.values(state.data.someArray)`), consider moving that logic into the `render()` function (which is called only if the properties update), using a selector, or adding some form of dirty checking

`stateChanged`はストア更新時にエレメントの表示更新とは関係なく**毎回**呼び出されるので注意してください。 したがって、上記の例では、 `stateChanged`は`state.counter.clicks`と `state.counter.value`が変わることなく複数回呼び出されています。Reduceストアから(`Object.values(state.data.someArray)`のようなもので)データを変換するなど、 `stateChanged`で高価な作業をしている場合は、そのロジックを`render()`関数（プロパティが更新された場合のみ呼び出されます）に移動し、セレクタか何らかのダーティチェック(元データとの変更確認)を追加してください:

```js
stateChanged(state) {
  if (this.clicks !== state.counter.clicks) {
    this.clicks = state.counter.clicks;
  }
}
```

<a id="dispatching-actions">

### ディスパッチアクション

要素がストアを更新するアクションをディスパッチする必要がある場合、アクションクリエータを呼び出す必要があります:

```js
import { increment } from './store/actions/counter.js';

firstUpdated() {
  // カウンタの表示が更新されるたびに、
  // これらの値をストアに保存します
  this.addEventListener('counter-incremented', function() {
    store.dispatch(increment());
  });
}
```

アクションクリエータは、システムが何をしているのか、それが何をしているのかを伝えます。このアクションクリエータは、同期アクションを返すことができます:

```js
export const increment = () => {
  return {
    type: INCREMENT
  };
};
```

非同期の場合は

```js
export const increment = () => (dispatch, getState) => {
  // ここで何か処理
  const state = getState();
  const howMuch  = state.counter.doubleIncrement ? 2 : 1;
  dispatch({
      type: INCREMENT,
      howMuch,
    });
  }
};
```

もしくは他のアクションクリエータに結果をディスパッチ:

```js
export const increment = () => (dispatch, getState) => {
  // ここで何か処理
  const state = getState();
  const howMuch = state.counter.doubleIncrement? 2 : 1;
  dispatch(incrementBy(howMuch));
};
```

<a id="case-study-walkthrough">

## ケーススタディをみる

このチュートリアルの目的は、標準的なReduxサンプルの2つを `pwa-starter-kit`テンプレートアプリケーションにどのように追加したのかを説明することによって、Reduxを始める方法を示します。

<a id="example-1-counter">

### 例1: カウンター

[counter](https://redux.js.org/docs/introduction/Examples.html#counter-vanilla)の例は非常に簡単です: counterカスタム要素（これは再利用可能であるものとして）を `my-view2.js`にコピーします。 この例は非常に詳細で、変更する必要があるコードのすべての行を示しています。上位レベルの例が必要な場合は、例2を参照してください。要素、アクションクリエータ、アクション、reducer、ストア間の相互作用は、次のようになります:
<img width="886" alt="screen shot 2018-01-25 at 12 44 24 pm" src="https://user-images.githubusercontent.com/1369170/35411408-7edd9d84-01cd-11e8-9044-d817dc1967da.png">

#### `counter.element.js`

これはReduxストアに接続していない[普通のエレメント](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/counter-element.js)です。Reduxストアに接続されていないので、`clicks`と`value`の2つのプロパティとカウントアップとカウントダウンをする2つのボタンをもっています(`clicks`は増え続けます)。

#### `my-view2.js`

このエレメントは[アプリケーションとしてのエレメント](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-view2.js) (再利用可能な要素とは対照的に)で、ストアに接続しています。これは、アプリケーションの状態を読み取り、更新できることを意味します。特に、`counter-element`からのvalueとclicksプロパティです。 必要な機能のは:

- このビューに `counter-element`を追加してください。 状態は要素ではなくReduxストアに存在するため、状態を要素に**落して**渡すことに注意してください。 これは、ボタンのいずれかをクリックするたびに `counter-element`が内部プロパティを更新するにもかかわらず、アプリとしては正しい状態が必ずしも必要ないことになります。 例えばもっと複雑な例としては別のビューもこのカウンタの値を更新する場合です。 ストアはデータの唯一の正しい情報元であり、 `counter-element`は常にそれを反映しなければなりません。

```html
<counter-element value="${props._value}" clicks="${props._clicks}"></counter-element>
```
- Reduxストアに応じて状態が変化し、`counter-element`の閉じられて隠されたものではないことを示す為に`clicks`プロパティを追加しましたcircleクラスに:

```html
<div class="circle">${props._clicks}</div>
```
- ビューをストアに接続すには:

```js
import { connect } from '@polymer/pwa-helpers/connect-mixin.js';
class MyView2 extends connect(store)(LitElement) {
...
}
```
- reducerを遅延読み込みします。これを必ず行う必要はありません（特にプロトタイピングの場合）。しかし、このビューは遅延ロードされているので、(PRPLのアプローチでは)reducerも同様にしなければなりません。

```js
import counter from '../reducers/counter.js';
store.addReducers({
  counter
});
```
- ストア内で何かが更新されたときに呼び出される `stateChanged`メソッドを実装します。ストアは2つのプロパティ（counter要素自体ではなく）の真のソースなので、Reduxストアの更新時にはローカルプロパティを更新する必要があります。これは `counter-element`を最新の状態に保ちます:

```js
stateChanged(state) {
  this._clicks = state.counter.clicks;
  this._value = state.counter.value;
}
```

- ここで `_clicks`と`_value`の両方がアンダースコアで始まることに注意してください。つまり、それらは保護されています - 私たちは `<my-view2>`要素の外側でそれらを変更してほしくないと考えます。
- 次に、`counter-element`がその値を更新すると（ボタンがクリックされたため）、その変更イベントを聞き、アクションクリエータをストアにディスパッチします:

```js
this.addEventListener('counter-incremented', function() {
  store.dispatch(increment());
})
```
- `increment`はアクションクリエータです。それは `src/actions/counter.js`で定義され、` INCREMENT`アクションを送ります（`decrement`も同じ）。ストアがこのアクションを受け取ると、状態を更新する必要があります。これは `src/reducers/counter.js`でreducerが交信します。

<a id="example-2-shopping-cart">

### 例1: ショッピングカート

[ショッピングカートの例](https://redux.js.org/docs/introduction/Examples.html#shopping-cart)はもう少し複雑です。メインビュー要素（`my-view3.js`）には、選択可能な製品のリストである`<shop-products> `と、ショッピングカートである`<shop-cart>`があります。リストから商品を選択してカートに追加することができます。各商品には在庫が限られており、在庫がなくなります。カート内でチェックアウトを実行することができます。これは、失敗する可能性があります（実際の生活では、クレジットカードの確認、サーバーエラーなどにより失敗する可能性があります）。これは次のようになります。
<img width="819" alt="screen shot 2018-01-25 at 12 50 22 pm" src="https://user-images.githubusercontent.com/1369170/35411643-53dccc62-01ce-11e8-8799-6a48da8901a5.png">

#### `my-view3.js`

これは、商品のリスト、カート、および[チェックアウト]ボタンの両方を表示する接続された要素です。カートにアイテムがあるかどうか（チェックアウトボタンを表示するかどうかなど）に基づいて条件付きUIを表示する必要があるため、接続されているだけです。Checkoutボタンがカートに属していた場合、これは接続されていない要素となります。たとえば:

- Checkoutボタンを押すと、 `checkout`アクションクリエータが呼び出されます。このアクションクリエータでは、カート/サーバーの妥当性チェックを行うので、操作が完了できない場合は、ここで `CHECKOUT_FAILURE`を実行します。私たちはコインを反転し、非同期アクションを条件付きでディスパッチすることでそれをシミュレートします。
- チェックアウトが成功した場合、 `products`オブジェクトは更新され（新しい在庫で）、`cart`はその初期値（空）にリセットされます。
- 注目すべきことは、 `src/reducers/shop.js`のreducerでは、多くのスライスreducerを使用しています。スライスreducerは、店舗全体（たとえば、1つの商品アイテム）のスライス（実際には）とそれを更新する役割を担います。店舗内の特定の商品IDの在庫を更新するには、商品全体を減らすために `products` スライド reducerを呼び出し、次にアクションで渡された商品IDの`product` スライス reducerを呼び出します。

#### `shop-products.js`

この要素は、 `getAllProducts`アクションクリエータをディスパッチすることによってストアからプロダクトのリストを取得します。 ストアが更新されると（例えば、サービスからプロダクトを取得することによって）、 `stateChanged`メソッドが呼び出され、`products`オブジェクトが生成されます。 最後に、このオブジェクトは製品のリストをレンダリングするために使用されます。
- `getAllProducts`は、サービスからデータを取得することをシミュレートするアクションクリエータです（ローカルオブジェクトから取得しますが、そのロジックから抜け出す場所です）。 データが準備できたら、非同期の `GET_PRODUCTS`アクションを送出します。
- 商品がカートに追加されるたびに、 `addToCart`アクションクリエータがディスパッチされることに注意してください。 これにより、Reduxストア内の `products`オブジェクトと` cart`オブジェクトの両方が更新され、 `shop-products`と`shop-cart`の両方で `stateChanged`が呼び出されます。
- アイテムをカートに追加すると、 `addToCart`アクションクリエータがディスパッチされます。アクションクリエータは、アイテムを実際にカートに追加する前に、（Redux側の）在庫を最初に再確認します。これは、在庫よりも多くのアイテムをカートに追加できるフロントエンドハッキングを避けるために行われます 😅

#### `shop-cart.js`

`shop-products`と同様に、この要素もストアに接続され、`products`と `cart`の両方の状態を監視します。 Reduxのルールの1つは、正しい情報元が1つだけあるべきであり、データを複製するべきではないということです。 この理由から、 `products`は正しい情報元であり（利用可能な全てのアイテムを含んでいます）、`cart`はカートに追加されたインデックスと数を含んでいます。

<a id="routing">

### ルーティング

私たちは非常にシンプルな（しかし、柔軟性のある）リデュースフレンドリーなルータを使用しています。ルータは、ウィンドウの場所を使用してストアに保管します。これは、 `pwa-helpers`パッケージから提供された`installRouter`ヘルパーメソッドを使用して行います:

```js
import { installRouter } from '@polymer/pwa-helpers/router.js';
firstUpdated() {
  installRouter((location) => this._locationChanged(location));
}
```

<a id="patterns">

## パターン

<a id="connecting-dom-events-to-action-creators">

### アクションクリエイターにDOMイベントを接続する

すべての要素をストアに接続したくない場合は、接続していない要素は、ストア内の状態を更新を伝達する必要があります。

<a id="manually">

#### 手動

これを手動で行うには、イベントを発生させます。 `<child-element>`が接続されてなく、プロパティ`foo`を表示して変更した場合:

- fooが変更されると、`<child-element>`はイベントを発生させます:

```js
_onIncrement() {
  this.value++;
  this.dispatchEvent(new CustomEvent('counter-incremented');
}
```

- `<child-element>`の接続された親はこのイベントをリッスンして、アクションをストアにディスパッチできます:

```html
<counter-element on-counter-incremented="${() => store.dispatch(increment())}"
```

JavaScriptでは

```js
firstUpdated() {
  this.addEventListener('counter-incremented', function() {
    store.dispatch(increment());
  });
}
```

<a id="automatically">

#### 自動

あるいは、`foo-changed`プロパティー変更イベントをReduxアクションに自動的に変換するヘルパーを書くことができます。 これは `<child-element>`のプロパティが通知する（つまり `notify：true`を持つ）必要があることに注意してください。 ここに[サンプル](https://gist.github.com/kevinpschaaf/995c9d1fd0f58fe021b174c4238b38c3#file-5-connect-element-mixin-js)があります。

<a id="reducers-slice-reducers">

### Reducers: スライスする

あなたのアプリをもっとモジュラー化するために、主要な状態オブジェクトをパーツ（"slices"）に分割し、より小さい"スライスレデューサ"を各パーツで動作させることができます（[スライスレデューサについて詳しく読む](https://redux.js.org/docs/recipes/reducers/SplittingReducerLogic.html)). `lazyReducerEnhancer`を使うと、アプリケーションは必要に応じて遅延縮小を追加することができます(例えば、`my-view2`が処理するのに`my-view2js`だけをインポートするには、`counter`スライスレデューサを追加します。)

**`src/store.js:`**
```js
export const store = createStore(
  (state, action) => state,
  compose(lazyReducerEnhancer(combineReducers), applyMiddleware(thunk))
);
```

**`src/components/my-view2.js:`**
```js
// この要素はReduxストアに接続する
import { store } from '../store.js';

// reducerを遅延読み込みする
import counter from '../reducers/counter.js';
store.addReducers({
  counter
});
```

<a id="avoid-duplicate-state">

### 重複状態を避ける

[Reselect](https://github.com/reactjs/reselect)ライブラリを使用して、状態に重複したデータを保存することは避けてください。 例えば、状態は項目のリストを含むことができ、その1つは選択された項目（例えば、URLに基​​づく）である場合です。選択したアイテムを別々に格納する代わりに、選択したアイテムを計算するセレクタを作成します:

```js
import { createSelector } from 'reselect';

const locationSelector = state => state.location;
const itemsSelector = state => state.items;

const selectedItemSelector = createSelector(
  locationSelector,
  itemsSelector,
  (location, items) => items[location.pathname.slice(1)]
);

// 選択した項目を取得するには:
console.log(selectedItemSelector(state));
```

この例を見るには、カートの例を見てください[カート個数セレクタ](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-view3.js#L107) もしくは [Redux-HN](https://github.com/PolymerLabs/polymer-redux-hn)の[アイテムセレクタ](https://github.com/PolymerLabs/polymer-redux-hn/blob/master/src/components/hn-item.ts#L59)。どちらの例でもセレクタはRedux側とビュー層の両方で使用されているため、実際にreducerで定義されています。

<a id="how-to-make-sure-third-party-components-dont-mutate-the-state">

### サードパーティのコンポーネントが状態を変更しないようにする方法

ほとんどのサードパーティコンポーネントは副作用のない方法で使用するようには作られておらず、Reduxストアに接続されていないため、ストアの更新を保証できません。例えば、 例えば`paper-input`は`value`プロパティを持っています。内部的なアクション（つまり、あなたの入力、検証など）に基づいて更新されます。 このような要素がストアを更新しないようにするには:
- 単方向データバインディングを使用して、プリミティブ（文字列、数値など）を要素に渡します。
  - `<paper-input value="${foo}"></paper-input>`
  - それはプリミティブの値なので、paper-inputは`foo`のコピーされた値を受け取ります。`foo`を更新すると、ストア内の実際のプロパティではなく**コピーのみ**が更新されます
  - 要素の外で `foo-changed`イベントを聞き、そこからストアを更新するアクションをディスパッチします（[サンプル](https://github.com/Polymer/pwa-starter-kit/blob/master/src/components/my-view2.js#L51)).
- 配列/オブジェクトは変更可能なので、配列またはオブジェクトの**コピー**を要素に渡します:
  - `<other-input data="${_copy(fooArray)}"></other-input>`
  - `<other-input data="${_deepCopy(fooObj)}"></other-input>`
  - 上のようにイベントを変更して、ストアを更新するアクションをディスパッチします。

<a id="routing-1">

### ルーティング

Reduxを使用すると、基本的にはルーティングは自分で制御します。しかし、私たちは[ルーターヘルパー](https://github.com/Polymer/pwa-helpers/blob/master/src/router.tsl)を提供しています。 私たちの提案は `window.location`に基づいて位置状態を更新することです。 つまり、リンクがクリックされるたびに（またはユーザーが後方をナビゲートすると）、アクションが送出されて場所に基づいて状態が更新されます。これは、タイムトラベルのデバッグではうまくいきます。以前の状態にジャンプしても、URLバーや履歴スタックには影響しません。

ルータの設置と使用の例:

```js
// ...
import { installRouter } from '@polymer/pwa-helpers/router.js';

class MyApp extends connect(store)(LitElement) {
    // ...
    firstUpdated() {
      installRouter((location) => this._locationChanged(location));

      // installRouterに渡される引数はコールバックです。あなたがアクションを
      // ディスパッチすること以外に他の仕事をしていない場合は、次のようなものを書くこともできます:
      // installRouter((location) => store.dispatch(navigate(location.pathname)));
    }

    _locationChanged(location) {
      // どのアクションクリエータをあなたが作成し、
      // どの部分に反映させるか
      store.dispatch(navigate(location.pathname));

      // 必要に応じてdrawerを閉じるなどします。
    }
  }
}
```

<a id="lazy-loading">

### 遅延読み込み

PRPLパターンの主な側面の1つは、必要に応じてアプリケーションのコンポーネントを遅延ロードすることです。これらの遅延ロードされた要素の1つがストアに接続されている場合は、その要素のreducerも遅延ロードできる必要があります。

あなたがこれを行うことができる方法はたくさんあります。 そのうちの1つを[ヘルパー](https://github.com/Polymer/pwa-helpers/blob/master/src/lazy-reducer-enhancer.ts)として実装しました。これはストアに追加することができます:

```js
import lazyReducerEnhancer from '@polymer/pwa-helpers/lazy-reducer-enhancer.js';

// 遅延読み込みされない初期のreducer
import app from './reducers/app.js';

export const store = createStore(
  (state, action) => state,
  compose(lazyReducerEnhancer, applyMiddleware(thunk))
);

// reducerの初期化
store.addReducers({
  app
});
```

この例では、アプリケーションは起動し、 `app` reducersをインストールしますが、他のものはインストールしません。遅延ロードされた要素では、reducerをロードするために、 `store.addReducers`を呼び出すだけです。:

```js
// この要素が遅延読み込みされる場合は、そのreducerも一緒に読み込みする必要があります
import { someReducer } from './store/reducers/one.js';
import { someOtherReducer } from './store/reducers/two.js';

// reducerを遅延読み込み
store.addReducers({someReducer, someOtherReducer});

class MyElement extends ... {
…
}
```

<a id="replicating-the-state-for-storage">

### ストレージの状態を複製する

あなたのアプリがやりたいことの1つは、アプリの状態をストレージに保存することです (データベースもしくは`localStorage`など。Reduxは、基本的には、状態をストレージにダンプする新しいreducerをインストールするだけで済むため、非常に便利です。)

これを行うには、最初に`saveState`と`loadState`という2つの関数を作成して、ストレージに読み書きします:

```js
export const saveState = (state) => {
  let stringifiedState = JSON.stringify(state);
  localStorage.setItem(MY_KEY, stringifiedState);
}
export const loadState = () => {
  let json = localStorage.getItem(MY_KEY) || '{}';
  let state = JSON.parse(json);

  if (state) {
    return state;
  } else {
    return undefined;  // 初期値としてreducerが利用
  }
}
```

そして、 `store.js`では、基本的に` loadState()`の結果をストアのデフォルト状態として使用し、ストアが更新されるたびに`saveState()`を呼び出します:

```js
export const store = createStore(
  (state, action) => state,
  loadState(),  // ローカルストレージデータがある場合は、ロードします。
  compose(lazyReducerEnhancer(combineReducers), applyMiddleware(thunk))
);

// このsubscriberは状態が更新されるたびにローカルストレージに書き込みます。
store.subscribe(() => {
  saveState(store.getState());
});
```

それでおしまい！プロジェクトのデモを見たい場合は、[**Flash-Cards**](https://github.com/notwaldorf/flash-cards/blob/master/src/localStorage.js)アプリが実装しているパターンを見てください。
