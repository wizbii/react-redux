# react-redux

What it takes to replace a framework by React+Redux.

* [Motivation](#motivation)
* [Project Structure](#project-structure)
* [Core](#core)
* [Data Structure](#data-structure)
* [Form Validation](#form-validation)
* [Tests](#tests)
* [Developer Experience](#developer-experience)
* [Things We Are Considering](#things-we-are-considering)
* [Examples](#examples)
* [Updates](#updates)

## Motivation

Looking into an alternative to Angular 2.0, we had to see what it's like to build a large React+Redux application.
This document gather all the resources we found to be useful to efficiently replace such a framework.

Read the full introductory article on Medium: *Coming soon*.

## Project Structure

*We're still experimenting to see what project structure works best but have the feeling that grouping stuff into modules would make sense.*

**Alternatives**

An usual Redux project looks like this:

```
actions/
  index.js
components/
  Explore.js
  List.js
  Repo.js
  User.js
containers/
  App.js
  RepoPage.js
  UserPage.js
reducers/
  index.js
store/
  configureStore.js
index.js
```

There's an other interesting approach called [ducks](https://github.com/erikras/ducks-modular-redux) that consists of grouping the reducer with its action creators.
We're giving it some thought but are concerned about the amount of code that may end up in the same file.

## Core

Those are the building blocks of almost any React+Redux app.
Should it be mentioned that you'll need [React](https://github.com/facebook/react) and [Redux](https://github.com/reactjs/redux)?

### [react-redux](https://github.com/reactjs/react-redux)

Redux alone has nothing to do with React, it's just a pattern that you can follow with any setup.
react-redux is just there to reduce boilerplate code and abstract the react specific bindings.

**Alternatives**

We'd suggest to go through the Dan Abramov's egghead serie "[Getting Started with Redux](https://egghead.io/series/getting-started-with-redux)" to see what react-redux looks like behind the scene.
As you'll see, it's actually just abstracting a few repetitive tasks like subscribing to a store.
Unless you like to WET, there's no reasons not to use this package.

### [isomorphic-fetch](https://github.com/matthew-andrews/isomorphic-fetch)

Until recently, the only solution to deal with http requests was to use the awkward [XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest).
The good news is that browsers have recently implemented a new api: [fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API).
As with every new browser feature, it's not yet supported everywhere so you'll need a polyfill such as isomorphic-fetch.
Internally it uses [github/fetch](https://github.com/github/fetch) but follows an "universal" approach, making it work as well on the server (NodeJS) as on the client.

There are a few things that may differ from your usual library so make sure to have a look at [GitHub's notes](https://github.com/github/fetch#caveats).

**Update:** it's important to keep in mind that fetch is a browser feature and spec'ed as such.
It's much simpler than a library like [request](https://github.com/request/request) and may end up not fitting your needs on the server.
Also, if you plan to reuse your ES6/7 wrapper then you'll need to set up the server accordingly.
In the end, using fetch is still a good idea but the isomorphic version may not be that useful.

**Alternatives**

* [XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) - For the old school dudes.
* [github/fetch](https://github.com/github/fetch) - If you don't care too much about having an "universal" fetch.

### [redux-thunk](https://github.com/gaearon/redux-thunk)

Redux action creators are pure, synchronous functions.
So to deal with asynchronous actions you'll need to setup a middleware (something that all actions go through).
redux-thunk is one that allows you to dispatch a function returned from an action creator.
This function is then able to dispatch as many actions as needed.

**Alternatives**

redux-thunk is a simple middleware that allows you to do almost anything but here are a few more specific ones:

* [redux-promise](https://github.com/acdlite/redux-promise) - Add support for promises.

### [lodash](https://github.com/lodash/lodash)

lodash has everything you can dream of when it comes to manipulating data.
The npm package contains every functions splitted as its own module, making it easy to require just what you need.

**Alternatives**

The JavaScript language already has built-in support for some of lodash's features (e.g `.map()`, `.reduce()`) so you might not need it for small apps.

* [underscore](https://github.com/jashkenas/underscore) - The older brother (if not father) of lodash.
* [ramda](https://github.com/ramda/ramda) - Goes a step further into functional programming.

### [react-router](https://github.com/reactjs/react-router)

If you're building a single page application (SPA), meaning all the routing is handled on the client, then you'll need react-router.

**Alternatives**

We couldn't find any active alternative so react-router is the way to go.

### [react-router-redux](https://github.com/reactjs/react-router-redux)

Synchronize the router with the application state and provides some "Redux-way" of manipulating routes.
It's important to understand that react-router-redux doesn't add any new features to the react-router, it's just a way to follow the Redux pattern properly.
By doing so, it also improve the developer experience by enabling time travel.

**Alternatives**

While we'd strongly advise to use react-router-redux, you could probably skip it if you're just getting started with React and the React Router.

* [redux-router](https://github.com/acdlite/redux-router) - Differs more from the react-router api but brings some niceties. Not official though and announced to be "experimental".

### [classnames](https://github.com/JedWatson/classnames)

Conditionally applying classes to an element is a little cluttered by default.
This package allows you to handle it gracefully, in a way that's going to look familiar to Angular developers.

**Alternatives**

Just stick to string concatenation if you don't find yourself dynamically adding classes so often.

## Data Structure

The Redux pattern involves having all the app's state in a single store so taking the time to design its shape properly is really important.
There are a few tools out there that helps enforce those decisions while abstracting the internals.

### [normalizr](https://github.com/gaearon/normalizr)

Built by Dan Abramov, the creator of Redux, normalizr flatten data structures.
It helps avoiding to duplicate data in different places of the application.
Instead of having different versions of the same user object in a publication, a comment and an article, it's (more or less) at the root of your store.

**Alternatives**

Duplicated data is a common issue to any kind of application.
For example, Angular tries to solve it by encouraging its users to store the data within a service.
While normalizr is not a requirement, it really helps solving it painlessly.

### [immutable-js](https://github.com/facebook/immutable-js/)

Not mutating objects in JavaScript is perfectly doable but takes some efforts (e.g `Object.assign({}, state, { counter: state.counter + 1 })`).
On the other hand, Immutable offers a more secure and friendlier api to achieve such tasks.

**Alternatives**

Not using an immutable library is perfectly fine but you'll need to pay special attention to how you're manipulating data.

* [mori](https://github.com/swannodette/mori) - Apparently it's more functional and faster but larger. It's also less known and has less support so it may be harder to find examples/answers.

### [flux-standard-action](https://github.com/acdlite/flux-standard-action)

This standard defines how an action should look, making them more predictable.

**Alternatives**

You don't need to follow this particular standard but we'd strongly advise to establish one with your team.

## Form Validation

### [kay](https://github.com/Zhouzi/kay)

The purpose of client side validation is to help and guide users through forms but not necessarily to strictly validate their inputs.
Surprisingly, the several form validation library we found were too large and complex for our needs so we decided to build one.

**Alternatives**

* [joi](https://github.com/hapijs/joi) - Really huge. Might be great on the server but clearly an overkill on the client.
* [yup](https://github.com/jquense/yup) - Inspired by joi but smaller.
* [validator.js](https://github.com/chriso/validator.js) - Looks cool but still too complex, does too much things.

## Tests

That's a subject we still have to explore but we've heard great things about [jest](https://github.com/facebook/jest), [enzyme](https://medium.com/airbnb-engineering/enzyme-javascript-testing-utilities-for-react-a417e5e5090f#.qkwnuwazn) and the well known [mocha](https://github.com/mochajs/mocha).
Note: the React team is thinking about making [enzyme the official TestUtils](https://github.com/reactjs/core-notes/blob/master/2016-05/may-05.md#testutils-and-enzyme).

## Developer Experience

React and Redux do an incredible job at telling you about what's going on, which is especially great when it's going wrong.
But beyond the libraries, there are quite a few nice developer tools available out there.

### [Redux DevTools](https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd)

This Chrome Extension logs every actions dispatched through the application, along with its state.
It then allow you to replay and reset them (among other things).
It doesn't involve much setup and have no impact in "production" mode.
Have a look at Dan Abramov's talk "[Live React: Hot Reloading with Time Travel at react-europe 2015](https://medium.com/r/?url=https%3A%2F%2Fwww.youtube.com%2Fwatch%3Fv%3DxsSnOQynTHs)" which introduced such concepts with Redux.

## Things We Are Considering

What's so great about Redux is that it's just pure vanilla JavaScript so it's really easy extend it.
Below is a list of things that we may need at some point.

### [topologically-combine-reducers](https://github.com/KodersLab/topologically-combine-reducers)

Sometimes a reducer depends on some other part of the state that is not accessible.
This library allows you to define and access those dependencies.

### [ReactTransitionGroup](https://facebook.github.io/react/docs/animation.html)

Official React component that offers a simple api for css based animations.

### [react-motion](https://github.com/chenglou/react-motion)

Advanced JavaScript animation components, seems to achieve great things with little code.

## Examples

We've spent some time studying awesome examples to make our mind so here's the list:

* [react-redux-universal-hot-example](https://github.com/erikras/react-redux-universal-hot-example)
* [react-redux-starter-kit](https://github.com/davezuko/react-redux-starter-kit)
* [redux real-world example](https://github.com/reactjs/redux/tree/master/examples/real-world)
* [react-router huge-apps example](https://github.com/reactjs/react-router/tree/master/examples/huge-apps)

## Updates

### 2016-04-02

* Updated tests: the React team is discussing about making [enzyme the official TestUtils](https://github.com/reactjs/core-notes/blob/master/2016-05/may-05.md#testutils-and-enzyme).
* Removed redux-actions: we faced several issues due to a lack of flexibility and inconsistencies.
* Added flux-standard-action: we dropped redux-actions but still follow this standard.
* Added enzyme: the redux team seem to be interested in refactoring some of their tests to use it (see [reactjs/redux#1481](https://github.com/reactjs/redux/issues/1481)).

### 2016-05-28

* Updated isomorphic-fetch: fetch may be too simple for certain server-side cases.
