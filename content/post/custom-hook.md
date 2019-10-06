---
title: "React Custom Hooks: What, Why and How"
date: 2019-10-05T18:31:28+06:00
author: "Rohan Faiyaz Khan"
bigimg: [{src: "/img/hook.jpg", desc: "https://unsplash.com/photos/f2LYxnmnKxI"}]
draft: true
---

Its been a year since the React team made their big hooks release. And like many others have stated before me, I absolutely love hooks and one of biggest reasons for that is the ability to write custom hooks.

So what are custom hooks? They are nothing more than a wrapper functions for existing React hooks. I know, I know that doesn't sound very exciting but bear with me for a second. A big limitiation of hooks are that they can only be used inside a React functional component. This is by design because the problems of state management and side effects are inherently tied to a component. As a result hooks cannot be abstracted out to a helper function to be reused elsewhere, unless of course, the said helper function is itself a hook!

Note that custom hooks require you to be familiar with hooks themselves, atleast [useState](https://reactjs.org/docs/hooks-state.html) and [useEffect](https://reactjs.org/docs/hooks-effect.html). If you haven't used I'd really recommend looking into them first cause they are the core building blocks of every custom hook you could make. Some of you might be saying, _but what useReducer, useMemo, useContext...? They are all important hooks!_ Yes they are but most of the other built-in are either used for performance optimizations or for specific use cases. They are important tools, but certainly not as essential as `useState` and `useEffect`.

Let's look at a simple example. Say we have an imaginary application