---
title: "Efficiency in the Editor : How I overcame my fear of Vim"
date: 2019-08-29T17:53:54+06:00
author: Rohan Faiyaz Khan
tags: [vim, beginners, productivity]
bigimg: [{src: "/img/vim.jpg", desc: "Vim"}]
share_img: "/img/vim.jpg"
---

 A colleague at my previous job was once looking at my screen and mentioned that he was impressed with how quickly I was moving around within my code editor. Granted he was not a developer and came from a non-technical background, but his comment surprised me as speed apart from that of typing was not something that I had given much thought before. Navigating through my workspace using a combination of arrow keys, find, go-to-line and the navigation keys such as Home and End were things I had learned simply through practice. 

 But it made sense. We, programmers, tend to spend some time writing code and a lot more time reading code, jumping between different points in our codebase, and tweaking minor things here and there. A significant chunk of our time is spent navigating, referencing and refactoring. Simply typing code faster is not enough to boost my productivity, I had to navigate and traverse my code faster.

## "Hey, did you try Vim?"

For the uninitiated [Vim](https://www.vim.org/) is a CLI based text editor that relies on short and expressive keyboard commands to do pretty much everything. Proponents of Vim argue that it is extremely inefficient to be constantly switching between your keyboard and mouse while typing, and your keyboard should really be able to do everything you need. 

Vim was something recommended by a friend and I had seen several Youtubers use it. I noticed that watching a skilled Vim user at his job looked nothing short of wizardry. It seemed as if they could bend the text editor to their will at the tap of a few keystrokes. Above all it seemed so... __intimidating__. 

_All this editor kung fu that ditches the mouse seems way too complex for my meager self, and besides I rely on waaaay too many things on my VS Code setup. No way I am ditching that._ Or at least, so I thought. 

First thing I learned was VS Code had a fantastic extension that emulates Vim, which let me leverage the full force of my Prettier and ESLint setup while also using Vim. So there went one of my excuses. Second thing I learned was that Vim was not very difficult to get started in. One of my favorite web dev Youtubers [Ben Awad](https://www.youtube.com/user/99baddawg) is an avid Vim advocate and he recommend to start with `vimtutor` and then go cold turkey, i.e. use Vim for everything.  

__Vimtutor__ is a fantastic interactive tutorial that comes out of the box when you install Vim. It takes roughly 30 minutes to complete and taught me all I need to get started with Vim. I wondered why a lot of the Vim tutorials I would find online were for a bit more advanced users till I realized it was beause Vimtutor pretty much covered everything a beginner needs to know, and it did a lot better than a blog post ever could. After completing it, I went the cold turkey route and have been using nothing but Vim (inside VS Code of course) for the past month and here is what I learned from it.

## It is intimidating for a reason

There is a learning curve associated with Vim, there is no doubt about that. For the first few days I was fumbling about blindly, needing to look up references for every little thing. Habits such as using the arrow keys for movement and backspace for correcting mistakes were so ingrained in my muscle meomry that they were really hard to let go of. And I was really slow, way slower than before, which made me seriously question whether this endeavour was even worth it.

However, within a few days I was learning how to be productive in Vim. I had to think less between actions and it was becoming more akin to muscle memory. After a week I had already caught to my old pace and after two weeks I had surpassed it. 

Turns out people were right, there is infact a lot of __efficiency__ to be gained from not dragging your hand to the mouse for every little motion you need to make. However, that is only a small part of what makes Vim so powerful, another major selling point is... 

## Vim is extremely expressive

Vim works using _motions_ and _commands_. _Motions_ annotate where the cursor should move and _commands_ annotate the verb the action will perform. 

The generic motion controls in Vim for left, down, up, right are h, j, k and l respectively, but that is boring and nothing your arrow keys couldn't already do. More interesting motions include __w__ord, __b__ack, and __e__nd which jump to the next word, previous word or end of current word respectively. These motions can also be modified, for instance __3w__ jumps forwards by 3 words and __5b__ jumps backward by 5 words. You can already how this would allow you to quickly move around in your file.

Commands are verbs such as __c__hange, __y__ank, __d__elete and __p__aste. But here's the fun part, the verbs do nothing unless combined with a motion. For instance, __d2w__ will delete the next two words, __yy__ will yank (or copy) the entire line and __cj__ will allow you to change all contents upto the line below. 

This is where the expressiveness of Vim comes into play. Because of the wild number of combinations possible, you can be very creative about how you choose to get your work done. As a result of this, you can very quickly go from thinking about doing an action and doing it, i.e. there is very little lag between thinking and doing. 

Already with this knowledge I was productive and having fun but then I discovered perhaps the best part about Vim.

## Macros for days!

Learning macros is where I realized I was never going to be able to go back to not using Vim. Macros allow you to repeat similar tasks. The most simplest form of this is the __.__ (dot command), which repeats the last change. This is easier to explain with an example. Consider the HTML below.command. 

```html
<div class="image-container">
    <img src="image.jpg>
    <img src="image2.jpg>
    <img src="image3.jpg>
</div>
```

Suppose are using a lazy loading library and need to add a class with the name "lazy" at each image tag. We can do this in Vim by using `/src` (to find the first src attribute), `i` (to insert text) and then type `class="lazy" `. From there on, we can hit n a few times to go to the next instance of src. and hit ` . ` to repeat the command.

{{<video src="/video/vim1.webm" >}}

However we might want to repeat tasks that involve more than one change. Worry not, for custom macros are here to the rescue. Custom macros can be recorded by pressing `q` followed up by the key you want to store the macro in. Pressing q again stops the recording. This is also easier to show rather than tell.

```javascript
const array = [
    item_one,
    item_two,
    item_three,
    item_four,
    item_five
]
```

Suppose I copied this array for another file and the variables are written in snake case. However I have written the rest of my code in camel case and would like to convert these variables to camel case as well (itemOne). To do this, we can go to the top of the file and press `j` to go down a line. Next we start recording the macro with `qe`. To convert to camel case we can use `f_` to move the cursor to the first underscore, and then `x` to delete it. Now our cursor will be on the "o", which we can capitalize by pressing `~`. Finally we move to the start of the line with `0`, and down once with `j` and end the recording with `q`.

To repeat the macro we can just hit `@e` and Vim will repeat the entire sequence of commands. We can also do `4@e` and it will repeat the commands for all four of the remaining lines. Here you can see it in action.

{{<video src="/video/vim2.webm">}}

## Very efficient, right?

Yes, Vim has indeed increased my efficiency significantly to the point where I use it for absolutely everything including writing this blog post. It can significantly help you speed up for workflow and as a result possibly get more work done or have more free time to enjoy. If you are like me and you haven't touched Vim yet cause you found it intimidating or were on the fence about it, I do hope this post can convince to give it a try cause it is certainly a lot of fun.
