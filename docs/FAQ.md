# Redux FAQ

## General

### When should I use Redux?

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
- [API: combineReducers](api/combineReducers.md)
- [Recipes: Structuring Reducers](recipes/StructuringReducers.md)

**Discussions**:
- [#601 - A concern on combineReducers, when an action is related to multiple reducers](https://github.com/reactjs/redux/issues/601)
- [#1400 - Is passing top-level state object to branch reducer an anti-pattern?](https://github.com/reactjs/redux/issues/1400)
- [SO - Accessing other parts of the state when using combined reducers?](http://stackoverflow.com/questions/34333979/accessing-other-parts-of-the-state-when-using-combined-reducers)
- [SO - Reducing an entire subtree with redux combineReducers](http://stackoverflow.com/questions/34427851/reducing-an-entire-subtree-with-redux-combinereducers)
- [Sharing State Between Redux Reducers](https://invalidpatent.wordpress.com/2016/02/18/sharing-state-between-redux-reducers/)


### Do I have to use a switch statement to handle actions?

No.  You are welcome to use any approach you'd like to respond to an action in a reducer.  A switch statement is the most common approach, but it's fine to use if statements, a lookup table of functions, or create a function that abstracts the process.

#### Further information:
**Documentation**:
- [Recipes: Reducing Boilerplate](recipes/ReducingBoilerplate.md)

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
- [SO - Why is state all in one place, even state that isn't global?](http://stackoverflow.com/questions/35664594/redux-why-is-state-all-in-one-place-even-state-that-isnt-global)
- [SO - Should all component state be kept in Redux store?](http://stackoverflow.com/questions/35328056/react-redux-should-all-component-states-be-kept-in-redux-store)


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
- [Advanced: Async Actions](advanced/AsyncActions.md)
- [Examples: Real World example](introduction/Examples.html#real-world)


**Discussions**:
- [#316 - How to create nested reducers?](https://github.com/reactjs/redux/issues/316)
- [#815 - Working with Data Structures](https://github.com/reactjs/redux/issues/815)
- [#946 - Best way to update related state fields with split reducers?](https://github.com/reactjs/redux/issues/946)
- [#994 - How to cut the boilerplate when updating nested entities?](https://github.com/reactjs/redux/issues/994)
- [#1255 - Normalizr usage with nested objects in React/Redux](https://github.com/reactjs/redux/issues/1255)



## Store Setup

### Can I / should I create multiple stores?  Can I import my store directly, and use it in components myself?

The original Flux pattern describes having multiple "stores" in an app, each one holding a different area of domain data.  This can introduce issues such as needing to have one store "waitFor" another store to update.  Redux is designed use a variation on this concept, where each individual Flux store would become a separate sub-reducer in the the single Redux store.

As with several other questions, it is _possible_ to create multiple distinct Redux stores in a page, but the intended pattern is to have only a single store.  However, having a single store enables using the Redux DevTools, makes persisting and rehydrating data simpler, and simplifies subscription logic.

Similarly, while you _can_ reference your store instance by importing it directly, this is not a recommended pattern in Redux.  For React usage, the wrapper classes generated by the React-Redux `connect()` function do actually look for `props.store` if it exists, but it's best if you simply use a `<Provider store={store} />` at the top of your component chain and let React-Redux worry about the store.  Importing a store directly also makes it harder to leverage server-side rendering.



#### Further information
**Documentation**:
- [API: Store](api/Store.md)


**Discussions**:
- [#1346 - Is it bad practice to just have a 'stores' directory?](https://github.com/reactjs/redux/issues/1436)
- [SO - Redux multiple stores, why not?](http://stackoverflow.com/questions/33619775/redux-multiple-stores-why-not)
- [SO - Accessing Redux state in an action creator](http://stackoverflow.com/questions/35667249/accessing-redux-state-in-an-action-creator)


### Is it OK to have more than one middleware chain in my store enhancer?  What is the difference between "next" and "dispatch" in a middleware function?

Redux middleware act like a linked list.  Each middleware function can either call `next(action)` to pass an action along to the next middleware in line, call `dispatch(action)` to restart the processing at the beginning of the list, or do nothing at all to stop the action from being processed further.  

This chain of middleware is defined by the arguments passed to the `applyMiddleware` function used when creating a store.  Defining multiple chains will not work correctly, as they would have distinctly different `dispatch` references and the different chains would effectively be disconnected.

#### Further information
**Documentation**
- [Advanced: Middleware](advanced/Middleware.md)
- [API: applyMiddleware](api/applyMiddleware.md)

**Discussions**
- [#1051 - Shortcomings of the current applyMiddleware and composing createStore](https://github.com/reactjs/redux/issues/1051)
- [Understanding Redux Middleware](https://medium.com/@meagle/understanding-87566abcfb7a)
- [Exploring Redux Middleware](http://blog.krawaller.se/posts/exploring-redux-middleware/)


### How do I subscribe to only a portion of the state?  Can I get the dispatched action as part of the subscription?

Redux provides a single `store.subscribe` method for notifying listeners that the store has updated.  Listener callbacks do not receive the current state as an argument - it is simply an indication that _something_ has changed.  The subscriber logic can then call `getState()` to get the current state value.

This API is intended as a low-level primitive with no dependencies or complications, and can be used to build higher-level subscription logic.  UI bindings such as React-Redux can create a subscription for each connected component.  It is also possible to write functions that can intelligently compare the old state vs the new state, and execute additional logic if certain pieces have changed.  Examples include [redux-watch](https://github.com/jprichardson/redux-watch) and [redux-subscribe](https://github.com/ashaffer/redux-subscribe), which offer different approaches to specifying subscriptions and handling changes.

The new state is not passed to the listeners in order to remove special-cases and simplify implementing addons such as the Redux DevTools.  In addition, subscribers are intended to react to the state value itself, not the action.  Middleware can be used if dealing with the action is needed.

#### Further information
**Documentation**
- [Basics: Store](basics/Store.md)
- [API: Store](api/Store.md)

**Discussions**
- [#303 - subscribe API with state as an argument](https://github.com/reactjs/redux/issues/303)
- [#580 - Is it possible to get action and state in store.subscribe?](https://github.com/reactjs/redux/issues/580)
- [#922 - Proposal: add subscribe to middleware API](https://github.com/reactjs/redux/issues/922)
- [#1057 - subscribe listener can get action param?](https://github.com/reactjs/redux/issues/1057)
- [#1300 - Redux is great but major feature is missing](https://github.com/reactjs/redux/issues/1300)


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


### Is there always a 1-1 mapping between reducers and actions?


- http://stackoverflow.com/questions/35406707/do-events-and-actions-have-a-11-relationship-in-redux
- http://stackoverflow.com/questions/35493352/can-i-dispatch-multiple-actions-without-redux-thunk-middleware/35642783
- https://github.com/reduxible/reduxible/issues/8

### How can I represent "side effects" such as AJAX calls?

- http://stackoverflow.com/questions/35411423/how-to-dispatch-a-redux-action-with-a-timeout/35415559
- http://stackoverflow.com/questions/32982237/where-should-i-put-synchronous-side-effects-linked-to-actions-in-redux/33036344
- http://stackoverflow.com/questions/32925837/how-to-handle-complex-side-effects-in-redux/33036594
- http://stackoverflow.com/questions/33011729/how-to-unit-test-async-redux-actions-to-mock-ajax-response/33053465
- http://stackoverflow.com/questions/35262692/how-to-fire-ajax-calls-in-response-to-the-state-changes-with-redux/35675447
- https://www.reddit.com/r/reactjs/comments/469iyc/help_performing_async_api_calls_with_reduxpromise/
- http://stackoverflow.com/questions/34570758/why-do-we-need-middleware-for-async-flow-in-redux

### Should I dispatch multiple actions in a row from one action creator?
- http://stackoverflow.com/questions/33637740/should-i-use-one-or-several-action-types-to-represent-this-async-action/33816695
- http://stackoverflow.com/questions/35406707/do-events-and-actions-have-a-11-relationship-in-redux/35410524


## Code Structure

### What should my file structure look like? How should I group my action creators and reducers in my project? Where should my selectors go?

- http://jaysoo.ca/2016/02/28/organizing-redux-application/
- http://stackoverflow.com/questions/32634320/how-to-structure-redux-components-containers/32921576

### How should I split my logic between reducers and action creators?

- https://github.com/reactjs/redux/issues/1171 


## Performance

### How well does Redux "scale" in terms of performance and architecture?

While there's no single definitive answer to this, most of the time this should not be a concern in either case. 

The work done by Redux generally falls into a few areas: processing actions in middleware and reducers (including object duplication for immutable updates), notifying subscribers after actions are dispatched, and updating UI components based on state changes.  While it's certainly _possible_ for each of these to become a performance concern in sufficiently complex situations, there's nothing inherently slow or inefficient about how Redux is implemented.  In fact, React-Redux in particular is heavily optimized to cut down on unnecessary re-renders.  

As for architecture, anecdotal evidence is that Redux works well for varying project and team sizes.  Redux is currently used by hundreds of companies and thousands of developers, with several hundred thousand monthly installations from NPM.  One developer reported:

> for scale, we have ~500 action types, ~400 reducer cases, ~150 components, 5 middlewares, ~200 actions, ~2300 tests

#### Further information
**Discussions**:
- [#310 - Who uses Redux?](https://github.com/reactjs/redux/issues/310)
- [Reddit - What's the best place to keep the initial state?](https://www.reddit.com/r/reactjs/comments/47m9h5/whats_the_best_place_to_keep_the_initial_state/)
- [Reddit - Help designing Redux state for a single page app](https://www.reddit.com/r/reactjs/comments/48k852/help_designing_redux_state_for_a_single_page/)
- [Reddit - Redux performance issues with a large state object?](https://www.reddit.com/r/reactjs/comments/41wdqn/redux_performance_issues_with_a_large_state_object/)
- [Twitter - Redux scaling](https://twitter.com/NickPresta/status/684058236828266496)


### Won't calling "all my reducers" for each action be slow?



### Do I have to deep-copy my state in a reducer? Isn't copying my state going to be slow?

### How can I reduce the number of store update events?

Redux notifies subscribers after each successfully dispatched action (ie, the action reached the store and was handled by reducers).  In some cases, it may be useful to cut down on the number of times subscribers are called, particularly if an action creator dispatches multiple distinct actions in a row.  There are a number of community addons that provide batching of subscription notifications after multiple actions are dispatched, such as [redux-batched-updates](https://github.com/acdlite/redux-batched-updates),  [redux-batched-subscribe](https://github.com/tappleby/redux-batched-subscribe), or [redux-batched-actions](https://github.com/tshelburne/redux-batched-actions).

#### Further information
**Discussions**:
- [#125 - Strategy for avoiding cascading renders](https://github.com/reactjs/redux/issues/125)
- [#542 - Idea: batching actions](https://github.com/reactjs/redux/issues/542)
- [#911 - Batching actions](https://github.com/reactjs/redux/issues/911)
- [React-Redux #263 - Huge performance issue when dispatching hundreds of actions](https://github.com/reactjs/react-redux/issues/263)

### Will having "one state tree" cause memory problems?  Will dispatching many actions take up memory?  

First, in terms of raw memory usage, Redux is no different than any other Javascript library.  The only difference is that all the various object references are nested together into one tree, instead of maybe saved in various independent model instances such as in Backbone.  Second, a typical Redux app would probably have somewhat _less_ memory usage than an equivalent Backbone app, because Redux encourages use of plain Javascript objects and arrays rather than creating instances of Models and Collections.  Finally, Redux only holds on to a single state tree reference at a time.  Objects that are no longer referenced in that tree will be garbage collected, as standard.

Redux does not store a history of actions itself.  However, the Redux DevTools do store actions so they can be replayed, but those are generally only enabled during development, and not used in production.

#### Further information
**Documentation**:
- [Docs: Async Actions](advanced/AsyncActions.md])


**Discussions**:
- [SO - Is there any way to "commit" the state in Redux to free memory?](http://stackoverflow.com/questions/35627553/is-there-any-way-to-commit-the-state-in-redux-to-free-memory/35634004)
- [Reddit - What's the best place to keep initial state?](https://www.reddit.com/r/reactjs/comments/47m9h5/whats_the_best_place_to_keep_the_initial_state/)




## React-Redux

### Why isn't my component re-rendering, or my mapStateToProps running?

Accidentally mutating or modifying your state directly is by far the most common reason why components do not re-render after an action has been dispatched.  Redux expects that your reducers will update their state "immutably", which effectively means always making copies of your data, and applying your changes to the copies.  If you return the same object from a reducer, Redux assumes that nothing has been changed, even if you made changes to its contents.  Similarly, React-Redux tries to improve performance by doing shallow equality reference checks on incoming props in `shouldComponentUpdate`, and if all references are the same, returns false to skip actually updating your original component.

It's important to remember that whenever you update a nested value, you must also return new copies of anything above it in your state tree.  If you have `state.a.b.c.d`, and you want to make an update to `d`, you would also need to return new copies of `c`, `b`, `a`, and `state`.  This [state tree mutation diagram](http://arqex.com/wp-content/uploads/2015/02/trees.png) demonstrates how a change deep in a tree requires changes all the way up.


Note that "updating data immutably" does _not_ mean that you must use the Immutable.js library, although that is certainly an option.  You can do immutable updates to plain JS objects and arrays using several different approaches: 
- Copying objects using functions like Object.assign() / \_.extend(), and array functions such as slice() and concat()
- The array spread operator in ES6, and the similar object spread operator that is proposed but not yet approved
- Utility libraries that wrap immutable update logic into simpler functions


#### Further information
**Documentation**:
- [Troubleshooting](Troubleshooting.md)
- [React-Redux: Troubleshooting](https://github.com/reactjs/react-redux/blob/master/docs/troubleshooting.md)
- [Recipes: Using the Object Spread Operator](recipes/UsingObjectSpreadOperator.md)

**Discussions**:
- [#1262 - Immutable data + bad performance](https://github.com/reactjs/redux/issues/1262)
- [React-Redux #235 - Predicate function for updating component](https://github.com/reactjs/react-redux/issues/235)
- [React-Redux #291 - Should mapStateToProps be called every time an action is dispatched?](https://github.com/reactjs/react-redux/issues/291)
- [SO - Cleaner/shorter way to update nested state in Redux?](http://stackoverflow.com/questions/35592078/cleaner-shorter-way-to-update-nested-state-in-redux)
- [Gist - state mutations](https://gist.github.com/amcdnl/7d93c0c67a9a44fe5761#gistcomment-1706579)
- [Pros and Cons of Using Immutability with React](http://reactkungfu.com/2015/08/pros-and-cons-of-using-immutability-with-react-js/)

### Why is my component re-rendering too often?

React-Redux implements several optimizations to ensure your actual component only re-renders when actually necessary.  One of those is a shallow equality check on the combined props object generated by the `mapStateToProps` and `mapDispatchToProps` arguments passed to `connect`.  Unfortunately, shallow equality does not help in cases where new array/object instances are created each time `mapStateToProps` is called.  A typical example might be mapping over an array of IDs and returning the matching object references, such as:

```js
let mapStateToProps = (state) => {
    return {
        objects : state.objectIds.map(id => state.objects[id])
    }
};
```

Even though the array might contain the exact same object references each time, the array itself is a different reference, so the shallow equality check fails and React-Redux would re-render the wrapped component.

The extra re-renders could be resolved by saving the array of objects into the state using a reducer, caching / memoizing the mapped array using the Reselect library, or implementing `shouldComponentUpdate` in the component and doing a more in-depth props comparison using a function such as `_.isEqual`.

For non-connected components, you may want to check what props are being passed in.  A common issue is having a parent component re-bind a callback inside its render function, like `<Child onClick={this.handleClick.bind(this)} />`.  That creates a new function reference every time the parent re-renders.  It's generally good practice to only bind callbacks once, in the parent component's constructor.



#### Further information

**Discussions**:
- [SO - Can a React-Redux app scale as well as Backbone?](http://stackoverflow.com/questions/34782249/can-a-react-redux-app-really-scale-as-well-as-say-backbone-even-with-reselect)
- [React.js pure render performance anti-pattern](https://medium.com/@esamatti/react-js-pure-render-performance-anti-pattern-fb88c101332f)


### How can I speed up my mapStateToProps?

While React-Redux does work to minimize the number of times that your `mapStateToProps` function is called, it's still a good idea to ensure that your `mapStateToProps` runs quickly and also minimizes the amount of work it does.  The common recommended approach is to create memoized "selector" functions using the [Reselect](https://github.com/reactjs/reselect) library.  These selectors can be combined and composed together, and selectors later in a pipeline will only run if their inputs have changed.  This means you can create selectors that do things like filtering or sorting, and ensure that the real work only happens if needed.

#### Further information
**Documentation**:
- [Recipes: Computed Derived Data](recipes/ComputingDerivedData.md)

**Discussions**:
- [#815 - Working with Data Structures](https://github.com/reactjs/redux/issues/815)
- [Reselect #47 - Memoizing Hierarchical Selectors](https://github.com/reactjs/reselect/issues/47)


### Why don't I have `this.props.dispatch` available in my connected component?

The `connect` function takes two primary arguments, both optional.  The first, `mapStateToProps`, is a function you provide to pull data from the store when it changes, and pass those values as props to your component.  The second, `mapDispatchToProps`, is a function you provide to make use of the store's `dispatch` function, usually by creating pre-bound versions of action creators that will automatically dispatch their actions as soon as they are called.

If you do not provide your own `mapDispatchToProps` function when calling `connect`, React-Redux will provide a default version, which simply returns the `dispatch` function as a prop.  That means that if you _do_ provide your own function, `dispatch` is _not_ automatically provided.  If you still want it available as a prop, you need to explicitly return it yourself in your `mapDispatchToProps` implementation.


#### Further information
**Documentation**
- [React-Redux API: connect](https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options)


**Discussions**:
- [React-Redux #89 - can i wrap multi actionCreators into one props with name?](https://github.com/reactjs/react-redux/issues/89)
- [React-Redux #145 - consider always passing down dispatch regardless of what mapDispatchToProps does](https://github.com/reactjs/react-redux/issues/145)
- [React-Redux #255 - this.props.dispatch is undefined if using mapDispatchToProps](https://github.com/reactjs/react-redux/issues/255)
- [SO - http://stackoverflow.com/questions/34458261/how-to-get-simple-dispatch-from-this-props-using-connect-w-redux/34458710](How to get simple dispatch from this.props using connect w/ Redux?)


### Should I only connect my top component, or can I connect multiple components in my tree?

Early Redux documentation advised that you should only have a few connected components, near the top of your component tree.  However, time and experience has shown that that generally requires a few components to know too much about the data requirements of all their descendants, and forces them to pass down a confusing number of props.  

The current suggested best practice is to categorize your components as "presentational" or "container", and extract a connected container component wherever it makes sense:

> Emphasizing "one container component at the top" in Redux examples was a mistake. Don't take this as a maxim.  Try to keep your presentation components separate. Create container components by connecting them when it's convenient.  Whenever you feel like you're duplicating code in parent components to provide data for same kinds of children, time to extract a container.  Generally as soon as you feel a parent knows too much about "personal" data / actions of its children, time to extract a container.

In general, try to find a balance between understandable data flow and areas of responsibility with your components.

#### Further information
**Documentation**:
- [Basics: Usage with React](basics/UsageWithReact.md)


**Discussions**
- [Presentational and Container Components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)
- [Twitter - emphasizing "one container" was a mistake](https://twitter.com/dan_abramov/status/668585589609005056)
- [#419 - Recommended usage of connect](https://github.com/reactjs/redux/issues/419)
- [#756 - container vs component?](https://github.com/reactjs/redux/issues/756)
- [#1176 - Redux+React with only stateless components](https://github.com/reactjs/redux/issues/1176)
- [SO - can a dumb component use a Redux container?](http://stackoverflow.com/questions/34992247/can-a-dumb-component-use-render-redux-container-component)


## Community

### Are there any larger, "real" Redux projects?


## Miscellaneous

### How can I implement authentication in Redux?

Authentication is essential to any real application. When going about authentication you must keep in mind that nothing changes with how you should organize your application and you should implement authentication in the same way you would any other feature.  It is straightforward:

1. Create action constants for LoginSuccess, LoginFailure, etc.

2. Create actionCreators that take in a type, credentials, a flag that signifies if authentication is true or false, a token, or errorMessage as a payload.

3. Create an async action creator with redux-thunk middleware or any middleware you see fit to fire a network request to an api that returns a token if the credentials are valid and proceeds to save in local storage or a response to the user if it is a failure which is handled by the appropriate actionCreators that you made in step 2.

4. Create a reducer that can return the next state for each possible authentication case (LoginSuccess, LogoutFailure).

#### Further information

**Discussions**:
- [Authentication with JWT by Auth0](https://auth0.com/blog/2016/01/04/secure-your-react-and-redux-app-with-jwt-authentication/)
- [Tips to Handle Authentication in Redux](https://medium.com/@MattiaManzati/tips-to-handle-authentication-in-redux-2-introducing-redux-saga-130d6872fbe7)
- [react-redux-jwt-auth-example](https://github.com/joshgeller/react-redux-jwt-auth-example)
- [redux-auth](https://github.com/lynndylanhurley/redux-auth)
