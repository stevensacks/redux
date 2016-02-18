# Redux FAQ


## General

### Can Redux only be used with React?  

Redux can be used as a data store for any UI layer.  The most common usage is with React, but there are bindings available for Angular, Vue, Mithril, and more.  Redux simply provides a subscription mechanism, which can be used by any other code.


### Do I need to have a particular build tool to use Redux?

Redux is written in ES6 and built for production with Webpack and Babel.  However, it should be usable in any Javascript build process.  There is also a UMD build that allows usage without any build process at all.


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
- [#1400 - Is passing top-level state object to branch reducer an anti-pattern?](https://github.com/reactjs/redux/issues/1400)


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

### Can I put functions, promises, or other non-serializable items in my store state?

### How do I organize nested/duplicate data in my state?


## Store Setup

### Can I / should I create multiple stores?

### Is it OK to have more than one middleware chain in my store enhancer?

### How do I subscribe to only a portion of the state?


## Actions

### Why should my action types be constants?

### Why should "type" be a string, or at least serializable?

### Should I have a 1-1 mapping between reducers and actions?

### How can I represent "side effects" such as AJAX calls?

### Should I dispatch multiple actions in a row from one action creator?


## Code Structure

### How should I group my action creators and reducers in my project?

### Where should my selectors go?

### How should I split my logic between reducers and action creators?


## Performance


### Won't calling "all my reducers" for each action be slow?

### Do I have to deep-copy my state in a reducer? Isn't copying my state going to be slow?

### How can I reduce the number of store update events?

### Will having "one state tree" cause memory problems?


## React-Redux

### Why isn't my component re-rendering, or my mapStateToProps running?

### Why is my component re-rendering too often?

### How can I speed up my mapStateToProps?


## Community

### Are there any larger, "real" Redux projects?
