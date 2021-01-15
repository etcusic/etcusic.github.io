---
layout: post
title:      "Setting Up REDUX"
date:       2021-01-15 23:48:11 +0000
permalink:  setting_up_redux
---


I had quite a time setting up Redux in my application, so I figured I'd blog out what I did to implement it for future reference. I'll go step by step, setting it up in reverse order of the direction the data moves in. That way, once we have an event to be triggered, we should be able to test it out immediately.

To begin, we'll need to install `redux` and `react-redux` - and then implement that in src/index.js file. Next in src/index.js we'll bring in Provider, createStore, and rootReducer. The Provider will contain `store` which is created by `createStore` and where our application's state will reside, and `rootReducer` will house all of the reducers that will handle the different state changes triggered by whatever actions we may want to implement. In the end, my src/index.js file looked like this:
```
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import { Provider } from 'react-redux'
import { createStore, compose, applyMiddleware } from 'redux'
import rootReducer from './reducers'
import thunk from 'redux-thunk'
import reportWebVitals from './reportWebVitals';

const composeEnhancers = window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ || compose;

let store = createStore(rootReducer, composeEnhancers(applyMiddleware(thunk)))

ReactDOM.render(<Provider store={store}><App /></Provider>, document.getElementById('root'));

reportWebVitals();
```
Now over to the reducers. Again, this is where state changes will be handled, waiting to be triggered by an action. These reducers will be switch statements that receive an action object, test against the `action.type` and then the matching case statement will return the `action.payload`, which will be a copy of the current state with the desired modifications made to it. Each reducer should have its own file and be imported into the `combineReducers` function provided by Redux. Also, each reducer should produce a default of `state` Here is what a simple implementation looks like:
```
// src/reducers/index.js

import { combineReducers } from 'redux'
import game from './gameReducer'

export default combineReducers({
    game
})


// src/reducers/game.js

const gameReducer = (state = {practice: "Initial state"}, action) => {
    switch(action.type){
        case "SOMETHING": 
            return action.payload
        default:
            return state
    }
}

export default gameReducer
```
Next, we'll implement an action, which will ultimately trigger a reducer. For this action, we'll be passing through the current state, copying it into a new object (so that we won't manipulate state directly), make the correct modifications to the copy of state, and then returning inside a Redux `dispatch` function to be read by the reducer. Here is a simple one:
```
export const changeSomething = (currentState) => {
    let object = currentState
    object.practice = "NEW STATE, YAY!!"
    return (dispatch) => dispatch({ type: "SOMETHING", payload: object})
}
```
Now that we've got an action set up, we can set that up in a component to be triggered. First we'll need to connect the component to the action before it can be triggered. For this we'll use the `connect` function that we get from redux and import the action to be connected to the component. Just one more thing we need, though before we're done hooking up the component - `mapStateToProps`. This function will take the new state, map over it, and then send down the necessary props to the component. This closes the loop, as the event triggers an action, which returns a `dispatch` to a reducer that returns the new state, which is mapped over and sent down as props from the Provider's store. So here is the component's code once its been connected to mapState to props and the appropriate actions(s):
```
import React, { Component } from 'react'
import { connect } from 'react-redux'
import { changeSomething } from '../actions/index'

// GameComponent goes here

const mapStateToProps = state => {
  return {
    practice: state.game.practice
  }
}

export default connect(mapStateToProps, { changeSomething })(GameComponent)
```
