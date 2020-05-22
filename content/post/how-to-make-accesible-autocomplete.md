---
title: "How to Make an Accesible Auto Suggest with vanilla Javascript"
author: "Rohan Faiyaz Khan"
share_img: "/img/google-search.png"
date: 2020-05-16T20:52:30+06:00
tags: ["autosuggest", "combobox", "a11y", "javascript"]
draft: false
---

![Example of auto suggest in Google search](/img/google-search.png)

## What is an Auto Suggest?

_Autosuggest_, also referred to semantically as a _Combobox_, is a web component we are all familiar with. It is comprised of an input where a user can type, and a dropdown menu with suggestions that the user can select. Depending on the use case, there may be some extra caveats. Some components will autofill the user's response based on the suggests, some will require that the user select something, some will fire a network request and so on.

A component such as this is pervasive in the modern web, search boxes, form inputs, and so many things utilize a variation of this component. It is a wonder that there isn't a standard HTML element to handle them.

### The datalist way

Well to be fair, there is one. The easiest way to go about making your own _autosuggest_ would be to use HTML5's `<datalist>` element which is now widely supported across all browsers. [The example from MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/datalist) shows how simple it is.

```html
<label for="ice-cream-choice">Choose a flavor:</label>
<input list="ice-cream-flavors" id="ice-cream-choice" name="ice-cream-choice" />

<datalist id="ice-cream-flavors">
  <option value="Chocolate"> </option>
  <option value="Coconut"> </option>
  <option value="Mint"> </option>
  <option value="Strawberry"> </option>
  <option value="Vanilla"> </option>
</datalist>
```

However datalist comes with its own set of problems. The datalist behaviour isn't consistent across every browser and you are limited to a single line of text for displaying the options. Focus management is inconsistent and any sort of custom behaviour you want is going to cause more pain than you might expect. Feel free to try this out but the results might not be what you want.

If this is all you need then great. If not, let's look at a custom albeit harder way.

### The combobox way

So if `<datalist>` doesn't work you will have to devise your own solution using a combination of an input and a list that can be shown and hidden using CSS. Seems simple right? Well there is still a problem we need to consider and that is **accessibility**. When we use a `<select>` element the browser implements accessibility features out of the box, the user than scroll up and down using the arrow keys, and use keyboard controls to open and close the dropdown. Assistive software for users with disabilities know how to announce that the element has a dropdown, and whether or not the dropdown is open.

Using a simple `<input type="text">` tag followed by a `<ul>` list will not give us these benefits out of the box, and so we need to code them in ourselves. The WAI-ARIA widget for an autosuggest is called a **combobox**].

You could perhaps use a library to implement this and that could work but a library might not have every feature you want or have features you don't want. Some of them are also not completely accessible. Even if you are using a library that you like, it is good to learn how it works on the inside.

Before we start I would like to add quickly that I am still new to writing accessible web components and due to the huge variance in screenreader behaviour there may be things that I missed. If you are experienced at accessibility please feel free to suggest improvements.

### Accessibility Requirements

