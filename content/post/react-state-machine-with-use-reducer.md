---
title: "Turning your React Component into a Finite State Machine With useReducer"
date: 2020-01-25T23:38:01+06:00
draft: false
author: "Rohan Faiyaz Khan"
bigimg: [{src: "/img/steps.jpg", desc: "State machine with use reducer"}]
tags: ["react", "hooks", "state", "state-machine"]
---

<a href="https://unsplash.com/photos/tXxyqjaXvRs"><small>Photo by Stéphane Mingot</small></a>

## Why state machines are relevant to frontend development

A finite state machine is not a new concept in the world of computing or mathematics. It is a mathematical model that be in one a few finite states. Transition to a new state could depend on the previous state and a set of external factors.

This model has become more relevant recently in the field of UI development because we have shifted a lot of the state management to the frontend now. Being a React developer one of the first things I learned was how to manage state inside a component and how to manage global state with Redux. The naive approach I usually used was to have booleans such as `isLoading` and `isError` and render my component based on that.

```jsx
const MyComponent = () => {
   const [state, setState] = useState({ isLoading: false, isError: false })

   const clickHandler = (e) => {
      setState({ isLoading: true })
      sendNetworkRequest()
         .then(() => setState({ isLoading: false }))
         .catch(() => setState({ isError: true, isLoading: false })
   }

   return (
       <div>
          <button onClick={clickHandler}>Send request</button> 
          { state.isLoading? "Loading" : state.isError ? "There has been an error" : "Success!" }
       </div>
   )

}
```
This is fine for the most part. The code above is very easy to skim through and it is easy to tell what conditions in state is doing what, Problem is though this approach scales horribly. In real life scenarios there are more factors that could change the loading and error states, and there might also be a success or failure state or even an idle state, and state transition may depend on the previous state. What starts innocently as a simple boolean state management system turns into a nightmare plate of spaghetti. 

```jsx
const MyComponent = (props) => {
   const [state, setState] = useState({ 
      isLoading: false, 
      isError: false,
      isSuccess: false,
      isIdle: true
   })

   const clickHandler = (e) => {
      setState({ isLoading: true })
      sendNetworkRequest()
         .then((result) => {
             if(/* some arbritrary condition */){
                setState({ isLoading: false, isIdle: false, isSuccess: true }))
             }else if(/* some other arbitrary condition */){
                setState({ isIdle: false, isSuccess: true }))
             }
         }
         .catch(() => setState({ isSuccess: false, isError: true, isLoading: false })
   }

   return (
       <div>
          { state.isIdle ? "Click to send request"
                         : state.isLoading ? "Loading" 
                         : state.isError ? "There has been an error" : "Success!" }
       </div>
   )

}
```

I can tell you from personal experience that an example like this is very much possible and an absolute nightmare to deal with. We have so many conditional checks that its very hard to debug exactly what is happening. There is also several bugs, for example, when sending a request we do not set `isIdle` to `false` and since that is the first check in the return statement, the loading state never gets shown. These kind of bugs are very hard to spot and fix, and even harder to test.

While there are many ways to fix this component, the method that I prefer is to turn it into a finite state machine. Notice that the states we have are all mutually exclusive, i.e. our component can only exist in one possible state at one time- either idle, success, failure or loading. If we limit ourselves to these possibilities, then we can limit the possible transitions as well.

## The state-reducer pattern

The object state pattern is something I have [discussed in detail before](https://rohanfaiyaz.com/post/context/) and is likely familiar with anyone that has used redux. Its a pattern for changing state using an action and the existing state as inputs, Using it, we can limit our states and our actions which limits the number of possibilities we will have to deal with to the below.

```js
const ComponentStates = Object.freeze({
   Idle: "IDLE",
   Loading: "LOADING",
   Success: "SUCCESS",
   Failure: "FAILURE"
})   

const ActionTypes = Object.freeze({
   RequestSent: "REQUEST_SENT",
   RequestSuccess: "REQUEST_SUCCESS",
   RequestFailure: "REQUEST_FAILURE"
})
```
This is very helpful for a number of reasons. If we know there are only going to be three possible actions we only have to account for three possible mutations to state. This of course becomes exponentially more complicated if we also account for current state but even so it is better than what we had before. Furthermore we don not have to juggle multiple conditional checks, we only have to keep track of what condition dispatch which actions, and what actions lead to what state change. This in my experience is a far easier mental debt.

```js
function reducer(state, action){
   switch(action.type){
      case ActionTypes.RequestSent:
         return ComponentStates.Loading
      case ActionTypes.RequestSuccess:
         return ComponentStates.Success
      case ActionTypes.RequestFailure:
         return ComponentStates.Failure      
      default:
         return ComponentStates.Idle
      }
}
```

## The useReducer hook

Finally we will use `useReducer` which is one of the base hooks provided by React. It is basically an extension of `useState` except that it takes a reducer function and initial state as arguments and returns a dispatch function along with state.

For those unfamiliar with redux, the dispatch function is used to dispatch an action, which includes a `type` (one of our action types) and an optional payload. The action is then _reduced_ by our reducer function, resulting in a new state. With this knowledge we can complete our state machine.

```jsx
const MyComponent = (props) => {
   const initialState = ComponentStates.Idle
   const [state, dispatch] = useReducer(reducer, initialState)

   const clickHandler = (e) => {
      dispatch({ type: ActionTypes.RequestSent })
      sendNetworkRequest()
         .then((result) => {
             if(/* some arbritrary condition */){
                dispatch({ type: ActionTypes.RequestSuccess }) 
             }
         }
         .catch(() => {
             dispatch({ type: ActionTypes,RequestFailed })
         })
   }

   return (
       <div>
          { state === ComponentStates.Idle ? "Click to send request"
                         : state === ComponentStates.Loading ? "Loading" 
                         : state === ComponentStates.Failure ? "There has been an error" 
                         : "Success!" }
       </div>
   )

}
```

## You can implement this however you like

This is simply my solution to a complex problem and your problem domain may not match with mine. However I hope this gives some inspiration as to how you could implement a state management solution yourself. Thank you for reading and I hope this helps!
