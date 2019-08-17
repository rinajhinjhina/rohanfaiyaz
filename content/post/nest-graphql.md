---
title: "NestJS GraphQL API with TypeORM and Postgres"
date: 2019-08-11T11:25:19+06:00
subtitle: Reap the rewards of GraphQL in a NestJS app
author: Rohan Faiyaz Khan
tags: [nestjs, typeorm, grahql, postgres]
---

{{<figure src="/img/nestjs-graphql.png" thumb="-thumb">}}

In a previous [post](https://https://rohanfaiyaz.com/post/nest.html), I covered how to make a __NestJS__ REST API using TypeORM and Postgres. While it is a relatively simple process to set up a REST API in NestJS, we still are left with the limitations of a REST API. REST depends on our endpoints being clearly defined and this is usually done using the frontend as a guide, cause obviously we want to provide the exact data the frontend needs.

But this is a problem. What if the frontend requirements change and we need an extra data field that is only available through a separate endpoint? Now we have to make two HTTP requests and we will have extra data which is wasteful for mobile internet usage. The other solution is to create a new endpoint on the backend, which is expensive.

Essentially, a REST api is too _low level_. We need something that offers a higher level of abstraction and that is __GraphQL__.

__GraphQL__ is a query language specification created originally by Facebook as a solution to the problems described above. GraphQL offers a way for frontends to describe the exact data needed and the shape of the data. The data is defined on the backend as a GraphQL schema. A GraphQL request (known as a _Query_) looks like this:

```gpl
{
    pokemon(id: 4) {
        name,
        types
    }
}
```

This returns a JSON response that looks like this:

```json
{
    "data": {
        "pokemon": {
            "name": "Charmander",
            "types": ["Fire"]
        }
    }
}
```

You might notice that the data sent back follows the exact shape of the query. What about adding data though? We do that using GraphQL _Mutations_ which look like this:

```gpl
mutation{
    addPokemon(name: "Scorbunny", types: "Fire") {
        id,
        name,
        types
    }
}
```

This will give a response that looks something like this:

```json
{
    "data": {
        "pokemon": {
            "id": 912,
            "name": "Scorbunny",
            "types": ["Fire"]
        }
    }
}
```
Of course there is a lot more to GraphQL than just that. If you want a more in depth look into what GraphQL is capable, there is no better place to start than the [official documentation](https://graphql.org/learn/). The rest of this post will focus on creating a GraphQL API on the server side using NestJS and TypeORM. 

## Setting up the project

We will first install __NestJS__ (if you don't already have it installed) using _NPM_ and use the Nest CLI to scaffold a new Nest project 

```bash
$ npm i -g @nestjs/cli 
$ nest new nest-app 
```

The database I will choose to integrate with is __Postgres__ although you can use any database of your choice. We will be using __TypeORM__ as our _ORM (Object Relation Mapping)_ which essentially removes the burden of writing raw SQL, as our ORM will handle communication with the database. 

If you do not have Postgres installed, follow the steps to install Postgres on your OS. For instance you can use the [official installer for Windows](https://www.postgresql.org/download/windows/), [use Homebrew on MacOS](https://gist.github.com/ibraheem4/ce5ccd3e4d7a65589ce84f2a3b7c23a3) or [apt on Ubuntu](https://wiki.postgresql.org/wiki/Apt). Once postgres is installed, we will create a new database using the Postgres CLI. On Linux and MacOS the steps will be as follows:

```bash
$ sudo -u postgres psql
postgres=# createdb rohan
postgres=# \q
$ psql
rohan=# \password
rohan=# Enter new password:
rohan=# Retype new password:
```

## Setting up dependencies and config

Firstly we will install the necessary dependencies. TypeORM is the suggested ORM and is very good for connecting with Postgres. NestJS also has its own way of configuring GraphQL which uses _apollo-server_ under the hood.

```bash
yarn add pg typeorm @nestjs/typeorm @nestjs/graphql apollo-server-express graphql-tools type-graphql graphql
```

Next we will create a config file at the root of our project and call it __ormconfig.json__. This file is automatically pulled in by TypeORM.

```json
// ormconfig.json
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
In NestJS each component has its own module where we encapsulate everything associated to the module, including providers, controllers and other modules. The root module file is __app.module.ts__, and it is here that we will specify external modules our entire app will have access to, which are GraphQL and TypeORM. We will use it to set up a schema for a _pokemon_ componenent which we will soon define. 

```ts
// src/app.module.ts

import { Module } from '@nestjs/common'
import { AppController } from './app.controller'
import { TypeOrmModule } from '@nestjs/typeorm'
import { AppService } from './app.service'
import { GraphQLModule } from '@nestjs/graphql'
import { PokemonModule } from './pokemon/pokemon.module'
@Module({
	imports: [
		TypeOrmModule.forRoot(),
		GraphQLModule.forRoot({
			autoSchemaFile: 'schema.gpl'
		}),
		PokemonModule
	],
	controllers: [ AppController ],
	providers: [ AppService ]
})
export class AppModule {}
```
  
## Setting up GraphQL endpoint

We will have a singular entity called _pokemon_. Make a directory inside the __src__ folder and call it __pokemon__. Inside it we will make a __pokemon.entity.ts__.

```ts
// src/pokemon/pokemon.entity.ts

import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm'

@Entity('pokemon')
export class PokemonEntity {
	@PrimaryGeneratedColumn('uuid') id: string

	@Column('varchar', { length: 500, unique: true })
	name: string

	@Column('varchar', { length: 500 })
	type: string

	@Column('numeric') pokedex: number
}
```

Now that we have our entity we can inject it into our pokemon module. Next we will create a _DTO (Data Transfer Object)_, which is a model that defines the shape of the data we are expecting to be sent. Make a folder inside _pokemon_ called __dto__ and create a file called __add-pokemon.dto.ts__.

```ts
// stc/pokemon/dto/create-pokemon.dto.ts

import { Field, ObjectType} from 'type-graphql'

@ObjectType()
export class CreatePokemonDto {
	@Field() readonly id?: string
	@Field() readonly name: string
	@Field() readonly type: string
	@Field() readonly pokedex: number
}
```

Similarly we will set up a _input_ object, which will define the shape of the input parameters to our _mutation query_. In an __input__ folder create __pokemon.input.ts__.

```ts
// src/pokemon/input/pokemon.input.ts

import { Field, InputType } from 'type-graphql'

@InputType()
export class inputPokemon {
	@Field() readonly name: string
	@Field() readonly type: string
	@Field() readonly pokedex: number
}

```

Next we will set up our Pokemon service class. _Services_ are a type of _provider_, and they are a fundamental part in NestJS because they handle the task of __injecting dependencies__. We can create service directly using Nest CLI.

```bash
$ nest generate service pokemon
```

All providers are preceded by the `@Injectable` decorator. Services are usually where we inject our data repository from our ORM, and set up interactions with our data. In our case we will create two methods, `createPokemon` and `getPokemon`.

```ts
// src/pokemon/pokemon.service.ts

import { Injectable } from '@nestjs/common'
import { InjectRepository } from '@nestjs/typeorm'
import { PokemonEntity } from './pokemon.entity'
import { Repository } from 'typeorm'
import { CreatePokemonDto } from './dto/create-pokemon.dto'

@Injectable()
export class PokemonService {
	constructor (@InjectRepository(PokemonEntity) private readonly PokemonRepository: Repository<PokemonEntity>) {}

	async createPokemon (data: CreatePokemonDto): Promise<PokemonEntity> {
		let pokemon = new PokemonEntity()
		pokemon.name = data.name
		pokemon.pokedex = data.pokedex
		pokemon.type = data.type

		await this.PokemonRepository.save(pokemon)

		return pokemon
	}

	async getPokemons () {
		return await this.PokemonRepository.find()
	}
}
```

Next we will make a _resolver_ class. Resolvers are what resolves a GraphQL request to a function. While there are many ways to make a GraphQL resolver, we will use the `@Resolver` decorator from the NestJS GraphQL library. In our resolver class, we will map our Query and Mutation to the functions created in our service class.

```ts
// src/pokemon/pokemon.resolver.ts

import { Resolver, Query, Mutation, Args } from '@nestjs/graphql'
import { PokemonEntity } from './pokemon.entity'
import { CreatePokemonDto } from './dto/create-pokemon.dto'
import { PokemonService } from './pokemon.service'
import { inputPokemon } from './inputs/pokemon.input'

@Resolver((of) => PokemonEntity)
export class PokemonResolver {
	constructor (private readonly pokemonService: PokemonService) {}

	@Query(() => [ CreatePokemonDto ])
	async pokemon () {
		return this.pokemonService.getPokemon()
	}

	@Mutation(() => CreatePokemonDto)
	async createPokemon (@Args('data') data: inputPokemon) {
		return this.pokemonService.createPokemon(data)
	}
}
```

Finally we will encapsulate everything inside our Pokemon module.

```ts
// src/pokemon/pokemon.module.ts

import { PokemonResolver } from './pokemon.resolver'
import { Module } from '@nestjs/common'
import { PokemonService } from './pokemon.service'
import { TypeOrmModule } from '@nestjs/typeorm'
import { PokemonEntity } from './pokemon.entity'

@Module({
	imports: [ TypeOrmModule.forFeature([ PokemonEntity ]) ],
	providers: [ PokemonResolver, PokemonService ]
})
export class PokemonModule {}
```

## Testing the endpoint in playground

And that's it! We can test our app by using `yarn start:dev` and our app will be hosted by default on port 3000. We can navigate to _localhost.com:3000_ to see a "Hello World" message, but if we navigate to _localhost.com:3000.graphql_ we will be greeted with GraphQL playground (the default testing GUI tool used by NestJS). We can test our mutation and query here.

{{< gallery >}}
  {{< figure link="/img/graphql-playground1.png" >}}
  {{< figure link="/img/graphql-playground2.png" >}}
{{< /gallery >}}
