---
title: Building your first Hugo site
subtitle: A blazing fast static site generator!
date: 2019-08-03
tags: [hugo, static]
bigimg: [{src: "/img/hugo.png", desc: "Hugo"}]
author: Rohan Faiyaz Khan
---

If you are unfamiliar with the concept of static site generators, they allow you to take your content (either as markdown or as input from a CMS) and output a static version of your website.

There are several advantages of static sites. 

- They are extremely fast because there is no need to dynamically fetch content. 
- There is next to no cost of hosting them as they can be served up for free (for instance this site is being hosted on [Netlify](https://www.netlify.com/))
- They are inherently more secure

Need any more reasons? Well how about this, it is possible to create something that is more than just a simple website. Using custom APIs (or a variety of available 3rd party ones) you can make a full fledged web app! This stack is being referred to these days as the JAMStack (JAvascript, Markup and API) and is a fantastic option for the modern web. For instance, once could have a static site and use Auth0 for authentication, Algolia for search, Cloudinary for images and Shopify for payments. The possiblities are endless so I will leave all the delicious JAM goodness for another post. This post will just cover how to get up and start with a simple static website.

While you can use any static site generator of choice such as Jekyll or Middleman if you prefer Ruby, Next.js or Gatsby if you prefer React, or Nust.js or Vue Press if you prefer Vue, this post will focus on [Hugo](https://gohugo.io), which is one of the more popular ones to choose from with over 36k stars on [Github](https://github.com/gohugoio/hugo). I personally like Hugo because it very easy to use, is very fast to build thanks to it being built in Go and it has a large community providing pre-built themes.

## Getting set up

First make sure to install Hugo. There are multiple ways to do this depending on your current platform so use the official [official docs](https://gohugo.io/getting-started/installing/) for reference.

Next setup a hugo project using the command line.

```bash
hugo new site hugo-starter
```
This sets up the following file structure.

```bash
├── archetypes
│   └── default.md
├── config.toml
├── content
├── layouts
│   └── partials
│       ├── footer_custom.html
│       └── head_custom.html
├── resources
│   └── _gen
│       ├── assets
│       └── images
├── static
└── themes
```

## Installing a theme

Next up we will be setting up a theme. Themes are the cornerstone of hugo sites cause they define how your content is to be displayed. It is possible to create a custom theme but for the sake of quickly setting things up we will choose one of the many [community built themes](https://themes.gohugo.io/) available to us.

For this example we will be using [Cocoa-Eh](https://themes.gohugo.io/theme/cocoa-eh-hugo-theme/). To install a theme we need to copy the theme into our __themes__ directory. We can do this manually or via git using the commands:

```bash
git init
git clone https://github.com/mtn/cocoa-eh-hugo-theme.git themes/cocoa-eh
cp -r ./themes/cocoa-eh/exampleSite/* ./
```
After cloning we set up a few sample pages using the contents of the theme's example site folder. One last thing we will have to do is set the theme in the __config.toml__ file by setting the following line:

```toml
theme="cocoa-eh"
```
Finally we just run `hugo server` in development mode which builds our app and sets up a local web server where we can preview our site. Hugo server by default uses live reload so any changes will almost instantly be updated (yes, Hugo builds that fast).

```bash
hugo server -D
```
And that's it! We can view our site by navigating to [localhost:1313](http://localhost:1313).

{{<figure src="/img/hugo-starter.png">}}

## Customizing you content

So obviously we want to customize our content. Let's start with the meta-data which is contained in our __config.toml__. Set the following values to your name and whatever you want the title to be. You can also use a custom image as your logo by pasting it into the __static/img__ directory.

```toml
author = "Rohan Faiyaz Khan"
title = "My fancy new website!"

logofile="/img/new-logo.png
```

Hugo like other static site generators uses _markdown_ as a source of content. It is also possible, and quite simple, to set this up using a CMS such as Wordpress, Forestry, Contentful, Netlify CMS or Sanity. Using a CMS allows to use a GUI to edit content and is ideal if you are building a website for somebody. I will cover connecting to a CMS in a separate but for now markdown will do just fine. If you have never used markdown before, [here](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet) is a handy guide on what you can do in markdown.

In order to edit our welcome message we will change our _home.md_ file inside our _content_ directory.

```markdown
+++
title = "Home"
+++

Hello there. Welcome to my site where I talk about __cat facts__. Feel free to purr through the posts below.
```

The top portion is what is referred to in static site generators as frontmatter. This section contains meta data about your content such as title, author and date. `+++` indicates the frontmatter is in _toml_ format. Another commonly seen case is `---` which indicates frontmatter in _yaml_ format. Next we can make a new post either by pasting it into our _content/blog_ directory or just by using the `hugo new` command.

```bash
hugo new blog/catnip.md
```
We can edit our new post about catnip however we wish!

```markdown
+++
title= "Catnip"
date= 2019-08-04T11:37:13+06:00
author= "Rohan Faiyaz Khan"
+++

Ever wonder why catnip lulls felines into a trance? The herb contains several chemical compounds, including one called nepetalactone, which a cat detects with receptors in its nose and mouth. The compounds trigger the typical odd behaviors you associate with the wacky kitty weed, including sniffing, head shaking, head rubbing, and rolling around on the ground.
```
And that's it! We can see our edits in the live preview.

{{< gallery >}}
  {{< figure link="/img/hugo-starter-2.png" >}}
  {{< figure link="/img/hugo-starter-3.png" >}}
{{< /gallery >}}

## That's all?

Yeah! That's all you need to get your site set up. You can play around however you please. And the best part is we have only scratch the surface of what Hugo is capable of! In future posts we will be covering templates, custom themes and more!



