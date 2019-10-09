---
title: "React Custom Hooks: What, Why and How"
date: 2019-10-05T18:31:28+06:00
author: "Rohan Faiyaz Khan"
bigimg: [{src: "/img/hook.jpg", desc: "https://unsplash.com/photos/f2LYxnmnKxI"}]
tags: ["react", "hooks"]
---

Now that its been a year since React hooks has been released, I can safely say that I absolutely love them and cannot imagine writing React components without them anymore.

However hooks come with some of their own limitations, one of the biggest being that they must be used in a React functional component. This is by design, because we would like our stateful logic to be tied to the component where they are needed and not just any function. However we do sometimes want to reuse our stateful logic between components. Before hooks, this was only possible through higher-order components and render props, a pattern common in libraries such as Redux with its `connect(Component)` or React Router's `withRouter(Component)`. I personally don't prefer writing higher-order components cause they are a slightly difficult abstraction that come with their set of gotchas. Thankfully, hooks provide a much easier way to share stateful logic without requiring you learn a difficult abstraction, and that way is custom hooks,

Custom hooks are simply a wrapper function surrounding our existing hooks. That's it! The only catch is that in order for React to recognize that a function is a custom hook, its name must start with `use`. The same [rules for using hooks](https://reactjs.org/docs/hooks-rules.html) also apply to custom hooks, for instance they cannot to nested in a condition or a loop and also cannot be called outside a functional component or yet another hook. 

Note that custom hooks require you to be familiar with hooks themselves, atleast [useState](https://reactjs.org/docs/hooks-state.html) and [useEffect](https://reactjs.org/docs/hooks-effect.html). If you haven't used I'd really recommend looking into them first cause they are the core building blocks of every custom hook you could make.

Let's do a quick example to see we write custom hooks. Suppose we have an imaginary app where once users login we fetch make an API request to fetch a list of their friends. Using functional components and hooks our component would look like this.

```js

import React, {useState, useEffect} from 'react'

function Dashboard(props){
	const [friends, setFriends] = useState([])
	const [error, setError] = useState({})
	ussEffect(() => {
		if(props.user.loggedIn){
			fetch(`/api/${props.user.id}/friends`).then(response => {
				response.json().then( friends => setFriends(friends))
			})
			.catch(error => setError(error)
		}
	}, [props.user.isLoggedIn, props.user.id])

	return <div>
		{ friends.length > 0 ? 
			friends.map(friend => <div key={friend.id}>{friend.name}</div> 
			: error.message
		</div>
}

export default Dashboard
```

Suppose we would like to replicate this behaviour in another component, such as a chatbox that displays friends that are online. Our stateful logic would mostly be the same. Instead of copying our code over, a better approach to the problem would be to extract out this logic into a custom hook. We'll call our custom hook `useFriendList`.

```js
// useFriendList.js

export default function useFriendList(user){
	const [friends, setFriends] = useState([])
	const [error, setError] = useState({})
	ussEffect(() => {
		if(user.loggedIn){
			fetch(`/api/${user.id}/friends`).then(response => {
				response.json().then( friends => setFriends(friends))
			})
			.catch(error => setError(error)
		}
	}, [user.id, user.isLoggedIn])

	return [friends, error]
}
```

Now we can simply refactor our previous component as such.

```js
import React from 'react'
import useFriendList from './useFriendList'

function Dashboard(props){
	const [friends, error] = useFriendList(props.user)

	return <div>
		{ friends.length > 0 ? friends.map(friend => <div key={friend.id}>{friend.name}</div> 
			: error.message
		</div>
}

export default Dashboard
```

And that's really all there is to it. Not only does our component look a lot cleaner, but any time we need to reuse this logic in a component, we just call our custom hook.

Personally I like this pattern a lot more than higher-order components because unlike HOC, custom hooks do not require you to think any differently than just using hooks normally. All you have to do is throw your existing logic in a wrapper function. This makes concepts such as component composition a lot more accessible and easier to get started with. If you found custom hooks to be a difficult concept I hope this post makes you want to give it a try.
