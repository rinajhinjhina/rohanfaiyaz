---
title: "React Declarative Composition"
date: 2019-10-26T11:41:03+06:00
draft: true
---

## What on earth is declarative composition?

__Declarative composition__ is a pattern fundamental to React. It has been a focus by the core team for good reason, as a pattern it allows component reusability without sacrificing flexibility. But what does it actually mean?

Well there are two parts to the phrase. __Declarative__ programming is a programming paradigm that expresses logic without describing its implementation. That's a bit of a mouthful, but in the context of React it basically means we want to be to see what our components do by looking at them, and not by digging into their implementation. Think of this component below.

```jsx
<Slider components={[ <img src={image1} />, <img src={ image2 } />, <img src={ image3 } />]} slideFunction={smoothSlide} />
```

This component, while functionally works just fine, has two problems. Firstly, its use of image tags as a prop consisting of an array of child components is very unclear. If you encounter this line while skimming through code, you would have to stop and do a double take to figure what is going on and that is exactly what we don't want. 

Secondly, we are passing a function to the `slideFunction` prop. This is an example of an opposing programming paradigm, _imperative_ programming. The idea of being declarative is to hide away our implementation details such as the slide function. It is the sort of thing that the component should handle, not the consumer of the component.

Let's rewrite this component such that we can do this:

```jsx
<Slider linearOrSmooth="smooth">
	<img src={image1} />
	<img src={image2} />
	<img src={image3} />
<Slider />
```

It is very clear from looking at this that the `img` tags are children of the slider component. It feels obvious because it is read the way HTML is read. And while you could most certainly suggest a better name than `linearOrSmooth`, it does a little better than our example above because it describes exactly what the prop does and hides the implementation details away.

This is what we strive to achieve by being declarative. We will undoubtedly come back to read our code again and again, and more likely than not, other people will also be reading our code. It helps therefore when we can understand what our code is doing by looking at it. This also helps with testing because as Kent C. Dodds puts it best, we want to avoid testing implementation details and instead test what our component does. That becomes a lot easier if we hide our implementation details away.

__Composition__ is a design pattern that encourages smaller encapsulated components that can be used to _compose_ larger, more complex components. 

