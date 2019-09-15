---
title: "Sharing more with React's Context API"
date: 2019-09-14T11:46:58+06:00
bigimg: [{src: "/img/bicycle.jpg", desc: "Photo by Pop and Zebra"}]
share_img: "/img/bicycle.jpg"
author: "Rohan Faiyaz Khan"
---

## The pains of growing state

In learning React, one of the first challenges I faced was figuring out state management. State is a vital part of any application that has more complexity than a simple blog or brochure site. React has a fantastic toolset to manage component level state both in the case of functional components with hooks, and class based components. However global state is a bit of a different story.

Almost every advanced feature such as authentication, shopping carts, bookmarks etc. heavily rely on state that multiple components need to be aware of. This can be done by passing state through props but as an application grows this gets complicated very fast. We end up having to pipe state through intermediary components and any change in the shape of the state needs to reflected in all of these components. We also end up with a bunch of code unrelated to the concern of the intermediary component, so we learn to ignore it. And if Uncle Bob taught me anything, the code we ignore is where the bugs hide. 

## The solution: Redux

[ Redux ](https://redux.js.org/) was born out of the problem of global state handling. Built by [ Dan Abramov ](https://overreacted.io) and his team, Redux provided a global store independant of local state that individual components could access. Furthermore it comes with some high level abstractons for dealing with state, such as the state reducer pattern.

_Wait, slow down, the state reducer what now?_

Yes I hear you, for this was my exact reaction when I heard of these words put together for the first time. The reducer pattern is a popular pattern even outside of Redux, and implements a way to change state. A reducer function is a _pure_ function (i.e. has no external state or side effects) that simply takes in the previous state and an action, and returns the new state. It looks like this below.


```javascript
function reducer(state, action){
    switch(action){
        case "increment":
            return state + 1
        case "decrement":
            return state - 1
        default:
            return state
    }
}
```
This pattern allows us to alter state predictably which is important because we need to how our application might react to changes in state. Under the pattern, mutating state directly is heavily discouraged.  

Redux also provides us with the action creator pattern, which is simply a way to organise how we dispatch our actions. Combined with the state reducer pattern, this gives us great tools to organize our global state management.

### Sounds good, so what's the problem?

While redux is great and I personally am a big fan of it, it has its fair share of detractors. 

- The first problem a lot of people have is that it is very boilerplate-y. This is especially apparent when you have an app that initially doesn't need global state, and then later on you realize you do and then \***_BOOM_**\* 200+ lines added in one commit. And every time global state has to be pulled in for a component, this extra boilerplate has to added in.

- Redux is opinionated and imposes limitations. Your state has to be represented as objects and arrays. Your logic for changing states have to be pure functions. These are limitations that most apps could do without.

- Redux has a learning curve of its own. This true for me personally, because React seemed very fun as a beginner until I hit the wall of Redux. These advanced high level patterns are something a beginner is not likely to appreciate or understand.

- Using Redux means adding about an extra 10kb to the bundle size, which is something we would all like to avoid if possible. 

Several other state management libraries have propped up such as MobX to solve the shortcomings of Redux, but each have their own trade-offs. Furthermore, all of them are still external dependencies that would bulk up the bundle size.

_But surely something this important has a native implementation? Right?_

Well there wasn't, until...

## All hail the magical context!

To be fair, the Context API has been around for a while, but it has gone through significant changes and alterations before it became what it is today. The best part about it is that it does not require any `npm install` or `yarn install`, it's built in with React, I personally have found the current iteration of the Context API to be just as powerful as Redux, especially when combined with hooks.

But there was a roadblock to learning, that being the official React documentation is terrible at explaining how powerful the Context API is. As a result, I dug through it and implemented a simple login system so that you don't have to. 

## Enough talk, show me how this works already

All we will be doing is logging in (using a fake authentication method wrapped in a Promise), and changing the title with the username of the logged in user.  **[ If you'd rather skip all the explanantion and just look at the code, feel free to do so. ] (https://github.com/rohanfaiyazkhan/react-context-api-example)**

{{<figure src="/img/context.gif">}}

The first thing we need to do to use context is `React.createContext(defaultValue)`. This is a function that returns an object with two components:

- `myContext.Provider` - A component that provides the context to all its child elements. If you have used Redux before, this does the exact same thing as the `Provider` component in the react-redux package
- `myContext.Consumer` - A component that is used to consume a context. As we shall soon see however, this will not be needed when we use the `useContext` hook

Lets use this knowledge to create a store for our state. 

```jsx
// store.js

import React from 'react';

const authContext = React.createContext({});

export const Provider = authContext.Provider;
export const Consumer = authContext.Consumer;
export default authContext;
```
Notice below that the `defaultValue` parameter passed to `createContext` is an empty object. This is because this parameter is optional, and is only read when a Provider is not used.

Next we have to wrap our application in the `Provider` so that we can use this global state. `Provider` needs a prop called `value` which is the value of the state being shared. We can then use the `useContext` hook in the child component to retrieve this value.

```jsx
function App(){
    return (
        <Provider value={someValue}>
            <ChildComponent />
        </Provider>
    )
}

function ChildComponent(){
    const contextValue = useContext(myContext)
    return <div>{contextValue}</div>
}
```
However you might notice a problem with this method. We can only change the value of the state in the component containing the Provider. What if we want to trigger a state change from our child component?

Well remember the reducer state pattern I talked about above? We can use it here! React provides a handy `useReducer` hook which takes in a `reducer` function and an `initialState` value and returns the current state and a dispatch method. If you have used redux before, this is the exact same reducer pattern we would observe there. Then we have pass the return value of the `useReducer` hook as the value inside `<Provider>`.

Lets define a reducer.

```js
// reducers/authReducer

export const initialAuthState = {
	isLoggedIn: false,
	username: '',
	error: ''
};

export const authReducer = (state, action) => {
	switch (action.type) {
		case 'LOGIN':
			return {
				isLoggedIn: true,
				username: action.payload.username,
				error: ''
			};
		case 'LOGIN_ERROR':
			return {
				isLoggedIn: false,
				username: '',
				error: action.payload.error
			};
		case 'LOGOUT':
			return {
				isLoggedIn: false,
				username: '',
				error: ''
			};
		default:
			return state;
	}
};
```

Now we can use our reducer in our `<Provider>`.

```jsx
// App.js 

import React, { useReducer } from 'react';
import Router from './components/Router';
import { Provider } from './store';
import { authReducer, initialAuthState } from './reducers/authReducer';

function App() {
	const useAuthState = useReducer(authReducer, initialAuthState);
	return (
		<Provider value={useAuthState}>
			<Router />
		</Provider>
	);
}

export default App;
```
Now all components in our application will have access to the `state` and the `dispatch` method returned by `useReducer`. We can now use this `dispatch` method in our login form component. First we will grab the state from our context so we can check if the user is logged in so we can redirect them or if we need to render an error. Next we will attempt to login (using our fake authentication method) and dispatch an action based on either authentication is successful or not.

```jsx
// components/LoginForm.jsx

import React, { useState, useContext, Fragment } from 'react';
import { Link, Redirect } from 'react-router-dom';
import authContext from '../store';
import attemptLogin from '../auth/fakeAuth';

const LoginForm = () => {
	const [ state, dispatch ] = useContext(authContext);
        const { isLoggedIn, error } = state;

	const [ fakeFormData, setFormData ] = useState({
            username: "Rohan", 
            password: "rohan123"
        });

	function onSubmit(event) {
		event.preventDefault();
		attemptLogin(fakeFormData)
			.then((username) => {
				dispatch({
					type: 'LOGIN',
					payload: {
						username
					}
				});
			})
			.catch((error) => {
				dispatch({
					type: 'LOGIN_ERROR',
					payload: {
						error
					}
				});
			})
			.finally(() => {
				setLoading(false);
			});
	}

	return (
		<Fragment>
			{isLoggedIn ? (
				<Redirect to="/" />
			) : (
				<Fragment>
					{error && <p className="error">{error}</p>}
					<form onSubmit={onSubmit}>
						<button type="submit">Log In</button>
					</form>
				</Fragment>
			)}
		</Fragment>
	);
};

export default LoginForm;
```
Finally we will wrap up the landing component to show the logged in user's username. We will also toggle the welcome message to prompt a login or logout based on whether the user is already logged in or not, and will create a method to dispatch a logout.

```jsx
// components/Hello.jsx

import React, { Fragment, useContext } from 'react';
import { Link } from 'react-router-dom';
import Header from './Header';
import authContext from '../store';

const Hello = () => {
	const [ { isLoggedIn, username }, dispatch ] = useContext(authContext);
	const logOut = () => {
		dispatch({
			type: 'LOGOUT'
		});
	};
	return (
		<Fragment>
			<Header>{`Well hello there, ${isLoggedIn ? username : 'stranger'}`}</Header>
			{isLoggedIn ? (
				<p>
					Click <Link to="/" onClick={logOut}>here</Link> to logout
				</p>
			) : (
				<p>
					Click <Link to="/login">here</Link> to login
				</p>
			)}
		</Fragment>
	);
};

export default Hello;
```

## And there you have it

We now have a fully functioning context-based state management system. To summarize the steps needed to create it:

- We created a store using `React.createContext()`
- We created a reducer using the `useReducer` hook
- We wrapped our application in a `Provider` and used the reducer as the value
- We used the `useContext` to retrieve the state and dispatched actions when necessary

You might be asking now whether this can completely replace Redux. Well, maybe. You might notice that we had to implement our own abstractions and structure when using the Context API. If your team is already used to the Redux way of doing things, then I don't see a lot of value in switching. But if you or your team does want to break away from Redux I would certainly recommend giving this a try.

Thank you for reading, and I hope you found this post useful.