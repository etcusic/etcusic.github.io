---
layout: post
title:      "Setting Up Redux"
date:       2021-01-15 18:48:12 -0500
permalink:  setting_up_redux
---


As of writing this my application is still pretty small, so incorporating Redux felt a bit less intuitive. After getting the hang of using the Redux store, though, I certainly understand how having it to manage global state can be super helpful as I grow an application. The setup was a bit trickier than I was anticipating, so I figured I’d document the bare bones of its setup for future reference.

To begin, we'll need to install `redux` and `react-redux` - and then implement that in src/index.js file. Next in src/index.js we'll bring in Provider, createStore, and rootReducer. The Provider will contain `store` which is created by `createStore` and where our application's state will reside, and `rootReducer` will house all of the reducers that will handle the different state changes triggered by whatever actions we may want to implement. And then of course we need to add `thunk` so that the application can handle asynchronous changes to gobal state. In the end, my src/index.js file looked like this:
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
Now over to the reducers. Again, this is where state changes will be handled, waiting to be triggered by an action. These reducers will be switch statements that receive an action object, test against the `action.type` and then the matching case statement will return the `action.payload`, which will be a copy of the current state with the desired modifications made to it. Each reducer should have its own file and be imported into the `combineReducers` function provided by Redux. Also, each reducer should produce a default of `state` Here is what my simple implementation looks like:
```
// src/reducers/index.js

import { combineReducers } from 'redux'
import user from './userReducer'
import decks from './decksReducer'
import gameLogs from './gameLogsReducer'

export default combineReducers({
    user, decks, gameLogs
})


// src/reducers/decksReducer.js

const decksReducer = (state = [], action) => {
    switch(action.type){
        case "LOAD_DECKS": 
            let decks = action.payload
            return decks

        default:
            return state
    }
}

export default decksReducer
```
Next, we'll implement an action, which will ultimately trigger a reducer. For this action, we'll be passing through the current state, copying it into a new object (so that we won't manipulate state directly), make the correct modifications to the copy of state, and then returning it inside a Redux `dispatch` function to be read by the reducer. Here is a simple one that retrieves all the decks from the Rails API:
```
export const fetchDecks = () => {
    return(dispatch) => {
        return fetch('http://localhost:3001/decks')
        .then(resp =>  resp.json())
        .then(decks => {
            dispatch({ type: "LOAD_DECKS", payload: decks })
        })
    }
}
```
Now that we've got an action set up, we can incorporate it into a component to be triggered. First we'll need to connect the component to the action before it can be triggered. For this we'll use the `connect` function that we get from Redux and import the action to be connected to the component. Just one more thing we need, though, before we're done hooking up the component - `mapStateToProps`. This function will take the new state, map over it, and then send down the necessary props to the component. This closes the loop, as the event triggers an action, which returns a `dispatch` to a reducer that returns the new state, which is mapped over and sent down as props from the Provider's store. For this application, I call the `fetchDecks()` function once the app has been started, so here is the `HomePage` component's code once its been connected to mapStateToProps and the appropriate actions(s):
```
import React, { Component } from 'react'
import { connect } from 'react-redux'
import SidePanel from '../containers/SidePanel'
import MainSection from '../containers/MainSection'
import { fetchDecks } from '../actions/index'

export class HomePage extends Component {

  componentDidMount(){
    if (this.props.decks.length === 0){
      this.props.fetchDecks()
    }
  }
  
  render() {
    return (
      <div className="row">
        <SidePanel decks={ this.props.decks } />
        <MainSection />
      </div>
    )
  }
}

const mapStateToProps = state => {
  return {
    decks: state.decks
  }
}

export default connect(mapStateToProps, { fetchDecks })(HomePage)
```
