---
title: NestJS CRUD with Postgres
subtitle: A robust and scalable web framework for backends
date: 2019-08-08T10:30:04+06:00
author: Rohan Faiyaz Khan
bigimg: [{src: "/img/nest-background.png", desc: "Nest"}]
tags: [nest, crud, postgres]
---

[NestJS](https://nestjs.com/) is a fantastic framework for writing robust web application backends. According to the docs:

> Nest (or NestJS) is a framework for building efficient, scalable Node.js server-side applications. It uses progressive JavaScript, is built with and fully supports TypeScript (yet still enables developers to code in pure JavaScript) and combines elements of OOP (Object Oriented Programming), FP (Functional Programming), and FRP (Functional Reactive Programming) Under the hood, Nest makes use of robust HTTP Server frameworks like Express (the default) and optionally can be configured to use Fastify as well!

Essentially __Nest__ takes inspiration from enterprise level frameworks such as the .NET Framework while also offering the simplicity and flexibility of building a simple __Node.js__ server with __Express__. Nest is inspired heavily from Angular and seeks to provide similar abstractions to the backend. 

That's all fine and good, but _why should you use Nest_?

Well let me preface this by first saying that if have a preferred backend of choice in another language such as Django, Laravel or .NET, Nest won't offer any new tools or tricks. However if you have been building Node servers before you might have experienced the frustration of setting up the same application over and over again. Or you might have had difficulties maintaining the app as scale grew. _Nest_ offers so many advantages over starting a project from scratch that I believe it will soon become a default for Node CRUD applications and APIs. Over this article I will try my best to show what those advantages are.

First of all there is _TypeScript_ support and an application structure right out of the box. You don't have waste time setting up your application and straight away jump into creating your endpoints. In addition Nest comes with its own command line that lets scaffold the application and add components very quickly. _Nest_ is also very platform agnostic and you can use any database or any frontend with it, For instance, it has TypeORM integration that can easily be set up with an MySQL, Postgres, SQL Server or even MongoDB database.

I will go through how to build a REST API capable of CRUD operations to demonstrate how easy it is to get started with Nest.

## Get started with Nest

