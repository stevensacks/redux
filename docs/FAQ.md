# Redux FAQ


## Reducers

### How do I share state between two reducers?


### Do I have to use combineReducers?

### Do I have to use a switch statement to handle actions?


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

### How should I group my actions and reducers in my project?

### Where should my selectors go?


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
