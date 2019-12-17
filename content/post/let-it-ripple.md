---
title: "Let It Ripple"
subtitle: "Recreating the material design ripple effect in React"
date: 2019-12-10T22:53:19+06:00
bigimg: [{src: "/img/ripples.jpg", desc: "https://unsplash.com/photos/Q5QspluNZmM"}]
share_img: "/img/ripples.jpg"
author: "Rohan Faiyaz Khan"
tags: [react, ripple, material-design]
---

We have all seen the ripple effect animation which was part of the [material design recommendation](https://material.io/develop/web/components/ripples/). It presents itself as a circle that appears at the point of a click and then enlarges and fades away. As a UI tool, it is a fantastic and familiar way to let the user know that there has been a click interaction.

 {{<video src="/video/ripple.webm">}}

## Rippling in React

While the ripple effect is perfectly doable in Vanilla JS, I wanted a way to integrate it with my React components. The easiest way would be to use [Material-UI](https://material-ui.com/) which is a popular UI library. This is a very good idea in general if you want a solid UI library that generates UI out of the box. However for a small project it makes little sense to learn to work with a large library just to achieve one effect. There had to be a way to do with a UI library.

I looked through a lot of projects implementing this over Codepen and Codesandbox and compiled some of the best methods. The ripple effect is possible on any web framework because it is achieved through a clever bit of CSS.

__For advanced readers who want to go straight to the code and skip the explanation behind it, feel free to browse it in this [Code Sandbox](https://codesandbox.io/s/react-material-design-ripple-effect-kn1tr).__

This is my implementation of the CSS for this effect.


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

The `overflow: hidden` property prevents the ripple from _rippling_ out of the container, which is in this case a button. The ripple is a circie (`border-radius: 100%`) which will start at 0% size and 75% opacity and scale to 200% its size as it fades out. 

We will however need to dynamically provide a few styles using Javascript. We need to find the positional coordinates i.e. `top` and `left`, which are based on where the user clicked, and the actual `height` and `width`, which depend on the size of the container.

So here's what our component will need to do.

- Render an array of _ripples_ (`span`s) in the container `<div>`
- On mouse down, append a new ripple to the array and calculate the ripple's position and size
- After a delay, clear the ripple array to not clutter up the DOM with old ripples
- Optionally take in the ripple duration and color. We want to be able to customize the ripple's behaviour if needed.

### Let's get started

I am using `styled-components` for my styles as I am comfortable with it but feel free to use whatever styling option you prefer. __The first thing we will do is include the above CSS in our components__.

```jsx
import React from 'react'
import styled from 'styled-components'

const RippleContainer = styled.div`
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;

  span {
    transform: scale(0);
    border-radius: 100%;
    position: absolute;
    opacity: 0.75;
    background-color: ${props => props.color};
    animation-name: ripple;
    animation-duration: ${props => props.duration}ms;
  }

  @keyframes ripple {
    to {
      opacity: 0;
      transform: scale(2);
    }
  }
`;
```

Notice that I left the `background-color` and `animation-duration` to be fetched from props. This is so that we can dynamically set these values later in our props. Let's define those now:

```jsx
import React from 'react'
import styled from 'styled-components'
import PropTypes from 'prop-types'

...

const Ripple = ({ duration = 850, color = "#fff" }) => {

  ...

}

Ripple.propTypes = {
  duration: PropTypes.number,
  color: PropTypes.string
}

export default Ripple
```

Next up we want to __define an array for our ripples and create a function for adding ripples__. Each element of the array will be an object with `x`, `y` and `size` properties, which are information needed to style the ripple. In order to calculate those values, we will fetch them from a `mousedown` event.

```javascript

const Ripple = ({ duration = 850, color = "#fff" }) => {
  const [rippleArray, setRippleArray] = useState([]);
  
  const addRipple = (event) => {

    const rippleContainer = event.currentTarget.getBoundingClientRect();
    const size = rippleContainer.width > rippleContainer.height
                  ? rippleContainer.width
                  : rippleContainer.height;

    const x = event.pageX - rippleContainer.x - rippleContainer.width / 2;
    const y = event.pageY - rippleContainer.y - rippleContainer.width / 2;
    const newRippleArray = {
      x,
      y,
      size
    };

    setRippleArray(newRippleArray);
  }

```

The above code uses a bit of the Browser DOM API. `getBoundClientRect()` allows us to get the longest edge of the container, and the `x` and `y` coordinates relative to the document.  This along with `MouseEvent.pageX` and `MouseEvent.pageY` allows us to calculate the `x` and `y` coordinates of the mouse relative to the container. If you want to learn more about how these work, there are much more detailed explanations for [getBoundClientRect](https://developer.mozilla.org/en-US/docs/Web/API/Element/getBoundingClientRect), [MouseEvent.pageX](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/pageX) and [MouseEvent.pageY](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/pageY) at the wonderful MDN Web Docs.

This allows us to create our ripple container and ripples.

```jsx
return (
    <RippleContainer duration={duration} color={color} onMouseDown={addRipple}>
      {
        rippleArray.length > 0 &&
        rippleArray.map((ripple, index) => {
          return (
            <span
              key={"ripple_" + index}
              style={{
                top: ripple.y,
                left: ripple.x,
                width: ripple.size,
                height: ripple.size
              }}
            />
          );
        })}
    </RippleContainer>
  );
```

`RippleContainer` is our styled component that takes in the duration and color as `props` along with our newly created `addRipple` as a `onMouseDown` event handler. Inside it we will map over all our ripples and assign our calculated parameters to their corresponding `top`, `left`, `width` and `height` styles.

With this we are _done_ adding a ripple effect! However, there is one more small thing we will need to do with this component and that is __clean the ripples after they are done animating__. This is to prevent stale elements from cluttering up the DOM.

We can do this by implementing a debouncer inside a custom effect hook. I will opt for `useLayoutEffect` over `useEffect` for this. While the differences between the two merit an entire blog post of its own, it is suffice to know that `useEffect` fires after render and repaint while `useLayoutEffect`fires after render but before repaint. This is important here as we are doing something that has immediate impact on the DOM. You can read more about this [here](https://kentcdodds.com/blog/useeffect-vs-uselayouteffect).

Below is our custom hook's implementation and usage where we pass a callback to clear the ripple array. We use a setTimeout that we can reset in order to create a simple _debouncer_. Essentially everytime we create a new ripple, the timer will reset. Notice that the timeout duration is much bigger than our ripple duration.

```jsx
import React, { useState, useLayoutEffect } from "react";

...

const useDebouncedRippleCleanUp = (rippleCount, duration, cleanUpFunction) => {
  useLayoutEffect(() => {
    let bounce = null;
    if (rippleCount > 0) {
      clearTimeout(bounce);

      bounce = setTimeout(() => {
        cleanUpFunction();
        clearTimeout(bounce);
      }, duration * 4);
    }

    return clearTimeout(bounce);
  }, [rippleCount, duration, cleanUpFunction]);
};

const Ripple = ({ duration = 850, color = "#fff" }) => {
  const [rippleArray, setRippleArray] = useState([]);

  useDebouncedRippleCleanUp(rippleArray.length, duration, () => {
    setRippleArray([]);
  });

  ...
```

Now we are done with our Ripple component. __Let's build a button to implent this__.

```jsx
import React from "react";
import Ripple from "./Ripple";
import styled from "styled-components";

const Button = styled.button`
  overflow: hidden;
  position: relative;
  cursor: pointer;
  background: tomato;
  padding: 5px 30px;
  color: #fff;
  font-size: 20px;
  border-radius: 20px;
  border: 1px solid #fff;
  text-align: center;
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.4);
`;

function App() {
  return (
    <div className="App">
      <Button>
        Let it rip!
        <Ripple />
      </Button>
      <Button>
        Its now yellow!
        <Ripple color="yellow" />
      </Button>
      <Button>
        Its now slowwwww
        <Ripple duration={3000} />
      </Button>
    </div>
  );
}

```

## And that's it

 {{<video src="/video/multiple-ripples.webm">}}

We now have ripples in all shades and speeds! Better yet our ripple component can reused in pretty much any container as long as they have `overflow: hidden` and `position: relative` in their styles. Perhaps to remove this dependency, you could improve on my component by creating another button that already has these styles applied. Feel free to have fun and play around with this!
