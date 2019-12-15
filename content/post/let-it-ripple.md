---
title: "Let It Ripple"
subtitle: "Recreating the material design ripple effect in React"
date: 2019-12-10T22:53:19+06:00
draft: true
bigimg: [{src: "/img/ripples.jpg", desc: "https://unsplash.com/photos/Q5QspluNZmM"}]
share_img: "/img/bicycle.jpg"
author: "Rohan Faiyaz Khan"
tags: [react, ripple, material-design]
---

We have all seen the ripple effect animation which was part of the [material design recommendation](https://material.io/develop/web/components/ripples/). It presents itself as a circle that appears at the point of a click and then enlarges and fades away. As a UI tool, it is a fantastic and familiar way to let the user know that there has been a click interaction.

 {{<video src="/img/ripple.webm">}}

## Rippling in React

__Before anyone complains, yes, this can easily be done using Vanilla JS__. I am just using React cause I am comfortable with it and want to demonstrate how it can be done in React.

__Also if anyone just wants to see a working code example and not go through the explanation behind it, feel free to browse it in this [Code Sandbox](https://codesandbox.io/s/react-material-design-ripple-effect-tsucg).__

There are several ways to achieve this ripple effect in our React components. The easiest would be to use [Material-UI](https://material-ui.com/) which is a popular UI library. However for a small project it doesn't make a lot of sense to pull in a large dependency like Material UI. I figured there had to be a way to this independant of libraries or framework.

Turns out there is because the ripple effect is mostly just CSS! The way most libraries implement it is simply appending a `div` or `span` inside an element, and the CSS does the rest of the magic. The CSS looks something like this.

```html
<button class="parent">
  <div class="ripple-container">
    <span class="ripple"></span>
  </div>
</button>
```
```css
.parent {
  overflow: hidden;
  position: relative;
}

.parent .ripple-container {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
}

.parent .ripple-container span {
  position: absolute;
  top: ...
  right: ...
  height: ...
  width: ...
  transform: scale(0);
  border-radius: 100%;
  opacity: 0.75;
  background-color: #fff;
  animation-name: ripple;
  animation-duration: 850ms;
}

@keyframes ripple {
  to {
    opacity: 0;
    transform: scale(2);
  }
}
```

The `overflow: hidden` property prevents the ripple from _rippling_ out of the container, which is in this case a button. The ripple container is an empty div which we will be populating with our ripples. As for the ripple itself, it will start at 0% size and 75% opacity and using a CSS animation, scale to 200% its size and fade out. We will however need to dynamically provide a few styles. We need to use Javascript to find the positional coordinates, which are based on where the user clicked, and the height and width, which depend on the size of the container.

So here's what our component will need to do.

- Render an array of _ripples_ (`<span>`s) in the container `<div>`
- On mouse down, append a new ripple to the array and calculate the ripple's position and size
- After a delay, clear the ripple array to not clutter up the DOM with old ripples
- Optionally take in the ripple duration and color. We want to be able to customize the ripple's behaviour if needed.

### Let's get started

