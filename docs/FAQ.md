# Redux FAQ

## General

### Can Redux only be used with React?  

Redux can be used as a data store for any UI layer.  The most common usage is with React, but there are bindings available for Angular, Vue, Mithril, and more.  Redux simply provides a subscription mechanism, which can be used by any other code.


### Do I need to have a particular build tool to use Redux?

Redux is written in ES6 and built for production with Webpack and Babel.  However, it should be usable in any Javascript build process.  There is also a UMD build that allows usage without any build process at all.  The [counter-vanilla](https://github.com/reactjs/redux/tree/master/examples/counter-vanilla) example demonstrates basic ES5 usage with Redux included in a script tag.  As the pull request that added this said:

> The new Counter Vanilla example is aimed to dispel the myth that Redux requires Webpack, React, hot reloading, sagas, action creators, constants, Babel, npm, CSS modules, decorators, fluent Latin, an Egghead subscription, a PhD, or an Exceeds Expectations O.W.L. level.

>Nope, it's just HTML, some artisanal `<script>` tags, and plain old DOM manipulation. Enjoy!


## Reducers

### How do I share state between two reducers? Do I have to use `combineReducers`?

The suggested structure for a Redux store is to split the state object into multiple "slices" or "domains" by key, and provide a separate reducer function to manage each individual data slice.  This is similar to how the standard Flux pattern has multiple independent stores, and Redux provides the [`combineReducers`](api/combineReducers.md) utility function to make this pattern easier.  However, it's important to note that `combineReducers` is NOT required - it is simply a utility function for the common use case of having a single reducer function per state slice, with plain Javascript objects for the data.

Many users later want to try to share data between two reducers, but find that `combineReducers` does not allow them to do so.  There are several approaches that can be used:

* If a reducer needs to know data from another slice of state, the store may need to be reorganized so that a single reducer is handling more of the data.
* You may need to write some custom functions for handling some of these actions.  This may require replacing `combineReducers` with your own top-level reducer function.  You can also use a utility such as [reduce-reducers](https://github.com/acdlite/reduce-reducers) to run `combineReducers` to handle most actions, but also run a more specialized reducer for specific actions that cross state slices.
* [Async action creators](advanced/AsyncActions.md) such as `redux-thunk` have access to the entire state through `getState()`.  An action creator can retrieve additional data from the state and put it in an action, so that each reducer has enough information to update its own state slice.

In general, remember that reducers are just functions - you can organize them and subdivide them any way you want, and you are encouraged to break them down into smaller, reusable functions ("reducer decomposition").  You just need to make sure that together they follow the basic rules of reducers: `(state, action) -> newState`, and update state immutably rather than mutating it directly.

#### Further information:
**Documentation**:
- [`combineReducers`](api/combineReducers.md)
- [Structuring Reducers](recipes/StructuringReducers.md)

**Discussions**:
- [#601 - A concern on combineReducers, when an action is related to multiple reducers](https://github.com/reactjs/redux/issues/601)
- [#1400 - Is passing top-level state object to branch reducer an anti-pattern?](https://github.com/reactjs/redux/issues/1400)
- [SO - Accessing other parts of the state when using combined reducers?](http://stackoverflow.com/questions/34333979/accessing-other-parts-of-the-state-when-using-combined-reducers)
- [Sharing State Between Redux Reducers](https://invalidpatent.wordpress.com/2016/02/18/sharing-state-between-redux-reducers/)


### Do I have to use a switch statement to handle actions?

No.  You are welcome to use any approach you'd like to respond to an action in a reducer.  A switch statement is the most common approach, but it's fine to use if statements, a lookup table of functions, or create a function that abstracts the process.

#### Further information:
**Documentation**:
- [Reducing Boilerplate](recipes/ReducingBoilerplate.md)

**Discussions**:
- [#883 - take away the huge switch block](https://github.com/reactjs/redux/issues/883)
- [#1167 - Reducer without switch](https://github.com/reactjs/redux/issues/1167)


## Organizing State

### Do I have to put all my state into Redux? Should I ever use React's setState?

There is no "right" answer for this.  Some users prefer to keep every single piece of data in Redux, to maintain a fully serializable and controlled version of their application at all times.  Others prefer to keep non-critical or UI state, such as "is this dropdown currently open", inside a component's internal state.  Find a balance that works for you, and go with it.

There are a number of community packages that implement various approaches for storing per-component state in a Redux store instead, such as [redux-ui](https://github.com/tonyhb/redux-ui), [redux-component](https://github.com/tomchentw/redux-component), [redux-react-local](https://github.com/threepointone/redux-react-local), and more.

#### Further information
**Discussions**:
- [#159 - Investigate using Redux for pseudo-local component state](https://github.com/reactjs/redux/issues/159)
- [#1098 - Using Redux in reusable React component](https://github.com/reactjs/redux/issues/1098)
- [#1287 - How to choose between Redux's store and React's state?](https://github.com/reactjs/redux/issues/1287)
- [#1385 - What are the disadvantages of storing all your state in a single immutable atom?](https://github.com/reactjs/redux/issues/1385)


### Can I put functions, promises, or other non-serializable items in my store state?

It is highly recommended that you only put plain serializable objects, arrays, and primitives into your store.  It's _technically_ possible to insert non-serializable items into the store, but doing so can break the ability to persist and rehydrate the contents of a store.

#### Further information
**Discussions**:
- [#1248 - Is it ok and possible to store a react component in a reducer?](https://github.com/reactjs/redux/issues/1248)
- [#1279 - Have any suggestions for where to put a Map Component in Flux?](https://github.com/reactjs/redux/issues/1279)
- [#1390 - Component Loading](https://github.com/reactjs/redux/issues/1390)
- [#1407 - Just sharing a great base class](https://github.com/reactjs/redux/issues/1407)


### How do I organize nested/duplicate data in my state?

Data with IDs, nesting, or relationships should generally be stored in a "normalized" fashion - each object should be stored once, keyed by ID, and other objects that reference it should only store the ID rather than a copy of the entire object.  It may help to think of parts of your store as a database, with individual "tables" per item type.  Libraries such as [normalizr](https://github.com/gaearon/normalizr) and [redux-orm](https://github.com/tommikaikkonen/redux-orm) can provide help and abstractions in managing normalized data.


#### Further information
**Documentation**:
- [Async Actions](advanced/AsyncActions.md)
- [Real World example](introduction/Examples.html#real-world)


**Discussions**:
- [#316 - How to create nested reducers?](https://github.com/reactjs/redux/issues/316)
- [#815 - Working with Data Structures](https://github.com/reactjs/redux/issues/815)
- [#946 - Best way to update related state fields with split reducers?](https://github.com/reactjs/redux/issues/946)
- [#994 - How to cut the boilerplate when updating nested entities?](https://github.com/reactjs/redux/issues/994)



## Store Setup

### Can I / should I create multiple stores?  Can I import my store directly, and use it in components myself?

The original Flux pattern describes having multiple "stores" in an app, each one holding a different area of domain data.  This can introduce issues such as needing to have one store "waitFor" another store to update.  Redux is designed use a variation on this concept, where each individual Flux store would become a separate sub-reducer in the the single Redux store.

As with several other questions, it is _possible_ to create multiple distinct Redux stores in a page, but the intended pattern is to have only a single store.  However, having a single store enables using the Redux DevTools, makes persisting and rehydrating data simpler, and simplifies subscription logic.

Similarly, while you _can_ reference your store instance by importing it directly, this is not a recommended pattern in Redux.  For React usage, the wrapper classes generated by the React-Redux `connect()` function do actually look for `props.store` if it exists, but it's best if you simply use a `<Provider store={store} />` at the top of your component chain and let React-Redux worry about the store.  Importing a store directly also makes it harder to leverage server-side rendering.



#### Further information
**Documentation**:
- [Store](api/Store.md)


**Discussions**:
- [#1346 - Is it bad practice to just have a 'stores' directory?](https://github.com/reactjs/redux/issues/1436)
- [SO - Redux multiple stores, why not?](http://stackoverflow.com/questions/33619775/redux-multiple-stores-why-not)


### Is it OK to have more than one middleware chain in my store enhancer?  What is the difference between "next" and "dispatch" in a middleware function?

Redux middleware act like a linked list.  Each middleware function can either call `next(action)` to pass an action along to the next middleware in line, call `dispatch(action)` to restart the processing at the beginning of the list, or do nothing at all to stop the action from being processed further.  

This chain of middleware is defined by the arguments passed to the `applyMiddleware` function used when creating a store.  Defining multiple chains will not work correctly, as they would have distinctly different `dispatch` references and the different chains would effectively be disconnected.

#### Further information
**Documentation**
- [Middleware](advanced/Middleware.md)
- [applyMiddleware](api/applyMiddleware.md)

**Discussions**
- [#1051 - Shortcomings of the current applyMiddleware and composing createStore](https://github.com/reactjs/redux/issues/1051)
- [Understanding Redux Middleware](https://medium.com/@meagle/understanding-87566abcfb7a)
- [Exploring Redux Middleware](http://blog.krawaller.se/posts/exploring-redux-middleware/)


### How do I subscribe to only a portion of the state?


## Actions

### Why should "type" be a string, or at least serializable? Why should my action types be constants?

As with state, having actions be serializable enables several of Redux's defining features, such as time travel debugging, and recording and replaying actions.  Using something like a Symbol for the "type" value would break that.  Strings are serializable and easily self-descriptive, and so are a better choice.  Note that it IS okay to use Symbols, Promises, or other non-serializable values in an action if the action is intended for use by middleware - actions just need to be serializable by the time they actually reach the store and are passed to reducers.

Encapsulating and centralizing commonly used pieces of code is a key concept in programming.  While it is certainly possible to manually create action objects everywhere, and write each "type" value by hand, defining reusable constants makes maintaining code easier.


#### Further information:
**Documentation**:
- [Reducing Boilerplate](http://rackt.github.io/redux/docs/recipes/ReducingBoilerplate.html#actions)

**Discussion**:
- [#384 - Recommend that Action constants be named in the past tense](https://github.com/reactjs/redux/issues/384)
- [#628 - Solution for simple action creation with less boilerplate](https://github.com/reactjs/redux/issues/628)
- [#1024 - Proposal: Declarative reducers](https://github.com/reactjs/redux/issues/1024)
- [#1167 - Reducer without switch](https://github.com/reactjs/redux/issues/1167)
- [SO - Why do you need 'Actions' as data in Redux?](http://stackoverflow.com/q/34759047/62937)
- [SO - What is the point of the constants in Redux?](http://stackoverflow.com/q/34965856/62937)


### Should I have a 1-1 mapping between reducers and actions?

- http://stackoverflow.com/questions/35406707/do-events-and-actions-have-a-11-relationship-in-redux

### How can I represent "side effects" such as AJAX calls?

### Should I dispatch multiple actions in a row from one action creator?


## Code Structure

### How should I group my action creators and reducers in my project?

### Where should my selectors go?

### How should I split my logic between reducers and action creators?

- https://github.com/reactjs/redux/issues/1171 


## Performance


### Won't calling "all my reducers" for each action be slow?

### Do I have to deep-copy my state in a reducer? Isn't copying my state going to be slow?

### How can I reduce the number of store update events?

### Will having "one state tree" cause memory problems?


## React-Redux

### Why isn't my component re-rendering, or my mapStateToProps running?


- https://github.com/reactjs/react-redux/issues/291

### Why is my component re-rendering too often?

### How can I speed up my mapStateToProps?


## Community

### Are there any larger, "real" Redux projects?
