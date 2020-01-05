---
title: "Lessons From My Junior Year in Tech"
date: 2020-01-05T12:17:03+06:00
tags: ["career"]
share_img: "/img/road.jpg"
bigimg: [{ src: "/img/road.jpg", desc: "https://unsplash.com/photos/5hvn-2WW6rY" }]
draft: true
---

About a year ago I was fresh out of university and moved back to Dhaka to start my career as a software developer. I knew very little of what I know today back then, I had no concept of engineering practices in my profession, nothing about how to get a web application up and running, and not a single clue about architechture. All I knew was some Java, Javscript, HTML and CSS, and how to hack together a web page or an Android app.

It took me two months to find a job, one where I had to had to long hours on legacy applications and felt unsatisfied and unfulfilled. 4 months later I burned out and quit, heading back to the drawing board as I tried to piece together what went wrong. Lucky for me though I found my next job very quickly using technology I was quite comfortable with- React, and that too at a remote position at a company with far better work culture.

I am extremely fortunate my junior led me through a well of different experiences. I got to experience a place of undisciplined engineering practices so that I could actually appreciate what the value of disciple in my craft was. Here is a list of lessons I learned that I will hold very dear to my heart.

## Stop stressing about what framework or technology to use

When I was learning about various web frameworks, I was overwhelmed by the sheer amount of choice that I had available to me. What should I learn? React, Angular or Vue? Or how about Svelte? Well okay about backend? Should I learn Python and Django? Or stick with Javascript and Express?

## There is a lot more to the job than just writing code

The first misconception unversity drilled in me is that the difficult part of the job was writing code. It really isn't. These days the barrier to learning code is lower than ever before and practically anyone can buy a course on Udemy for $10 and start learning. I am not saying it is easy, but its _doable_. And anyone who knows how to code can theoretically keep throwing code against a wall until something sticks.

Of course that is not what they pay me for. Building real world applications have two very important restrictions, __limited time and resources__. You only have a set amount of time to deliver a project and to do it right. Writing bad code will not only make this task difficult but impossible to maintain once requirements change and iterations come in (more on this later). This is where every other _soft skill_ comes in. Communication with teammates, project planning, issue tracking, version control and flow, tracer code, writing tests and so on. It is the combination of all these disciplines and more that make one a reliable and productive software engineer.

## Code quality matters, but not as much as you think

When I was starting out my code would be an unmaintainable mess, so I invested some time reading _Clean Code_ by Robert Martin, which was recommended to me by a lot of people. However afterwards the converse happebed, I would stress a lot over coding style, and spend a ridiculous amount of time trying to make my code as clean as possible, painstakingly abstracting every small detail.

Turns out this wasn't the best approach either. Clean code is very important as a means to an end, and the end being good software. But when coupled with the restrictions on time and resources I mentioned earlier, sometimes tradeoffs have to be made. I found it more helpful to not focus on the code itself, but rather the software and making sure it is as reliable and maintainable as possible and letting this goal drive the style of the code. Basically the big picture is always more important than the nitty gritty details.

This became much more apparent when I faced a problem with something that was part of the AWS Amplify SDK. I had to dig into the source files to figure it out and found to my chagrin that the code was far from what Bob Martin would describe as clean. But this was written by Amazon, which hired software engineers that were way better than me. What was going on? Well without any context it might be easy to dismiss it as bad code, but real life scenarios are not ideal, and most likely those engineers had a restriction that forced them to write the code the way they did.
