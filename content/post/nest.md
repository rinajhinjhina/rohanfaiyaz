---
title: NestJS CRUD with Postgres
subtitle: A robust and scalable web framework for backends
date: 2019-08-08T10:30:04+06:00
author: Rohan Faiyaz Khan
bigimg: [{src: "/img/nest-background.png", desc: "Nest"}]
tags: [nestjs, crud, postgres]
---

[NestJS](https://nestjs.com/) is a fantastic framework for writing robust web backends. According to the docs:

> Nest (or NestJS) is a framework for building efficient, scalable Node.js server-side applications. It uses progressive JavaScript, is built with and fully supports TypeScript (yet still enables developers to code in pure JavaScript) and combines elements of OOP (Object Oriented Programming), FP (Functional Programming), and FRP (Functional Reactive Programming) Under the hood, Nest makes use of robust HTTP Server frameworks like Express (the default) and optionally can be configured to use Fastify as well!

Essentially __Nest__ takes inspiration from enterprise level frameworks such as the .NET Framework while also offering the simplicity and flexibility of building a simple __Node.js__ server with __Express__. Nest is inspired heavily from Angular and seeks to provide similar abstractions to the backend. 

That's all fine and good, but __why should you use Nest__?

Well let me preface this by first saying that if have a preferred backend of choice in another language such as Django, Laravel or .NET, Nest won't offer any new tools or tricks. However if you have been building Node servers before you might have experienced the frustration of setting up the same application over and over again. Or you might have had difficulties maintaining the app as scale grew. _Nest_ offers so many advantages over starting a project from scratch that I believe it will soon become a default for Node CRUD applications and APIs. Over this article I will try my best to show what those advantages are.

First of all there is _TypeScript_ support and an application structure right out of the box. You don't have waste time setting up your application and straight away jump into creating your endpoints. In addition Nest comes with its own command line that lets scaffold the application and add components very quickly. _Nest_ is also very platform agnostic and you can use any database or any frontend with it, For instance, it has TypeORM integration that can easily be set up with an MySQL, Postgres, SQL Server or even MongoDB database.

I will go through how to build a REST API capable of CRUD operations to demonstrate how easy it is to get started with Nest.

## Get started with Nest

We will first install NestJS (if you don't already have it installed) using _NPM_ and set up a new Nest project

```bash
$ npm i -g @nestjs/cli
$ nest new nest-app
```
This will scaffold the application and ask you which package manager you prefer, _npm_ or _yarn_. I used _yarn_ for this example but feel free to use _npm_ if you like. After installing we can test the server.

{{<figure src="/img/installing-nest.png">}}

We can test the installation by using `yarn start`. This will start the app by default on port 3000. We can navigate over to see a "Hello world" message.

## Setting up Postgres

This part will assume you have Postgres installed. If not, install Postgres on OS, for instance you can use the [official installer for Windows](https://www.postgresql.org/download/windows/), [use Homebrew on MacOS](https://gist.github.com/ibraheem4/ce5ccd3e4d7a65589ce84f2a3b7c23a3) or [apt on Ubuntu](https://wiki.postgresql.org/wiki/Apt).

Once we have Postgres installed, we will want to create a database. We can do this using a GUI tool such as [PGAdmin](https://www.pgadmin.org/) or simply using Postgres CLI. I am using Linux so I will use `sudo` to switch to the _postgres_ user, which is the default superuser. We need the super user in order to call the the `createdb` command.

```bash
$ sudo -u postgres psql
postgres=# createdb rohan
postgres=# \q
```
The command `psql` opens the Postgres CLI. `\q` quits the Postgres CLI. We can set a password to our local user so we can securely access  the local database.

```bash
$ psql
rohan=# \password
rohan=# Enter new password:
rohan=# Retype new password
```
## Connecting our app to our Database

Firstly we will install the necessary dependencies. TypeORM is the suggested ORM and is very good for connecting with Postgres.

```bash
yarn add pg typeorm @nestjs/typeorm @nestjsx/crud
```

Next we will create a config file at the root of our project and call it __ormconfig.json__.

```json
{
    "type": "postgres",
    "host": "localhost",
    "port": 5432,
    "username": "rohan",
    "password": "password",
    "database": "rohan",
    "entities": ["dist/**/*.entity.js"],
    "synchronize": true,
    "logging": true
}
```
Nest uses an application structure similar to Angular in the sense that there are controllers, modules and providers (or services) for each component. The root of the project contains an app.module and an app.controller. In order to include our TypeORM module, we must import in our app.module. Doing so makes the module available all over our application without needing to import it anywhere else. In the __app.module.ts__ file inside the src directory, type:

```ts
import { Module } from '@nestjs/common'
import { AppController } from './app.controller'
import { AppService } from './app.service'
import { TypeOrmModule } from '@nestjs/typeorm'
import { PokemonModule } from './pokemon/pokemon.module'

@Module({
	imports: [ TypeOrmModule.forRoot(), PokemonModule],
	controllers: [ AppController ],
	providers: [ AppService ]
})
export class AppModule {}
```
Don't worry about the Pokemon module, we will be creating it soon. You might notice here that the module is what connects the different parts of the component. We will include the __TypeOrmModule__'s _forRoot_ module with empty parameters. By default it will the configuration in our __ormconfig.json__ as its parameters.

## Creating an entity

Next let's create an entity. We will make an entity for pokemon! Make a new directory called __pokemon__ and inside it create a file called __pokemon.entity.ts__. 

```ts

import { Entity, Column, PrimaryGeneratedColumn} from 'typeorm'

@Entity('pokemon')
export class PokemonEntity {
	@PrimaryGeneratedColumn('uuid') id: string

    @Column('varchar', { length: 500, unique: true})
    name: string

    @Column('varchar', { length: 500 })
    type: string

    @Column('numeric')
    pokedex: number
}
```
## Setting up CRUD

Before setting up our CRUD operactions we have to import the entity created in our model. We can generate a model using the Nest CLI.
```bash
nest generate module pokemon
```
This will auto generate a module for us called __pokemon.module.ts__.

```ts
import { Module } from '@nestjs/common'
import { PokemonService } from './pokemon.service'
import { PokemonController } from './pokemon.controller'
import { PokemonEntity } from './pokemon.entity'
import { TypeOrmModule } from '@nestjs/typeorm'

@Module({
	imports: [ TypeOrmModule.forFeature([ PokemonEntity ]) ],
	controllers: [ PokemonController ],
	providers: [ PokemonService ]
})
export class PokemonModule {}
```
Next the service.
```bash
nest generate service pokemon
```
Inside the newly created __pokemon.service.ts__ file we will inject our CRUD service.
```ts
import { Injectable } from '@nestjs/common'
import { InjectRepository } from '@nestjs/typeorm'
import { TypeOrmCrudService } from '@nestjsx/crud-typeorm'
import { PokemonEntity } from './pokemon.entity'

@Injectable()
export class PokemonService extends TypeOrmCrudService<PokemonEntity> {
	constructor (@InjectRepository(PokemonEntity) repo) {
		super(repo)
	}
}
```
Finally in our pokemon controller we will set up our CRUD endpoints using the `@nestjsx/crud` package. This really demonstrates an area where NestJS shines. While it is possible to make all our endpoints manually (using @Get, @Post, @Put, @Patch and @Delete decorators in our controller), the CRUD package allows us to set up all of this effortlessly in a few lines of code.

```ts
import { Controller} from '@nestjs/common'
import { Crud } from '@nestjsx/crud'
import { PokemonService } from './pokemon.service'
import { PokemonEntity } from './pokemon.entity'
@Crud({
	model: {
		type: PokemonEntity
	},
	params: {
		id: {
			field: 'id',
			type: 'uuid',
			primary: true
		}
	}
})
@Controller('pokemon')
export class PokemonController {
	constructor (public service: PokemonService) {}
}
```

ANd with that CRUD automatically sets up the following endpoints:

- `GET` /pokemon Get all
- `GET` /pokemon/:id Get one by id 
- `POST` /pokemon Add one
- `POST` /pokemon/bulk Add many
- `PUT` /pokemon/:id Replace one
- `PATCH` /pokemon/:id Update one

You can test out the new endpoints in Postman or curl.

## That's it!

I don't know about you but when I first learned to make a REST API there was a lot more boilerplate code to figure out. Nest offers just the right amount of high level abstractions to make building APIs a fun process again because you will be focus on what really matters- the endpoints and the entities, Furthermore, NestJS's [official docs](https://docs.nestjs.com/) has comprehensive recipes on integrating with GraphQL (_a post on that coming soon_) and others. If you are a NodeJS developer, NestJS is something very exciting to keep your eye on.