Using the [official WAI-ARIA guidelines as reference](https://www.w3.org/TR/wai-aria-practices/#combobox), we can identify some features that our component needs to have to ensure it is accessible. Ignoring some optional cases or ones that are not applicable to our use case, we can list the requirements as follows.

__1. Aria roles, states and properties__
  - The container needs to have `role="combobox"`
  - The input field inside the combobox needs to have `role="textbox"`
  - Combobox element contains or owns an element that has role listbox, tree, grid, or dialog. For our use case, we will be using a listbox
  - The textbox element has `aria-controls` set to a value that refers to the combobox popup element.
  - When the combobox popup is not visible, the element with role combobox has `aria-expanded="false"`. When the popup element is visible, `aria-expanded="true"`.
  - When a descendant of a listbox, grid, or tree popup is focused, DOM focus remains on the textbox and the textbox has `aria-activedescendant` set to a value that refers to the focused element within the popup.
  - When a suggested value is visually indicated as the currently selected value, the option containing that value has `aria-selected` set to true.
  - If the combobox has a visible label, the element with role combobox has `aria-labelledby` set to a value that refers to the labelling element.

__2. Keyboard interaction__

  - When focus is on the textbox:
    - `Down Arrow`: If the popup is available, moves focus into the popup
    - `Escape`: Dismisses the popup if it is visible

  - When focus is on the listbox:
    - `Enter`: Accepts the focused option in the listbox by closing the popup and placing the accepted value in the textbox with the input cursor at the end of the value.
    - `Escape`: Closes the popup and returns focus to the textbox.
    - `Right Arrow`: Returns focus to the textbox without closing the popup and moves the input cursor one character to the right. If the input cursor is on the right-most character, the cursor does not move.
    - `Left Arrow`: Returns focus to the textbox without closing the popup and moves the input cursor one character to the left. If the input cursor is on the left-most character, the cursor does not move.
    - Any printable character: Returns the focus to the textbox without closing the popup and types the character.
    - `Down Arrow`: Moves focus to and selects the next option. If focus is on the last option, either returns focus to the textbox or does nothing.
    - `Up Arrow`: Moves focus to and selects the previous option. If focus is on the first option, either returns focus to the textbox or does nothing.
    -  Right Arrow: Returns focus to the textbox without closing the popup and moves the input cursor one character to the right. If the input cursor is on the right-most character, the cursor does not move.
    - Left Arrow: Returns focus to the textbox without closing the popup and moves the input cursor one character to the left. If the input cursor is on the left-most character, the cursor does not move.
    - Any printable character: Returns the focus to the textbox without closing the popup and types the character.

### Implementation

Now that we have our requirements out of the way let us implement this. As I do with all my blog posts, I have implemented this in [Codesandbox which you can view here](https://codesandbox.io/s/vanillajs-accessible-autocomplete-b0v5n) if you are the type to dive straight into the code.

#### Markup and styles

First of all let's set the markup. Of course the specifics of the markup will depend entirely on you as long as fulfill the accessibility requirements listed above. Here is my implementation. I am using a container `div` as my `combobox` container which contains an `input` that serves the role of `textbox` and an empty `ul` with a role of `listbox`. There is also a button containing an svg arrow for toggling the list.

```html
<label for="autocomplete-input" id="autocomplete-label">'
   Type a name of your favorite color
</label>

<!-- Combobox container -->
<div
  class="autocomplete__container"
  role="combobox"
  aria-labelledby="autocomplete-label"
>
  <input
    role="textbox"
    aria-expanded="false"
    aria-controls="autocomplete-results"
    id="autocomplete-input"
    class="autocomplete__input"
  />
  <!-- Arrow for toggling the dropdown -->
  <button aria-label="toggle dropdown" class="autocomplete__dropdown-arrow">
    <svg width="10" height="5" viewBox="0 0 10 5" fill-rule="evenodd">
      <title>Open drop down</title>
      <path d="M10 0L5 5 0 0z"></path>
    </svg>
  </button>
  <ul
    role="listbox"
    id="autocomplete-results"
    class="autocomplete__results"
  >
     <!-- This is where we will be inserting our list items -->
  </ul>
</div>
```

The children of the listbox which we will dynamically enter will look like this. The `tabindex="0"` enables this element to be focused.

```html
<li class="autocomplete-item" id="autocomplete-item-index" role="listitem" tabindex="0">
   <!-- content -->
</li>
```

Here are the styles that make this work. Notice that I use the `visible` class on the list and the `expanded` class on the dropdown as state indicators.

```css
.autocomplete__container {
  position: relative;
  margin-top: "0.8rem";
  width: 100%;
  max-width: 350px;
}

.autocomplete__results.visible {
  visibility: visible;
}

.autocomplete__input {
  display: block;
  width: 100%;
  padding: 0.4rem 0rem 0.4rem 1rem;
  border: 2px solid hsl(212, 10%, 80%);
  border-radius: 5px;
}

.autocomplete__input:focus {
  border-color: hsl(221, 61%, 40%);
}

.autocomplete__dropdown-arrow {
  position: absolute;
  right: 0;
  top: 0;
  background: transparent;
  border: none;
  cursor: pointer;
  height: 100%;
  transition: transform 0.2s linear;
}

.autocomplete__dropdown-arrow.expanded {
  transform: rotate(-180deg);
}

.autocomplete__results {
  visibility: hidden;
  position: absolute;
  top: 100%;
  margin-top: 0;
  width: 100%;
  overflow-y: auto;
  border: 1px solid #999;
  padding: 0;
  max-height: 200px;
}

.autocomplete__results > li {
  list-style: none;
  padding: 0.4rem 1rem;
  cursor: pointer;
}

.autocomplete__results > li:hover {
  background: hsl(212, 10%, 60%);
}

.autocomplete__results > li:focus {
  background: hsl(212, 10%, 70%);
}
```

#### Toggling the listbox

Getting started with the javascript, let's first handle opening and closing of the listbox. There are several triggers for this such as clicking on the input, focusing on the input and pressing the down arrow, and clicking the toggle dropdown arrow. There are also several triggers for closing, clicking outside the listbox, pressing the escape key while the input is focused and selecting an option in the listbox. It is best if we encapsulate the logic for opening and closing so we can reuse it.

```js
// Extracting the relevant DOM nodes
const input = document.getElementById("autocomplete-input");
const resultsList = document.getElementById("autocomplete-results");
const dropdownArrow = document.querySelector(".autocomplete__dropdown-arrow");
const comboBox = document.querySelector(".autocomplete__container");

// Boolean used for signalling
let isDropDownOpen = false;

// Signals which list item is focused, useful for updown keyboard navigation
let currentListItemFocused = -1;

function openDropdown(){
  isDropDownOpen = true;
  resultsList.classList.add("visible");
  dropdownArrow.classList.add("expanded");
  comboBox.setAttribute("aria-expanded", "true");
}

function closeDropdown() {
  isDropDownOpen = false;
  resultsList.classList.remove("visible");
  dropdownArrow.classList.remove("expanded");
  comboBox.setAttribute("aria-expanded", "false");
  input.setAttribute("aria-activedescendant", "");
}
```

#### Populating the list with data

For the sake of this example I will be populating my list with static data. This can easily be modified to take in data from an API if you wish so.

```js

const colors = [
  "Red",
  "Orange",
  "Yellow",
  "Green",
  "Blue",
  "Cyan",
  "Violet",
  "Black",
  "White"
];


// Take an input array of string values and insert them into the list
function setResults(results) {
  if (Array.isArray(results) && results.length > 0) {

    // Transform array of strings to a list of HTML ul elements
    const innerListItems = results
      .map(
        (item, index) =>
          `<li class="autocomplete-item" 
             id="autocomplete-item-${index}" 
             role="listitem" 
             tabindex="0"
            >
                ${item}
           </li>`
      )
      .join("");

    resultsList.innerHTML = innerListItems;
    
    // Reset focus when list changes
    currentListItemFocused = -1;
  }
}

setResults(colors);
```

#### Handling focusing and selecting a list item

Focusing and selecting is a simple process but you do need to ensure the appropriate ARIA properties are set as per our requirements. 

Note that for certain use cases you may want to disable the input on selection as well and add a button (or Backspace key) to clear it.

```js
function focusListItem(listItemNode) {
  const id = listItemNode.id;
  input.setAttribute("aria-activedescendant", id);
  listItemNode.focus();
}

function selectValue(listItemNode) {
  const value = listItemNode.innerText;
  input.value = value;
  listItemNode.setAttribute("aria-selected", "true");
  input.removeAttribute("aria-activedescendant");
  input.focus();
  closeDropdown();
}
```

#### Adding click handlers

We need click handlers for three things:

- Clicking the input opens the listbox
- Clicking outside closes it
- Clicking the arrow toggles the listbox
- Clickung an option from the list selects it

```js
input.addEventListener("click", openDropdown);

dropdownArrow.addEventListener("click", event => {
  event.preventDefault();
  if (!isDropDownOpen) {
    openDropdown();
  } else {
    closeDropdown();
  }
});

document.addEventListener("click", () => {
  const dropdownClicked = [
    input,
    dropdownArrow,
    ...resultsList.childNodes
  ].includes(event.target);

  if (!dropdownClicked) {
    closeDropdown();
  }
);

resultsList.addEventListener("click", event => {
  if ([...resultsList.childNodes].includes(event.target)) {
    selectValue(event.target);
  }
});
```

#### Adding keyboard controls

Keyboard controls are a bit complicated as we need to make sure our list is completely navigable by keyboard and follows the conditions in the accessibility requirements. 

One thing that might trip up people is scrolling. If you have a long list you will want to allow scrolling, but pressing up and down in a scrollable view will cause the view to scroll. As we want to use up and down arrow keys for navigation we need to prevent this with an `event.preventDefault()`. Then simply focusing each element as we navigate to it will cause said element to scroll into view.

```js

function handleKeyboardEvents(event) {
  const listItems = resultsList.childNodes;
  let itemToFocus = null;

  switch (event.key) {
    case "ArrowDown":
      event.preventDefault();
      if (currentListItemFocused < listItems.length - 1) {
        if (!isDropDownOpen) {
          openDropdown();
        }
        currentListItemFocused = currentListItemFocused + 1;
        itemToFocus = listItems.item(currentListItemFocused);
        focusListItem(itemToFocus);
      }
      break;
    case "ArrowUp":
      event.preventDefault();
      if (currentListItemFocused > 0) {
        currentListItemFocused = currentListItemFocused - 1;
        itemToFocus = listItems.item(currentListItemFocused);
        focusListItem(itemToFocus);
      }
      break;
    case "Home":
      if (currentListItemFocused > 0) {
        currentListItemFocused = 0;
        itemToFocus = listItems.item(currentListItemFocused);
        focusListItem(itemToFocus);
      }
      break;
    case "End":
      if (currentListItemFocused < listItems.length - 1) {
        currentListItemFocused = listItems.length - 1;
        itemToFocus = listItems.item(currentListItemFocused);
        focusListItem(itemToFocus);
      }
      break;
    case "Enter":
      event.preventDefault();
      if (!isDropDownOpen) {
        openDropdown();
      } else {
        if (listItems[currentListItemFocused].innerText) {
          selectValue(listItems[currentListItemFocused]);
        }
      }
      break;
    case "Escape":
      if (isDropDownOpen) {
        closeDropdown();
      }
      break;
    default:
       if (event.target !== input) {

        // Check if list is focused and user presses an alphanumeric key, or left or right
        if (/([a-zA-Z0-9_]|ArrowLeft|ArrowRight)/.test(event.key)) {

          // Focus on the input instead
          input.focus();
        }
      }     
      break;
  }
}

input.addEventListener("keydown", handleKeyboardEvents);
resultsList.addEventListener("keydown", handleKeyboardEvents);

```

Notice the default case which fulfills the last three conditions for keyboard controls in the accessibility requirements. If the user presses left, right or any printable key we need to return focus to the input field. We can use a simple regular expression to test for this and focus the input if needed. Simply focusing on the input will cause the characters to be printed on the input instead.

#### Deboucing input (optional)

We have covered almost everything except filtering the list results when we type. Before we do this though I want to briefly cover debouncing which you will certainly want if you are either:
- Sending network requests with each input
- Performing an expensive filter function 

What a debouncer does is wait until your input has stopped changing for a set timeout before launching the callback, thus reducing unnecessary calls to it. If you don't need a debounce feel free to skip this.

```js
let bounce = undefined;
function debounce(callback) {
  clearTimeout(bounce);
  bounce = setTimeout(() => {
    callback();
  }, [500]);
}
```

#### Filtering input

Finally once all our bases have been covered we can write our filter function that gets called when the user types. This function will vary completely based on your requirements. I will demonstrate a very simple example using a very simple regular expression that checks if the word starts with the input string entered by the user.

```js
function filter(value) {
  if (value) {
    const regexToFilterBy = new RegExp(`^${value}.*`, "gi");
    filteredResults = colors.filter(color => regexToFilterBy.test(color));
  } else {
    filteredResults = [...colors];
  }
  setResults(filteredResults);
}

input.addEventListener("input", event => {
  const value = event.target.value;

  debounce(() => {
    filter(value);
    if (!isDropDownOpen) {
      openDropdown();
    }
  });
});
```

### Conclusion 

With that our implementation should be off and working. You can test it in the aforementioned [Codesandbox which you can view here](https://codesandbox.io/s/vanillajs-accessible-autocomplete-b0v5n) before implementing yourself.

I should add however that while I have tried my best to adhere to the official WAI-ARIA guidelines, screen-readers widely vary in terms of what they announce to the user. As I am still new to learning about accessibility it is entirely possible that I have missed something. Please feel free to suggest improvements to this code either via a comment or a pull request to the [repository](https://github.com/rohanfaiyazkhan/vanillajs-accessible-autocomplete).
