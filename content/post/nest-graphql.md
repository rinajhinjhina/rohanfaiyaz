---
title: "NestJS GraphQL API with TypeORM and Postgres"
date: 2019-08-11T11:25:19+06:00
draft: true
---

{{<figure src="/img/nestjs-graphql.png" thumb="-thumb">}}

In a previous [post](https://https://rohanfaiyaz.com/post/nest.html), I covered how to make a __NestJS__ REST API using TypeORM and Postgres. While it is a relatively simple process to set up a REST API in NestJS, we still are left with the limitations of a REST API. REST depends on our endpoints being clearly defined and this is usually done using the frontend as a guide, cause obviously we want to provide the exact data the frontend needs.

But this is a problem. What if the frontend requirements change and we need an extra data field that is only available through a separate endpoint? Now we have to make two HTTP requests and we will have extra data which is wasteful for mobile internet usage. The other solution is to create a new endpoint on the backend, which is expensive.

Essentially, a REST api is too __low level__. We need something that offers a higher level of abstraction and that is __GraphQL__.

__GraphQL__ is a query language specification created originally by Facebook as a solution to the problems described above. GraphQL offers a way for frontends to describe the exact data need and the shape of the data. The data is defined on the backend as a GraphQL schema. A GraphQL request (known as a _Query_) looks like this:

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
mutatuin{
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
