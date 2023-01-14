---
title: Redux & Thunk
date: 2022-12-28 14:11:36
tags:
---
* Redux: to save global state


* when state changes, render UI components that subscribe to store
* store: manage current state.
  - created by passing in a reducer
  - `store.getState()`
  - to update state: `store.dispatch(action)` to run reducer inside store to update state; should use `import { useSelector, useDispatch } from 'react-redux'` and `const dispatch = useDispatch()` as we cannot access store
* action: obj
```javascript
{
  type: "domain/eventName"
  payload: 3
}
```
* reducer: (state, action) => newState, sync, no side effect, immutable update
  - can only make seemingly mutating logic inside createSlice and createReducer, which use Immer
    ```javascript
    function reducerWithImmer(state, action) {
      state.first.second[action.someId].fourth = action.someValue
    }
    ```
* slice: logic for a feature, including reducers, actions, etc

  - create
    ```javascript
    createSlice({name: "domain", initialState: {value: 0},
      reducers: {"eventName1": state => state.value += 1,
                "eventName2": (state, action) => {}}})
    ```
    ```javascript
    import { createSlice } from '@reduxjs/toolkit'

    export const counterSlice = createSlice({
    name: 'counter',
    initialState: {
    value: 0
    },
    reducers: {
    increment: state => {
    // Redux Toolkit allows us to write "mutating" logic in reducers. It
    // doesn't actually mutate the state because it uses the immer library,
    // which detects changes to a "draft state" and produces a brand new
    // immutable state based off those changes
    state.value += 1
    },
    decrement: state => {
    state.value -= 1
    },
    incrementByAmount: (state, action) => {
    state.value += action.payload
    }
    }
    })

    export const { increment, decrement, incrementByAmount } = counterSlice.actions

    export const selectCount = state => state.counter.value

    export default counterSlice.reducer
    ```
  - auto-generate action creator (i.e. a func to return an action)
  ```javascript
  console.log(counterSlice.actions.increment())
  // {type: "counter/increment"}
  ```
  - generate reducer
    ```javascript
    const newState = counterSlice.reducer(
    { value: 10 },
    counterSlice.actions.increment()
    )
    console.log(newState)
    // {value: 11}
    ```

* selector: a function to extract info from state: `const selectCounterValue = state => state.value`
  - `import { useSelector } from 'react-redux'` and `const someStateValue = useSelector(selector)`
  - e.g. `const count = useSelector(selectCount)`
  - if dispatch, useSelector will re-run selector function. If the selector returns a different value than last time, useSelector will make sure component re-renders.

* thunk: Redux function that can have asynchronous logic, written using:
  - inside thunk function, (dispatch, getState) => {}
  - outside creator function, which creates and returns the thunk function
  ```javascript
  export const incrementAsync = amount => (dispatch => {
    setTimeout(() => {
      dispatch(incrementByAmount(amount))
    }, 1000)
  });
  ```
  ```javascript
  store.dispatch(incrementAsync(5))
  ```
  ```javascript
  const fetchUserById = userId => {
    return async (dispatch, getState) => {
      try {
        const user = await userAPI.fetchById(userId)
        dispatch(userLoaded(user))
      } catch (err) {
      }
    }
  }
  ```
  - requires redux-thunk middleware, Redux Toolkit's configureStore function already sets that up
* provide store
```javascript
import React from 'react'
import ReactDOM from 'react-dom'
import './index.css'
import App from './App'
import store from './app/store'
import { Provider } from 'react-redux'
import * as serviceWorker from './serviceWorker'

// Provider
ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
)
```
