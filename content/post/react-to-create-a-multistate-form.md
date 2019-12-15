---
title: "Creating a reusable multistep form component in React"
date: 2019-10-22T23:18:44+06:00
draft: true
author: Rohan Faiyaz Khan
bigimg: [{src: "/img/steps.jpg", desc: "https://unsplash.com/photos/tXxyqjaXvRs"}]
tags: ["react", "multistep-form", "declarative", "composition"]
---

## Declarative Composition

If you hear these two words thrown around a lot but are not sure what they mean, then don't worry cause you are not alone. Declarative composition is a core philosophy that went into the design of React and how the React team wants you to write components, but don't worry, it's a simpler than people make it out to be.

To put it simply, _declarative_ in React means we should be able to tell what our components are doing by looking at how they are being used. The more closely our components resemble HTML elements, the less time we take to figure out what the component is doing and the easier it is to share components among team members. _Composition_ is a pattern where we compose our larger components of smaller atomic units. This can be facilitated through smaller components or through hooks. If you work with React for a while, you will find that a lot of React's design has been built to facilitate _declarative_ components that can be _composed_.

The best way to showcase is to write a component with declarative compostion in mind. A few weeks back I ran into a problem where I needed a multistep form component. You might be quite familiar with forms like this which are broken into multiple steps, and the state is persisted between the steps. It is an important UX consideration to not overload your users with too much information at once. 
