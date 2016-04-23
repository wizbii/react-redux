# react-redux

What it takes to replace a framework by React+Redux.

* [Motivation](#motivation)
* [Project Structure](#project-structure)
* [Core](#core)
* [Data Structure](#data-structure)
* [Form Validation](#form-validation)
* [Tests](#tests)
* [Things We Are Considering](#things-we-are-considering)
* [Examples](#examples)

## Motivation

Looking into an alternative to Angular 2.0, we had to see what it's like to build a large React+Redux application.
This document gather all the resources we found to be useful to efficiently replace such a framework.

Read the full article on Medium: *Coming soon*.

## Project Structure

*We're still experimenting to see what project structure works best but have the feeling that grouping stuff into modules could be great.*

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

There's an other interesting approach called [ducks](ducks-modular-redux) that consists of grouping the reducer with its action creators.
While that sounds interesting, I'm afraid you end up with a lot of code in the same file as the app grows.
Especially when you're having several request/success/failure actions.

## Core

Those are the common things that you'll need in almost every application.
Should I mention that you'll need [React](https://github.com/facebook/react) and [Redux](https://github.com/reactjs/redux)?

### [react-redux](https://github.com/reactjs/react-redux)

Redux alone has nothing to do with React, it's just a pattern that you can follow with any setup.
That been said, react-redux is a package that abstract some react specific bindings.

**Alternatives**

I'd suggest to go through the Dan Abramov's egghead serie "[Getting Started with Redux](https://egghead.io/series/getting-started-with-redux)" to see what react-redux looks like behind the scene.
As you'll see, it's actually just abstracting a few repetitive tasks like subscribing to a store changes within a component leading to less boiler plate.
You could totally code it yourself but why would you do that?

### [isomorphic-fetch](https://github.com/matthew-andrews/isomorphic-fetch)

At some point you'll need to deal with http request.
Until recently, the only solution was to use the awkward [XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) but the good news is that browsers have implemented a new api: [fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API).
The problem is that it's not yet available in every browser so you'll need a polyfill and that's what isomorphic-fetch is.
Internally it uses [github/fetch](https://github.com/github/fetch) but makes it work as well on the server (NodeJS) as on the client.

Note: there are a few things that may differ from your usual library so make sure to have a look at [GitHub's notes](https://github.com/github/fetch#caveats).

**Alternatives**

* [XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) - For the old school dudes.
* [github/fetch](https://github.com/github/fetch) - If you don't care too much of having an "universal" fetch.

### [redux-thunk](https://github.com/gaearon/redux-thunk)

Quick enough you'll come through the problem of asynchronous actions.
The thing is, an action is synchronous and has no side effects, it's just an object.
To deal with anything that's not really synchronous or have side effects, you can create an action that returns a function instead of an object.
This function is then able to dispatch as many action as it wants.
But for Redux to accept a function as an action you'll have to add a middleware.

**Alternatives**

Below is a list of some more specific middleware but almost everything is doable in a function so I guess in most cases redux-thunk alone should be enough.

* [redux-promise](https://github.com/acdlite/redux-promise) - A middleware that allows you to return a promise from an action creator.

### [lodash](https://github.com/lodash/lodash)

lodash has everything you can dream of when it comes to manipulate data.
With the npm package's library comes the features as separate modules, making it easy to require just what you need.

**Alternatives**

I guess you could be ok without lodash as JavaScript already has built-in support for some of its features (e.g `.map()`, `.reduce()`) but that's until you give it a try.

* [underscore](https://github.com/jashkenas/underscore) - The older brother (if not father) of lodash.
* [ramda](https://github.com/ramda/ramda) - This library goes a step further into functional programming.

### [react-router](https://github.com/reactjs/react-router)

If you're building a single page application (SPA), meaning all the routing is handled on the client, then you'll need react-router.

**Alternatives**

The only one I had heard of is now deprecated so the only solution seems to be react-router (unless you don't need a router, of course).

## Data Structure

The Redux pattern involves having all the app's data in a single store so taking the time to design its shape properly is important.
There are a few tools out there that helps enforce those decisions while abstracting the internals.

### [normalizr](https://github.com/gaearon/normalizr)

Built by Dan Abramov (creator of Redux), normalizr flatten data structures.
It helps you avoid duplicating data in different places.
So instead of having different versions of the same user object in a publication, a comment and an article, it's all in the root of your store.
It means that everything points to the same, updated, user object.

**Alternatives**

normalizr is not a requirement to build a Redux app and you could just go with your usual structure but it really helps.
It's a well known problem that Angular tries to solve with its services for example.

### [immutable-js](https://github.com/facebook/immutable-js/)

Reducers are pure function that have no side effects and should not mutate the state object.
While it's perfectly doable to not mutate an object in JavaScript, it takes some efforts (e.g `Object.assign({}, state, { counter: state.counter + 1 })`).
On the other hand, Immutable offers a more secure and friendlier api to achieve such tasks (oh and it's a facebook library :)).

**Alternatives**

Not using an immutable library is perfectly fine but you'll need to pay some special attention when manipulating your data.

* [mori](https://github.com/swannodette/mori) - Apparently it's more functional, faster but also larger. It's also less known, has less support which means it may be harder to find examples/answers.

### [redux-actions](https://github.com/acdlite/redux-actions)

In a team, developers should follow the same conventions so there should be only one way of structuring an action.
Then you have two choices: define your own standards or follow one that have already convinced a lot of developers: [flux-standard-action](https://github.com/acdlite/flux-standard-action).
If you decide to go with the flux-standard-action's approach then redux-actions might come handy.
It's a little wrapper that helps abstract and follow the rules without the trouble.

**Alternatives**

The point here is not much about using redux-actions but having a standard.
If you decide not to use it, just make sure to use one that fit best your needs.

## Form Validation

### [kay](https://github.com/Zhouzi/kay)

When looking for a form validation library we've come across a few interesting ones but that were doing too much things.
Client side validation is often there just to help the user but not necessarily to strictly validate the data.
Angular is probably the best at handling form validation and when you think about it, it's pretty simple (I'm talking about form validation, not the form state which is way more complex).
It mostly relies on the HTML5 attributes and constraints and that works great in 99% of the cases.
That's why I ended up building a micro, low level library to achieve such tasks.

**Alternatives**

* [joi](https://github.com/hapijs/joi) - Really huge. Might be great on the server but clearly an overkill on the client.
* [yup](https://github.com/jquense/yup) - Inspired by joi but smaller, still too complex in my opinion.
* [validator.js](https://github.com/chriso/validator.js) - Smaller and looks cool but may still do too much things.

## Tests

That's a subject we still have to explore but we've heard great things about [jest](https://github.com/facebook/jest) and the well known [mocha](https://github.com/mochajs/mocha).

## Things We Are Considering

What's so great about Redux is that it's just pure vanilla JavaScript in the end so it's really easy extend it.
Below is a list of things looks and may become really useful at some point in an application.

### [topologically-combine-reducers](https://github.com/KodersLab/topologically-combine-reducers)

Sometimes a reducer depends on some other part of the state that is not accessible.
This library allows you to define and access those dependencies.

## Examples

Looking at how others are dealing with large applications is really interesting so here are few we're inspired by.

* [react-redux-universal-hot-example](https://github.com/erikras/react-redux-universal-hot-example)
* [react-redux-starter-kit](https://github.com/davezuko/react-redux-starter-kit)
* [redux real-world example](https://github.com/reactjs/redux/tree/master/examples/real-world)
* [react-router huge-apps example](https://github.com/reactjs/react-router/tree/master/examples/huge-apps)